# Implementation Plan - Complete UI Redesign (Latent Design System)

This plan outlines the complete UI redesign of **Latent Library** by integrating the official **Latent Design System** (`Latent-Design-System`). It replaces the legacy, hardcoded "Deep Neon" / multi-theme CSS setup with a desaturated dark-mode-only token system, standardized Vue 3 UI primitives, Lucide iconography (`lucide-vue-next`), frameless titlebar chrome with process health status monitoring, and unified card/panel styling.

## Design Decisions Approved

- **Single Dark Mode Standardisation**: Per the Latent Design System specifications, the app transitions to a single, desaturated dark-mode theme (`--color-bg-canvas #0A0A0D` with desaturated Latent Cyan `#4FD8D0` & Latent Violet `#9B7EF5` accents). The legacy theme picker (Deep Neon, Light, Gold, Fanfiction) will be removed in favor of the unified design system tokens.
- **Icon System Standardisation**: Installing `lucide-vue-next` in `frontend/package.json` for clean, type-safe Vue 3 SVG icon components matching stroke weight (1.5–1.8px) across the Latent tool suite.

---

## Proposed Changes

### 1. Design System Tokens & Brand Assets

Copy tokens and official brand SVGs from `Latent-Design-System` into `frontend/src/`.

#### [NEW] [styles.css](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/css/latent/styles.css)
Main stylesheet linking `@import` rules for all Latent token files.

#### [NEW] [colors.css](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/css/latent/tokens/colors.css)
Color scale tokens (`--gray-*`, `--cyan-*`, `--violet-*`), desaturated accent definitions, surfaces (`--color-surface-1/2/3`), and hairline borders (`--color-border-subtle/default/strong`).

#### [NEW] [typography.css](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/css/latent/tokens/typography.css)
Font size, weight, line-height tokens for Inter (UI) and JetBrains Mono (paths/hashes/prompts).

#### [NEW] [spacing.css](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/css/latent/tokens/spacing.css)
Spacing scale tokens (`--space-1` to `--space-16`).

#### [NEW] [effects.css](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/css/latent/tokens/effects.css)
Border radius (`--radius-sm/md/lg/xl/full`), dark panel shadows (`--shadow-card/panel`), focus glows (`--glow-primary`), and subtle transition timing (`--duration-fast/base/slow`, `--ease-standard`).

#### [NEW] [fonts.css](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/css/latent/tokens/fonts.css)
Font face imports for Inter and JetBrains Mono.

#### [NEW] [latent-mark.svg](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/latent-mark.svg) & [latent-lockup.svg](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/latent-lockup.svg)
Official Latent brand mark and lockup vectors.

#### [DELETE] `frontend/src/assets/css/themes/*`
Delete legacy theme files (`neon.css`, `light.css`, `gold.css`, `fanfriction.css`, `fanfriction-light.css`).

---

### 2. Reusable Vue 3 Design System Components (`frontend/src/components/ds/`)

Implement the 15 standard Latent Design System primitives as Vue 3 `<script setup>` SFCs using tokens and class-free / token-bound styles.

#### [NEW] [Titlebar.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/Titlebar.vue)
52px fixed frameless titlebar with `-webkit-app-region: drag`, title/logo lockup, window controls, and integrated backend process health status pill.

#### [NEW] [StatusPill.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/StatusPill.vue)
Backend service health indicator (`online`, `starting`, `offline`) with pulsing dot animation (`ds-pulse`).

#### [NEW] [LButton.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LButton.vue)
Button primitive with `primary`, `secondary`, `ghost`, `danger`, and `cta` (brand gradient) variants.

#### [NEW] [LIconButton.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LIconButton.vue)
Icon button primitive with soft accent focus ring and dark surface background.

#### [NEW] [LInput.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LInput.vue)
Input primitive with subtle border, focus glow ring (`--glow-primary`), and search icon support.

#### [NEW] [LSelect.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LSelect.vue)
Custom dropdown select primitive using desaturated surface styling.

#### [NEW] [LCheckbox.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LCheckbox.vue) & [LSwitch.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LSwitch.vue)
Checkbox and toggle switch primitives with smooth transition state.

#### [NEW] [LSlider.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LSlider.vue)
Range slider primitive for interrogator threshold & comparator position control.

