# Agent Instructions

## Project

- **Name:** Latent Library
- **Purpose:** High-performance local-first desktop asset manager for AI image generation ecosystem with SQLite FTS5 search, Smart Collections, and live hot-folder monitoring.
- **Stack:** Java 21 / Spring Boot 3.3 (Backend), Vue 3 + PrimeVue (Frontend), SQLite + Flyway, Electron 31 (Desktop Shell)
- **Build:** Backend: `cd backend && ./mvnw clean package -DskipTests` | Frontend: `cd frontend && npm run build` | Electron: `cd electron && npm run dist`
- **Test:** Backend: `cd backend && ./mvnw test`
- **Run locally:** Backend: `cd backend && ./mvnw spring-boot:run` | Frontend: `cd frontend && npm run dev` | Electron: `cd electron && npm start`

## Workflow

- Before non-trivial changes: state your plan in 2–5 bullet points, then implement.
  If requirements are ambiguous, ask — don't guess.
- Work in small, verifiable steps. One logical change at a time.
- Write or update the test for a behavior change **before or with** the code,
  never "later".
- Before claiming anything works: run the test/build command and show the result.
  "Should work" is not done — verified is done.
- When a task touches unfamiliar code, read the surrounding files first and follow
  the patterns already there.
- Check `.agents/skills/` for an applicable skill before starting specialized work
  (framework setup, UI design, reviews); use it if its description matches the task.

## Engineering rules

- **YAGNI:** build what the task needs, nothing speculative. No extra config options,
  abstraction layers, or "flexibility" that wasn't asked for.
- **DRY, but not premature:** extract shared code on the third occurrence, not the
  second. Duplication is cheaper than the wrong abstraction.
- **Single responsibility:** one reason to change per class/module/function. If a file
  needs "and" to describe what it does, split it.
- **Depend on interfaces at boundaries** (service ↔ persistence, domain ↔ external
  APIs); don't interface-ify everything else.
- Keep functions short and files focused. A file approaching ~300 lines is a signal
  to split.
- Prefer boring, idiomatic solutions over clever ones. Optimize only with a
  measurement in hand.
- Fail fast: validate inputs at system boundaries, throw early with specific messages,
  never swallow exceptions silently.

## Testing

- Every bugfix gets a regression test that fails before the fix and passes after.
- Test behavior through public interfaces, not implementation details.
- Never delete, skip, or weaken a test to make a change pass. If a test seems wrong,
  say so and ask.
- Tests must be deterministic: no sleeps for synchronization, no order dependence,
  no shared mutable state between tests.

## Git

- Small commits, one logical change each. Imperative subject line ≤ 72 chars;
  body explains *why* when it isn't obvious.
- **No AI attribution anywhere in git:** no `Co-Authored-By` trailers naming an AI,
  no "Generated with ..." lines in commit messages, PR descriptions, or code
  comments. Commits carry the human author's identity only.
- Never commit secrets, credentials, or generated artifacts.
- Never force-push or rewrite history on shared branches.

## Security

- No secrets in code or config files — use environment variables or a secret manager.
- All user input is untrusted: validate at the boundary, use parameterized
  queries/bound parameters, escape output in templates.
- Don't add dependencies for trivial tasks; when adding one, prefer well-maintained,
  widely-used libraries.

## Spring Boot

> Skill: `.agents/skills/java-springboot/` covers these rules in depth — invoke it for
> non-trivial Spring work.

- Spring Boot **3.x**: imports are `jakarta.*`, never `javax.*`. Don't apply Boot 2
  idioms (e.g. `WebSecurityConfigurerAdapter` is gone — use a `SecurityFilterChain` bean).
- Package by feature (`com.app.order`, `com.app.user`), not by layer.
- Constructor injection only, dependencies `private final`. No `@Autowired` on fields.
- Controllers are thin: validate, delegate to a service, map to DTO. Never expose JPA
  entities in request/response bodies — use DTOs (Java `record`s) with Bean Validation
  annotations and `@Valid`.
- Business logic lives in `@Service` classes; services are stateless;
  `@Transactional` on the service method that forms the unit of work (note: it only
  applies to calls that cross a proxy boundary — self-invocation is not transactional).
- One global exception handler: `@RestControllerAdvice` returning RFC 9457
  `ProblemDetail`. No try/catch-and-log-and-rethrow noise in controllers/services.
- Configuration: `application.yml` + `@ConfigurationProperties` records; profiles for
  environment differences; secrets only via environment variables.
- Set `spring.jpa.open-in-view: false`; load what the use case needs explicitly.
- Logging via SLF4J with parameterized messages (`log.info("user {}", id)`); never
  log secrets or full request bodies.
- Tests: unit-test services with JUnit 5 + Mockito; slice tests `@WebMvcTest` /
  `@DataJpaTest` for the edges; Testcontainers for integration tests against the real
  database engine — never test JPA mappings against H2 when production is
  Postgres/MySQL. Reserve full `@SpringBootTest` for a few end-to-end paths.

## Databases & persistence

- **Schema changes only via versioned migrations** (Flyway or Liquibase). Never edit a
  migration that has been applied anywhere — write a new one. Never let JPA
  `ddl-auto` manage schema beyond `validate` outside local experiments.
- Parameterized queries / bound parameters everywhere. String-concatenated SQL is
  forbidden — including in `@Query(nativeQuery = true)` and JDBC templates.
