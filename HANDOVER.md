# Handover Document - Latent Library

## Project Summary

**Latent Library** is a high-performance, local-first desktop asset manager designed for the AI image generation ecosystem. It features SQLite FTS5 search, dynamic Smart Collections, live hot-folder monitoring, and an ONNX-powered AI auto-tagger.

- **Backend:** Java 21 / Spring Boot 3.3 (`backend/`)
- **Frontend:** Vue 3 + Vite + PrimeVue + Pinia (`frontend/`)
- **Desktop Packaging:** Electron 31 (`electron/`)
- **Database:** SQLite with Flyway migrations (`data/`)
- **Active Feature Branch:** `feature/ui-redesign`

---

## AI Assistant Setup & Configuration

The project is configured using the **Drop-in Brain** system:

- **Rulebook (`AGENTS.md` / `GEMINI.md`):** Contains universal engineering guidelines, git rules, testing contracts, and technology-specific directives (Spring Boot 3.x, SQLite/Flyway, Vue 3, Modular Monolith).
- **Assistant Shims:**
  - `CLAUDE.md` — Imports `@AGENTS.md` for Claude Code compatibility.
  - `GEMINI.md` — Full identical copy of `AGENTS.md` for Gemini CLI compatibility.
  - `.claude/settings.json` — Permission allowlist for Maven, npm, and git operations.
- **Skills Directory (`.agents/skills/`):**
  - `ai-setup-doctor` — Diagnostic self-check tool for instruction loading and link integrity.
  - `sql-migrations` — Flyway conventions, schema changes, and error recovery.
  - `spring-security` — Spring Boot 3 security filter chain & CORS patterns.
  - `api-design` — REST conventions, `ProblemDetail` error responses, and pagination.

---

## UI Redesign Implementation Status

The application has undergone a complete UI redesign based on the official **Latent Design System** specifications (`Latent-Design-System`).

### 1. Design System Primitive Library (`frontend/src/components/ds/`)

Built 15 reusable Vue 3 `<script setup>` SFC primitives wrapped with type-safe `lucide-vue-next` icons:
- `Titlebar.vue` — Frameless titlebar with window IPC controls and engine status indicator.
- `StatusPill.vue` — Engine health indicator (`online`, `starting`, `offline`).
- `LButton.vue` & `LIconButton.vue` — Standardized button components (`primary`, `secondary`, `danger`, `ghost`).
- `LInput.vue` & `LSelect.vue` — Form input and select controls.
- `LCheckbox.vue`, `LSwitch.vue`, `LSlider.vue` — Checkbox, toggle switch, and range slider controls.
- `LBadge.vue` & `LProgressBar.vue` — Status badge pill and progress bar indicators.
- `LCard.vue` & `LDialog.vue` — Card containers and modal dialog windows.
- `SegmentedControl.vue` & `NavItem.vue` — View toggle and navigation item primitives.

### 2. App Shell & Navigation Layout

- **Frameless Titlebar (`Titlebar.vue`)**:
  - Displays the official Latent vector mark (`latent-mark.svg`), app title ("Latent Library"), and `StatusPill` right next to the title text.
  - Exposes minimize, maximize, and close window controls via Electron IPC (`window.windowAPI` / `window.electronAPI`).
- **Single Stacked Left Sidebar Layout**:
  - `Sidebar.vue` (240px fixed width): Consolidated single left navigation sidebar hosting top view links (`Gallery`, `Collections`, `Comparator`, `Scrubber`, `Speed Sorter`, `Duplicates`), embedded scrollable `FolderNav` tree view (`Collections`, `Pinned`, `This PC`), `Settings` modal toggle button, and developer credit logo link (`alx_logo.png`).
  - `FolderNav.vue`: Embedded directly inside `Sidebar.vue` between navigation links and settings with seamless transparent container background.