#### [NEW] [LBadge.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LBadge.vue) & [LProgressBar.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LProgressBar.vue)
Status badge pill and gradient progress bar primitives.

#### [NEW] [LCard.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LCard.vue) & [LDialog.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/LDialog.vue)
Surface primitives with `--color-surface-1`, 1px hairline border (`--color-border-subtle`), and `--shadow-card`.

#### [NEW] [SegmentedControl.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/SegmentedControl.vue) & [NavItem.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ds/NavItem.vue)
Navigation primitives for mode switching and sidebar items with desaturated active indicator fills.

---

### 3. Application Shell & PrimeVue Overrides

#### [MODIFY] [package.json](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/package.json)
Install `lucide-vue-next` for type-safe Vue 3 icon components.

#### [MODIFY] [main.js](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/main.js)
Import global Latent Design System stylesheet (`/assets/css/latent/styles.css`) and register global DS components.

#### [MODIFY] [App.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/App.vue)
Integrate the 52px frameless `Titlebar.vue` at top, fixed sidebar (`FolderNav.vue`), and scrollable content workspace with flat radial dark canvas background (`--color-bg-canvas`).

#### [MODIFY] [FolderNav.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/FolderNav.vue)
Restyle navigation sidebar (~200–224px width) using `NavItem`, search/filter inputs, folder tree icons (Lucide), and developer footer credit (`alx_logo.png`).

#### [MODIFY] [primevue-overrides.css](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/assets/css/components/primevue-overrides.css)
Update PrimeVue component overrides (Toast, Dialog, ContextMenu, Dropdown) to consume Latent Design System tokens instead of hardcoded hex codes.

---

### 4. Views & Feature Component Overhauls

#### [MODIFY] [BrowserToolbar.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/BrowserToolbar.vue) & [ImageBrowserView.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/views/ImageBrowserView.vue)
Restyle top toolbar with `LInput`, `LSelect`, filter `LButton`, and virtual grid image cards (`ImageCard.vue`) with hairline borders and desaturated star ratings.

#### [MODIFY] [MetadataSidebar.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/MetadataSidebar.vue) & [TaggerSidebar.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/TaggerSidebar.vue)
Update sidebar drawer panels to `--color-surface-1` with 1px `--color-border-subtle`, code block formatting in JetBrains Mono (`--font-mono`), and `LSwitch` / `LSlider` for ONNX interrogator controls.

#### [MODIFY] [CollectionsView.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/views/CollectionsView.vue)
Restyle collection cards using the 3D-stacked card effect (`transform: translate(6px, -6px)`), `LBadge` for Smart/Static tags, and brand gradient CTA for collection creation.

#### [MODIFY] [ComparatorView.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/views/ComparatorView.vue) & [ImageSplitViewer.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/components/ImageSplitViewer.vue)
Restyle image comparison view with `SegmentedControl` mode selector, custom slider handle, and metadata diff panel.

#### [MODIFY] [SpeedSorterView.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/views/SpeedSorterView.vue)
Restyle speed sorter hotkey badges, target folder destination cards, and trash action buttons using `LButton` and `LBadge`.

#### [MODIFY] [DuplicateDetectiveView.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/views/DuplicateDetectiveView.vue) & [ScrubView.vue](file:///c:/Users/error/IdeaProjects/Projects/Latent-Library/frontend/src/views/ScrubView.vue)
Restyle duplicate cluster cards, similarity score badges, and privacy scrubbing options (`LCheckbox`) with unified DS token styles.

---

## Verification Plan

### Automated Tests
- Run Vue frontend build to verify zero compile or styling errors:
  ```bash
  cd frontend && npm run build
  ```
- Run backend tests to verify API endpoints remain intact:
  ```bash
  cd backend && ./mvnw test
  ```

### Manual Verification
- **Visual Inspection**: Verify dark-mode canvas background (`#0A0A0D`), hairline borders, desaturated cyan/violet accents, frameless Titlebar drag region, and Lucide icons across all views (Gallery, Collections, Comparator, Speed Sorter, Duplicate Detective, Scrubber).
- **Responsive Layout**: Test window resizing to ensure 52px titlebar, fixed sidebar, and virtual gallery grid scale cleanly.
