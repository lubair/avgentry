# AVG Entry

Stock average-entry calculator. Static site, no build step, no dependencies.

- Repository: https://github.com/lubair/avgentry (`origin`, branch `main`)
- Production: https://lubair.github.io/avgentry/

## Project structure

- `index.html` — the entire app: one `<style>` block (design tokens on `:root`, dark mode via `@media (prefers-color-scheme: dark)`), the markup, and one `<script>` IIFE (`"use strict"`, ES5-style `var`/`function`).
- `stocks.json` — ticker directory (`{ popular, stocks: [[ticker, name], ...] }`), loaded with `fetch("stocks.json")` for the stock picker.

To run locally, serve the folder over HTTP (e.g. `python3 -m http.server`) — `fetch` of `stocks.json` fails from `file://`.

## Deployment

GitHub Pages, legacy build from `main` at `/`. Pushing to `origin/main` publishes; there is no CI or build. Commits use the existing local Git configuration.

## Data and storage

- Current key: `trade-calculator-v4` (`{ version: 4, activeId, stocks, history }`). Only v4 is written.
- `trade-calculator-v3`, `-v2`, `-v1` are read as fallbacks and never modified or deleted.
- `parseState`/`readRows`/`readHistory` sanitize everything loaded. Any new field must be optional, default safely when absent, and keep older saved data loading unchanged. Never rename or drop existing keys or fields.
- All numeric inputs are stored as the raw strings the user typed; they are parsed at calculation time.

## Calculation model (do not change without explicit approval)

- `purchase()`: shares mode → `shares × price`; amount mode → `amount / price` shares. Cost basis = gross + commission. Blank commission = $0.
- The original order is the oldest lot; additional buys and sales replay in `seq` order (`replay()`).
- Cost basis is **FIFO**: sales consume the oldest lots first, partial lots give up cost pro rata. Realized P/L = net proceeds (shares × price − commission) − consumed FIFO cost.
- A sale's displayed estimate is priced at the average cost before the sale (display only; never feeds the lots).
- `SHARE_TOLERANCE = 0.00005`: selling within the displayed 4 decimals of all shares closes the position.
- A sale exceeding held shares stops the replay; results after it are unknown (shown as —).
- Formatting: money 2 decimals, prices/shares up to 4 decimals. Round only at display time, never in intermediate math.

## Rules

### Architecture
- Keep vanilla HTML/CSS/JS in the existing single-file structure. No frameworks, libraries, build tools, or dependencies without approval.
- Keep the app lightweight, fast, and responsive.
- Match the existing code conventions (ES5 `var`/`function`, `$()` helper, concise comments that explain *why*).
- Preserve localStorage data and backward compatibility (see above).

### Design
- Use and preserve the existing CSS custom properties; both light and dark values must be kept in sync for any new token.
- Maintain light and dark mode, mobile and desktop layouts (breakpoints at 419/599/600/720px, `pointer: coarse`, `prefers-reduced-motion`).
- Keep green/red trading colors (`--up`/`--accent`, `--down`) and existing spacing.
- Do not redesign or refactor unrelated components.

### Calculation accuracy
- Never silently change formulas or financial assumptions; if a request would, say so and ask first.
- Maintain accurate average entry, share quantity, commission, cost basis, and realized/unrealized P/L calculations, including fractional shares, additional buys, sales, and FIFO basis, as currently implemented.
- Avoid introducing rounding or floating-point drift.

### Workflow
- Implement only what is requested; keep diffs minimal and focused.
- After a change, verify it in a browser (light and dark, mobile and desktop widths as relevant) and check affected calculations with concrete numbers.
- Avoid unnecessary refactoring, tests, documentation, experiments, or prototypes.
- Fix any errors your change introduces.

### Security
- This repository and its Pages site are public: never put passwords, API keys, tokens, or other credentials in any file.

### Git and publishing
- Preserve the existing GitHub Pages deployment workflow. Never change hosting providers or deployment architecture without approval.
- **Application implementation tasks:** publishing is part of the default workflow. After completing and validating the change, use the existing Git configuration to commit the relevant files and push to `origin/main`.
- Never publish broken or unverified implementation changes.
- Use short, imperative commit messages in the existing style (e.g. "Clarify sale profit and loss display").
- After pushing, confirm the Pages build succeeded and the live site reflects the change. If deployment fails, report the failure; never claim the site was updated.
- Never force-push, rewrite Git history, or perform destructive Git operations.
- Never create branches or pull requests unless explicitly requested.

### When not to publish
- **Setup tasks, audits, documentation changes, investigations, and read-only requests:** do not commit, push, or deploy unless explicitly requested, and do not modify application code unnecessarily.
- **Explicitly local-only requests:** do not commit, push, or deploy.
- Report implementation and publication results concisely.