- **LoRA Pills Formatting, Click-to-Copy & Tooltips**:
  - `MetadataSidebar.vue` & `layout.css`: Formatted pill labels as full `<lora:name:weight>` tag strings (e.g. `<lora:ZxY_Krea2_v4:1>`).
  - Added click-to-copy handler on `.lora-chip` that copies the full tag string (`<lora:Pony_QualityV4.0:1>`) to clipboard with a 1.5s info toast message.

  - Added native `title="Click to copy LoRA name"` hover tooltip and `cursor: pointer` hover indicator.
  - Removed legacy neon gradient glow pseudo-elements (`.lora-chip::before` / `::after`) in `layout.css`.
  - Restyled `.lora-chip` using Latent Design System tokens: `#23252F` surface background, `#9B7EF5` accent text, `JetBrains Mono` font, 6px border radius, and soft subtle borders.
- **Canvas Zoom-Out Double Background Fix**:
  - `SingleImageViewer.vue`: Removed `shadow-8` from `<img class="absolute inset-0 z-1">` so the image DOM element scales down transparently when zooming out without leaving a floating dark box-shadow rectangle on the outer canvas.
- **Single Native Splash Screen Experience**:
  - `App.vue`: Removed duplicate inline Vue splash screen overlay (`loading-overlay-ds`). Electron's native splash window (`electron/splash.html`) handles startup during backend boot, and the app shell reveals instantly with no double splash flashing.



- **Electron Native Splash Screen (`electron/splash.html`)**:
  - Restyled with `#0A0A0D` canvas background, ambient radial cyan/violet glow, official SVG mark, brand gradient text, and animated loading bar.

### 3. Feature Views & Toolbars

- **`MetadataSidebar.vue` & `TaggerSidebar.vue`**:
  - Restyled metadata inspection sidebar and WD14 AI Auto-Tagger with `#0A0A0D` canvas, `#14151B` surface level, `#23252F` inputs, confidence slider, and monospace LoRA badges (`#9B7EF5`).
- **`BrowserToolbar.vue`**:
  - Refactored toolbar containing search input, AI tag toggle, view mode switcher (`Gallery` / `Browser`), and metadata dropdown filters (`Model`, `Sampler`, `LoRA`, `Stars`).
- **`ComparisonMetadataPanel.vue` & `ComparatorView.vue`**:
  - Side-by-side comparison panels with brand gradient headers (`linear-gradient(90deg, #67E0D8, #9B7EF5)`), surface card containers, and synchronized split-viewer integration.
- **`ScrubView.vue`**:
  - Metadata scrubber featuring a drag-and-drop file zone, shield icon, and clean export copy button.
- **`SpeedSorterView.vue`**:
  - High-efficiency keyboard-driven triage view (`1-5`, `X`/`DEL`, `Ctrl+Z`, `SPACE`) with fixed container bounds preventing taskbar clipping.
- **`DuplicateDetectiveView.vue`**:
  - Perceptual dHash and cryptographic SHA-256 duplicate pair inspector with batch resolve dialog.

### 4. System Stability & Global Overrides

- **Global Response Error Interceptor (`api.js`)**:
  - Updated to ignore global error toast popups on standard `404 Not Found` responses, allowing components to handle missing metadata/thumbnails gracefully.
- **PrimeVue Overrides (`primevue-overrides.css` & `buttons.css`)**:
  - Removed all legacy pseudo-element blur glow rules (`filter: blur(4px)`, `--grad-hover`). Buttons, sliders, context menus, and dropdown panels now feature clean Latent DS surface levels and borders.

### 5. Civitai Resources & Metadata Cache Refresh

- **Civitai Resources Parsing (`CommonStrategy.java`)**:
  - Enhanced `CommonStrategy.java` to parse embedded `Civitai resources:` JSON arrays (handling escaped Unicode sequences like `OB\u534A\u5199...`), extracting `Model = "FLUX (Dev)"` and LoRAs into tags (`<lora:OB半写实肖像画...:0.55>`).
- **Metadata Cache Auto-Refresh (`ImageMetadataService.java`)**:
  - Upgraded `ImageMetadataService.getCachedMetadata` to detect stale/incomplete cached metadata entries in SQLite (where `Model` is missing or `-`) and automatically re-extract parameters from the file, refreshing the database cache.
- **Unit Tests**:
  - Added unit test cases in `CommonStrategyTest.java` and `ImageMetadataServiceTest.java` covering Civitai Unicode resource parsing and stale cache auto-refresh.

### 6. Design System Deep-Dive Audit & Token Cleanup

Follow-up audit pass to remove remaining legacy (pre-redesign) CSS variables and non-DS
component wrappers that survived the initial redesign:

