# AGENTS.md

## Overview
This repo is a Chrome extension that injects custom CSS to darken Canvas. Most of the work is in `plugin/css/styles.css`. Canvas uses a mix of stable semantic classes (e.g. `ic-*`, `ui-*`) and dynamically generated/hashed classes (e.g. `css-1a2b3c-...`). The goal is to avoid brittle selectors and keep the rules resilient across Canvas updates.

## Core Principles
1. **Avoid hashed classnames.**
   - Classes like `.css-15ffo05-textInput__facade` are generated and change frequently.
   - If a selector depends on these hashes, it will break quickly.

2. **Prefer stable hooks.**
   - `data-testid`, `data-automation`, `role`, `aria-*`, IDs, and Canvas semantic classes (`.ic-*`, `.ui-*`) are preferred.
   - Stable container classes (e.g. `body.files`, `body.people`, `.navigation-tray-container.courses-tray`) are ideal for scoping.

3. **If you must match obfuscated classes, match by *suffix* with strict scoping.**
   - InstUI/Emotion often keeps a readable suffix (e.g. `textInput__facade`, `baseButton__content`, `view--block-list`).
   - Use substring selectors with tight scope to reduce collateral styling.
   - Example:
     ```css
     body.files [class*="textInput__facade"] { ... }
     .navigation-tray-container.courses-tray [class*="view-listItem"] { ... }
     ```

4. **Scope aggressively.**
   - Use page-level or container-level qualifiers (`body.files`, `.courses-tray`, `#dashboard-planner-header`) to avoid global side effects.
   - Avoid generic selectors like `[class*="text"]` without a tight container.

5. **Prefer variables where possible.**
   - Canvas exposes theme variables (see `:root` overrides at the top of `plugin/css/styles.css`).
   - If a color can be expressed via variables, do that instead of adding many per-component overrides.

## Workflow for Fixing UI Issues
1. **Inspect the DOM**
   - Identify the element that needs styling.
   - Find any stable parent containers (`body.*` classes, tray containers, IDs, etc.).

2. **Find stable selectors first**
   - Check for `data-testid`, `role`, `aria-*`, or well-known Canvas classes.

3. **Fallback to suffix-based substring selectors**
   - If only hashed classes exist, match the suffix:
     - `textInput__facade`
     - `baseButton__content`
     - `view--block-list`
   - Always scope these to a stable container to avoid bleeding.

4. **Add or adjust rules in `plugin/css/styles.css`**
   - Keep related rules grouped with short comments describing the target area.

5. **Verify visually**
   - Open Canvas pages affected by the change.
   - Check both light-on-dark contrast and hover/active states.

6. **Commit changes**
   - Commit only the tracked extension files (e.g. `plugin/css/styles.css`).
   - Do **not** commit any ad hoc or scraped files (e.g. a temporary top-level `css/` directory).

## Patterns That Work Well
- **Text inputs (InstUI):**
  ```css
  .some-container [class*="textInput__facade"] { ... }
  .some-container [class*="textInput__layout"] { ... }
  .some-container input[role="combobox"] { ... }
  ```

- **Buttons (InstUI):**
  ```css
  .some-container [class*="baseButton__content"] { ... }
  .some-container [class*="baseButton__children"] { ... }
  ```

- **Lists / list items:**
  ```css
  .some-container [class*="view--block-list"] { ... }
  .some-container li[class*="view-listItem"] { ... }
  ```

## Things to Avoid
- Directly matching `.css-<hash>` classes without a stable suffix.
- Global substring selectors like `[class*="text"]` without a tight container.
- Styling unrelated UI surfaces because the selector is too broad.

## Current Hotspots
- **Courses tray**: `.navigation-tray-container.courses-tray` is the main stable scope.
- **Files page**: `body.files` is reliable for scoping Inputs, tables, and controls.
- **Planner/Grades**: Use page bodies and stable containers to avoid global bleed.

## Commit Notes
- Keep commits small and descriptive (e.g. “Fix courses tray subtext color”).
- Avoid committing any scraped CSS files or temporary directories.
