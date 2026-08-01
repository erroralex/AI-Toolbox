# Handover Document - Latent Library

## Project Summary

**Latent Library** is a high-performance, local-first desktop asset manager designed for the AI image generation ecosystem. It features SQLite FTS5 search, dynamic Smart Collections, live hot-folder monitoring, and an ONNX-powered AI auto-tagger.

- **Backend:** Java 21 / Spring Boot 3.3 (`backend/`)
- **Frontend:** Vue 3 + Vite + PrimeVue + Pinia (`frontend/`)
- **Desktop Packaging:** Electron 31 (`electron/`)
- **Database:** SQLite with Flyway migrations (`data/`)

---

## AI Assistant Setup & Configuration

The project has been fully configured using the **Drop-in Brain** system:

- **Rulebook (`AGENTS.md`):** Contains universal engineering guidelines, git rules, testing contracts, and technology-specific directives (Spring Boot 3.x, SQLite/Flyway, Vue 3, Modular Monolith).
- **Assistant Shims:**
  - `CLAUDE.md` — Imports `@AGENTS.md` for Claude Code compatibility.
  - `GEMINI.md` — Full identical copy of `AGENTS.md` for Gemini CLI compatibility.
  - `.claude/settings.json` — Permission allowlist for Maven, npm, and git operations.
- **Skills Directory (`.agents/skills/`):**
  - `ai-setup-doctor` — Diagnostic self-check tool for instruction loading and link integrity.
  - `sql-migrations` — Flyway conventions, schema changes, and error recovery.
  - `spring-security` — Spring Boot 3 security filter chain & CORS patterns.
  - `api-design` — REST conventions, `ProblemDetail` error responses, and pagination.
  - Junction `.claude/skills` -> `.agents/skills` linked and added to `.gitignore`.

Verification can be re-run at any time by saying **"check my AI setup"**.

---

## UI Redesign Status & Roadmap

A complete UI redesign plan has been authored based on the official **Latent Design System** (`Latent-Design-System`).

- **Implementation Plan Location:** [docs/implementation_plan.md](docs/implementation_plan.md)
- **Source Design System Repository:** `https://github.com/erroralex/Latent-Design-System.git` (`c:\Users\error\IdeaProjects\Projects\Latent-Design-System`)

### Key Decisions Confirmed

1. **Single Dark Mode Standardisation:**
   Transitioning from legacy multi-themes (`neon`, `light`, `gold`, `fanfriction`) to the desaturated Latent dark mode canvas (`#0A0A0D`), flat surface levels (`#14151B`), hairline white borders (`rgba(255,255,255,0.06-0.18)`), and desaturated cyan (`#4FD8D0`) / violet (`#9B7EF5`) accents.
2. **Type-safe Lucide Icons:**
   Installing `lucide-vue-next` to standardize stroke weight (1.5-1.8px) across all UI views and components.
3. **App Shell Restructuring & Brand Assets:**
   - Integrated official brand mark (`latent-mark.svg`) and lockup (`latent-lockup.svg`) vectors.
   - Updated developer credit logo in `README.md` and app sidebar (`FolderNav.vue`) to the official Design System developer mark (`alx_logo.png`).
   - Integrating a 52px frameless titlebar with custom window controls and a `StatusPill` component monitoring backend process health (`online`, `starting`, `offline`).

---

## Next Execution Steps

1. **Install Dependencies:**
   ```bash
   cd frontend && npm install lucide-vue-next
   ```
2. **Copy Tokens & Brand Assets:**
   Copy `styles.css` and `tokens/*.css` from `Latent-Design-System` into `frontend/src/assets/css/latent/`. (Brand vectors `latent-mark.svg`, `latent-lockup.svg`, and `alx_logo.png` have already been copied into `frontend/src/assets/` and linked).
3. **Build Component Primitives:**
   Implement Vue 3 `<script setup>` SFCs in `frontend/src/components/ds/` (`Titlebar`, `StatusPill`, `LButton`, `LInput`, `LSelect`, `LSwitch`, `LSlider`, `LCard`, `LDialog`, `SegmentedControl`, `NavItem`).
4. **Restyle Shell & Views:**
   Update `App.vue`, `FolderNav.vue`, `BrowserToolbar.vue`, `ImageCard.vue`, `MetadataSidebar.vue`, `CollectionsView.vue`, `ComparatorView.vue`, `SpeedSorterView.vue`, `DuplicateDetectiveView.vue`, and `ScrubView.vue`.
5. **Verify:**
   Build frontend (`cd frontend && npm run build`) and run backend tests (`cd backend && ./mvnw test`).