- **`ImageSplitViewer.vue`**: `--accent-primary` → `--color-accent-primary`; removed
  non-existent `--grad-hover`; "Left"/"Right" badges converted from PrimeFlex alpha
  classes to a DS surface-overlay badge style.
- **`SingleImageViewer.vue`**: `--status-danger`/`--text-primary` → `--color-danger`/
  `--color-text-primary`; nav arrow hover states converted from PrimeFlex alpha
  classes to DS surface hover styles.
- **`VirtualGallery.vue`**: selection outline/glow switched from legacy PrimeVue
  `--primary-color`/`--primary-color-rgb` to `--color-accent-primary`/`--glow-primary`;
  removed custom webkit scrollbar rules (now inherits the global DS scrollbar from
  `base.css`).
- **`MetadataSidebar.vue`**: removed a stale, duplicated `<style scoped>` block
  referencing obsolete tokens (`--bg-sidebar-right`, `--glass-blur`, `--border-light`,
  `--bg-input`, `--border-input`, `--grad-hover`, `--grad-text`).
- **`SystemError.vue`**: switched to DS tokens (`--color-bg-canvas`,
  `--color-surface-1`, `--color-border-strong`), replaced hardcoded `#ff4d4d` with
  `--color-danger`, and swapped the raw PrimeVue `<Button>` for `<LButton variant="primary">`.
- **`ImageBrowserView.vue`**: rename-modal style overrides moved to DS tokens; footer
  actions swapped from raw PrimeVue `<Button>` to `<LButton variant="secondary">` /
  `<LButton variant="primary">`.
- **`LButton.vue`**: added an `icon` slot alias (alongside the existing `icon-left`/
  `icon-right`) so the `<template #icon>` convention already used at ~20 call sites
  across the app (SystemError, ImageBrowserView, TaggerSidebar, CollectionsView,
  ComparatorView, DuplicateDetectiveView, ScrubView, SpeedSorterView,
  ComparisonMetadataPanel) actually renders — previously those icons were silently
  dropped because no slot named `icon` existed.
- **Toast notifications (`primevue-overrides.css`)**: added a full `.p-toast` override
  block (message card, summary/detail text, severity accent borders for
  info/success/warn/error, close icon) using DS tokens throughout. Verified every
  fallback hex against `Latent-Design-System/tokens/colors.css` and `effects.css` —
  exact match (`--color-accent-primary #4FD8D0`, `--color-success #3DD68C`,
  `--color-warning #F5B84E`, `--color-danger #F2665B`, `--radius-lg 12px`,
  `--shadow-panel`, `--duration-fast 120ms`).

**Known pre-existing, out-of-scope issue**: the legacy multi-theme system
(`themes/neon.css`, `themes/gold.css`, `themes/light.css`, `themes/fanfriction*.css`)
and `components/layout.css` still define/consume the old token namespace
(`--bg-app`, `--accent-primary`, `--grad-hover`, `--status-danger`, etc.). These
predate the Latent DS redesign and are a separate theming feature — left untouched
since no component in this audit depends on them anymore (all fixed components now
use `--color-*` DS tokens with hardcoded fallbacks).

Verified via `cd frontend && npm run build` (clean) and `cd backend && ./mvnw test`
(153 tests, 0 failures).

### 7. Nav-Tree Root Selection Fix & Cross-View Header Consistency

- **`FolderNav.vue`**: the `Collections` / `Pinned` / `This PC` root nodes now carry
  `selectable: false`. Previously clicking one of these group headers applied
  PrimeVue's `p-highlight` selection background (visible accent tint) even though
  `navigateToNode` already no-ops for `type: 'root'` — the row looked "selected" but
  did nothing. `selectable: false` disables PrimeVue's click-to-select path entirely
  for those nodes (verified directly against `primevue/tree` source —
  `onNodeClick`/`handleSelectionWith(out)MetaKey` gate on `node.selectable !== false`),
  so clicking a root row now only expands/collapses via the toggler, matching how a
  tree section header should behave.
