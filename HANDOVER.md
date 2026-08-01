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

