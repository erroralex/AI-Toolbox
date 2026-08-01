---
name: spring-security
description: Use when adding or changing security in a Spring Boot 3 app — configuring SecurityFilterChain, securing endpoints, login/logout, JWT or OAuth2 resource servers, CORS, password storage, or method-level rules. Also for debugging 401 vs 403 responses, "403 Forbidden" on POST/PUT from a browser (CSRF), redirect loops on the login page, WebSecurityConfigurerAdapter compile errors after an upgrade, or writing security tests with MockMvc.
metadata:
  author: custom (this starter kit)
  version: "1.0"
---

# Spring Security 6 (Spring Boot 3)

Security is configuration you can get silently wrong: the app still runs, requests
still answer — but the wrong people get in, or the right people get blocked and the
error says nothing useful. These are the modern idioms and the traps.

## Golden rules

1. **`WebSecurityConfigurerAdapter` is gone** (removed in Spring Security 6). Declare
   a `SecurityFilterChain` bean with the lambda DSL. Any tutorial showing the adapter
   or `antMatchers` is for Boot 2 — don't copy it.
2. **Matcher order is first-match-wins.** Put specific rules before broad ones and
   end with a catch-all. A `permitAll()` placed after `anyRequest().authenticated()`
   never runs — and won't warn you.
3. **Deny by default.** End every chain with `.anyRequest().authenticated()` (or
   `denyAll()` for an API gateway-fronted service). An endpoint you forgot to map
   should fail closed, not open.
4. **Don't disable CSRF to "fix" a 403** unless the app is genuinely stateless
   (token-auth API, no cookies). Session/browser apps need it; see below.
5. **Passwords**: `PasswordEncoderFactories.createDelegatingPasswordEncoder()`
   (BCrypt by default). Never MD5/SHA-x, never plaintext comparisons, never a custom
   scheme.

## The two archetypes

**Stateless API (JWT/OAuth2 resource server):**

```java
@Bean
SecurityFilterChain api(HttpSecurity http) throws Exception {
    return http
        .securityMatcher("/api/**")
        .csrf(csrf -> csrf.disable())               // OK: no cookies, no sessions
        .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/public/**").permitAll()
            .requestMatchers("/api/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated())
        .oauth2ResourceServer(rs -> rs.jwt(Customizer.withDefaults()))
        .build();
}
```

Validate JWTs with `spring-boot-starter-oauth2-resource-server` and
`spring.security.oauth2.resourceserver.jwt.issuer-uri` — do not hand-roll token
parsing or signature checks.

**Server-rendered (Thymeleaf, sessions):**

```java
@Bean
SecurityFilterChain web(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/css/**", "/js/**", "/error").permitAll()
            .anyRequest().authenticated())
        .formLogin(form -> form.loginPage("/login").permitAll())
        .logout(Customizer.withDefaults())
        .build();      // CSRF stays on (default); Thymeleaf injects the token into forms
}
```

Two chains in one app? Give each a `securityMatcher` and `@Order` — the first chain
whose matcher hits handles the request exclusively.

## CSRF, concretely

- Browser + session cookie → CSRF protection on. Thymeleaf `th:action` forms get the
  token automatically; AJAX needs the `XSRF-TOKEN` cookie/header pair
  (`CookieCsrfTokenRepository.withHttpOnlyFalse()`).
- Pure token API (Authorization header, no cookies) → CSRF off is correct, not a hack.
- Symptom of getting this wrong: GETs work, every POST/PUT/DELETE returns 403 with an
  empty body.

## Method security

`@EnableMethodSecurity` on a config class, then `@PreAuthorize` on service methods:

```java
@PreAuthorize("hasRole('ADMIN') or #ownerId == authentication.name")
public Order cancel(String ownerId, long orderId) { ... }
```

Use it as the inner line of defense for business rules (ownership checks); URL rules
alone break when a second controller exposes the same service.

## CORS

Configure CORS *inside* the security chain (`http.cors(...)` + a
`CorsConfigurationSource` bean). `@CrossOrigin` on controllers doesn't cover the
security filter's preflight handling — the browser's OPTIONS call will 401/403 and
the error will look like a frontend bug. Never `allowedOrigins("*")` together with
`allowCredentials(true)` — Spring rejects it at runtime.

## Common errors

| Symptom | Cause | Fix |
|---|---|---|
| 403 on every POST, GETs fine | CSRF token missing | Browser app: send the token; API: disable CSRF only if truly stateless |
| 401 expected but got 403 | Authenticated but unauthorized — role/authority mismatch | Check `ROLE_` prefix: `hasRole("ADMIN")` ≡ authority `ROLE_ADMIN` |
| Redirect loop on /login | Login page itself requires auth | `.requestMatchers("/login").permitAll()` (and its CSS/JS) |
| `WebSecurityConfigurerAdapter` not found | Boot 2 → 3 upgrade | Rewrite as `SecurityFilterChain` bean |
| Static resources blocked | Catch-all before resource matcher | Permit `/css/**`, `/js/**`, favicon before `anyRequest()` |
| Second filter chain ignored | Missing `securityMatcher`/`@Order` | First matching chain wins — scope each chain |

## Testing

`spring-security-test` is the difference between testing security and assuming it:

```java
@WebMvcTest(OrderController.class)
class OrderControllerSecurityTest {
    @Test void anonymousIsRejected() throws Exception {
        mvc.perform(get("/api/orders")).andExpect(status().isUnauthorized());
    }
    @Test @WithMockUser(roles = "ADMIN")
    void adminCanDelete() throws Exception {
        mvc.perform(delete("/api/orders/1").with(csrf())).andExpect(status().isNoContent());
    }
}
```

Write the negative tests (wrong role, anonymous, missing CSRF) — passing positive
tests prove access works, only negative tests prove protection works.