- **Shared view header typography**: audited all tool views (Scrubber, Comparator,
  Duplicate Detective, Collections, Speed Sorter) for visual consistency. Found the
  "hero" title/subtitle pattern had drifted independently per-file — 28px in three
  views but 24px in Duplicate Detective (fighting a stray PrimeFlex `text-3xl`
  class), and Comparator's subtitle had no explicit font-size at all (fell back to
  browser default instead of Scrub's deliberate 14px). Also found `.text-white`
  reimplemented identically (as a PrimeFlex-white → DS-token override) in 7 separate
  files.
  - Extracted `.view-title-hero`, `.view-subtitle`, and a global `.text-white` into
    `base.css` and pointed Scrub/Comparator/Collections/Duplicate Detective at them,
    deleting the now-redundant local copies (net −45 lines).
  - Speed Sorter's compact toolbar header was deliberately left alone — it's a
    denser, keyboard-driven layout, not a page hero, so forcing the same treatment
    there would hurt usability rather than help consistency.
- **Verification**: `npm run build` clean, backend tests 153/153. Live-verified by
  launching `electron/` standalone (`npm start`) and screenshotting the running app —
  confirmed the Pinned/Test Suite nav-tree behavior visually (root row unhighlighted,
  child row correctly highlighted) and that the window renders normally end-to-end.
  Note for future dev-mode browser testing: the Vite proxy (`vite.config.js`) targets
  a fixed `localhost:8080`, but the backend binds a random port per launch by design
  (see `SecurityConfig`'s handshake token, normally supplied to the renderer via
  Electron IPC) — plain-browser testing against `npm run dev` needs the backend
  started with `--server.port=8080` and the handshake token manually injected, or use
  the packaged Electron app instead.

### 8. ScrubView Vertical Centering Fix (found from real screenshots)

The horizontal-centering check in section 7 passed, but missed a real bug: unlike
Comparator/Collections/Duplicate Detective (title pinned near the top via
`mb-4`/`mb-5`, no `justify-content-center` on the outer flex container), `ScrubView`'s
root element had `justify-content-center` on the *whole* content block (title +
card), vertically centering the entire group mid-viewport instead of anchoring the
title to the top like every sibling view. Full-window ShareX screenshots from the
user made the mismatch obvious — title sat around y≈355px in a 1350px-tall window
vs. y≈117px for Collections/Comparator.

- **Fix**: `ScrubView.vue` root now matches the Comparator/Collections structure —
  outer container is `flex flex-column h-full p-4 overflow-hidden` (no
  `justify-content-center`), title wrapper is `flex flex-column align-items-center
  mb-4 flex-shrink-0`, and the `LCard` drop-zone is wrapped in a new
  `flex-grow-1 flex align-items-center justify-content-center` div so it still
  centers vertically *within the remaining space* below the anchored title, mirroring
  how Comparator centers its dropzones.
- **Process note — stale JAR trap**: verifying this fix live initially gave a false
  negative. Electron (standalone `npm start`, not via IntelliJ) spawns
  `java -jar backend/target/backend.jar`, which bundles a *packaged snapshot* of
  `frontend`'s build output taken at the JAR's last `mvn package`. Running
  `cd frontend && npm run build` alone updates `backend/src/main/resources/static/`
  on disk but does **not** repackage `backend/target/backend.jar` — so Electron kept
  serving the pre-fix UI until `cd backend && ./mvnw clean package -DskipTests` was
  run to rebuild the jar. Any time a frontend change needs to be checked in the
  standalone Electron shell (as opposed to IntelliJ's "Full Stack Dev", which runs
  `BackendApplication` directly from `target/classes` and reflects source changes
  after a Maven build automatically), rebuild the backend jar first.
- **Verification**: rebuilt frontend + backend jar, relaunched Electron, screenshotted
  Scrubber and Comparator back-to-back in the same session — titles now align at the
  identical y-position. Backend tests 153/153.

---

## Verification & Build Commands

- **Build Frontend:**
  ```bash
  cd frontend && npm run build
  ```
- **Run Backend Unit Tests:**
  ```bash
  cd backend && ./mvnw test
  ```
- **Run Desktop App Locally:**
  ```bash
  # Terminal 1 (Backend)
  cd backend && ./mvnw spring-boot:run

  # Terminal 2 (Frontend)
  cd frontend && npm run dev

  # Terminal 3 (Electron Shell)
  cd electron && npm start
  ```