- Every foreign key gets an index; every query pattern in the code should have a
  matching index strategy. When adding a query, state which index serves it.
- Pagination is mandatory for any list endpoint/query — no unbounded `findAll()`.
- JPA specifics:
  - Associations `LAZY` by default; load eagerly per use case with `JOIN FETCH` or
    `@EntityGraph`. Watch for N+1: a loop that triggers a query per row is a bug.
  - Use DTO projections for read-heavy queries; don't hydrate entities to copy
    three fields.
  - Optimistic locking (`@Version`) on entities that are concurrently edited.
  - `equals`/`hashCode` on entities: based on ID with care, or business key — never
    on all fields.

### PostgreSQL
- Prefer `identity` or UUID primary keys; `timestamptz` for timestamps; `text` over
  `varchar(n)` unless a real constraint exists; `jsonb` for genuinely schemaless data
  (not as an excuse to skip modeling).

### MySQL
- `utf8mb4` charset everywhere (plain `utf8` is broken for emoji/4-byte chars);
  InnoDB engine; mind index length limits on long varchar columns; `TIMESTAMP` vs
  `DATETIME` semantics differ — store UTC and be explicit.

### SQLite
- For local dev, embedded, and single-writer use only — not a concurrent-writes
  production server.
- Run with `PRAGMA foreign_keys = ON` (off by default).
- `ALTER TABLE` is limited (no drop/modify column before 3.35) — write migrations as
  create-new-table / copy / rename when needed.
- Don't fight its dynamic typing: declare sensible column types and validate in the app.

## Frontend core (HTML / CSS / JavaScript)

> Skills: `.agents/skills/frontend-design/` for building distinctive UI;
> `.agents/skills/web-design-guidelines/` for auditing existing UI.

- Semantic HTML first: native elements (`button`, `nav`, `dialog`, `details`) before
  div+JS reimplementations. Heading levels in order; one `h1` per page.
- Accessibility is not optional: every input has a `label`; interactive elements are
  keyboard-reachable with visible focus; images have meaningful `alt` (or empty for
  decorative); color contrast ≥ WCAG AA; ARIA only when no native element fits.
- Responsive = mobile-first: base styles for small screens, `min-width` media queries
  upward; relative units (`rem`, `%`) over fixed px for layout; test at 320px,
  768px, 1280px.
- CSS: design tokens as custom properties (colors, spacing, type scale) defined once;
  prefer flexbox/grid over floats and absolute positioning; avoid `!important` and
  deep selector chains; co-locate component styles.
- JavaScript: ES modules; `const`/`let`, never `var`; `async/await` with explicit
  error handling over raw promise chains; `fetch` with status checks (a 404 does not
  reject); no jQuery in new code.
- Forms: validate client-side for UX, **always** revalidate server-side; disable the
  submit button while a request is in flight; show field-level errors.
- Escape user content rendered into the DOM — `textContent` over `innerHTML`;
  if HTML insertion is unavoidable, sanitize.
- Performance basics: `defer` scripts; set width/height (or aspect-ratio) on images;
  lazy-load below-the-fold images; don't ship a framework for a static page.

## Vue (3.x)

> Skill: `.agents/skills/vue/` (Anthony Fu, generated from the official Vue docs)
> has detailed references for script-setup macros, reactivity, and built-in
> components — consult it for non-trivial Vue work.

- Composition API with `<script setup>` in Single-File Components. No Options API in
  new code; don't mix the two styles in one component.
- Don't apply Vue 2 idioms: no `this`, no mixins (use composables), no event bus,
  no `Vue.set` (reactivity is proxy-based now), filters are gone.
- State: component-local with `ref`/`reactive`; shared app state in Pinia stores —
  not prop-drilled five levels, not a global reactive object.
- Data flow: props down, `emit` up. Never mutate a prop; declare props/emits with
  types (`defineProps<...>()` / `defineEmits<...>()`).
- Derived state is `computed`, not a method, not a watcher copying values. Watchers
  are a last resort for side effects (e.g. fetch on param change).
- `v-for` always with a stable `:key` (not the index when the list reorders).
  Don't pair `v-if` with `v-for` on the same element.
- Extract reuse into composables (`useThing()`), one concern per composable.
- Router: lazy-load route components; guard auth in navigation guards, not in
  components.
- Tests: Vitest + Vue Test Utils; test rendered behavior and emitted events, not
  internal refs.
- Never use `v-html` with user-provided content.

## Architecture: modular monolith

- One deployable, but strict internal module boundaries: each feature package
  (`order`, `user`, `billing`) is a module with an explicit API (public services,
  events, DTOs) and private internals (entities, repositories, helpers).
- Modules talk through each other's **API only** — never import another module's
  entity, repository, or internal class. Cross-module reads return DTOs.
- For side effects across modules, prefer application events
  (`ApplicationEventPublisher` / `@EventListener` or Spring Modulith) over direct
  service-to-service call chains.
- Keep the shared kernel (`common/`) minimal: truly generic utilities and base types
  only. If a "common" class knows about a feature, it belongs to that feature.
- Enforce boundaries with tooling when the project grows: ArchUnit tests or Spring
  Modulith verification — a convention nobody checks will erode.
- One database is fine; keep each module's tables owned by that module. No cross-module
  joins in application queries — go through the owning module's API.
- Don't split into microservices speculatively. A well-modularized monolith can be
  split later along these same boundaries if scale ever demands it.
