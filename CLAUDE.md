# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Stack at a glance:** TypeScript · VS Code extension API · esbuild (CJS bundle) · pnpm · Node >= 20.12 · Vitest · Oxlint + Oxfmt · Husky · release-please. Ships to the VS Code Marketplace (vsce) and Open VSX (ovsx).

## Commands

| Command                         | Purpose                                                              |
| ------------------------------- | ------------------------------------------------------------------- |
| `pnpm test`                     | Run all unit tests (Vitest).                                        |
| `pnpm vitest run -t "name"`     | Run tests matching a name.                                          |
| `pnpm run check-types`          | `tsc --noEmit` (covers `src` and `test`).                           |
| `pnpm run lint`                 | Oxlint.                                                             |
| `pnpm run fmt` / `fmt:check`    | Oxfmt write / check (sorts imports; also sorts package.json keys).  |
| `pnpm run compile` / `watch`    | esbuild bundle to `dist/extension.js` (watch for the dev host).     |
| `pnpm run package`              | Build a `.vsix` (`vsce package --no-dependencies`).                 |

The Husky **pre-push** hook runs `fmt:check`, `lint`, `check-types`, and `test`; run all four before declaring work done. **CI** (`.github/workflows/ci.yml`) runs the same on every push to `main` and PR.

**Running it:** press **F5** to launch the Extension Development Host (uses `.vscode/launch.json`, which runs the esbuild `watch` task). The `vscode` API is only available there — not in Vitest.

**Releasing:** `release-please` (on merge to `main`) bumps the version from [Conventional Commits](https://www.conventionalcommits.org/) and writes `CHANGELOG.md`. Publishing the resulting GitHub Release triggers `.github/workflows/release.yml`, which runs `vsce publish` + `ovsx publish`. That step is **inert until** `package.json`'s `publisher` is a real id (not `your-publisher`) and `VSCE_PAT` + `OVSX_PAT` repo secrets are set.

## Architecture

**One activation entry** — [src/extension.ts](src/extension.ts) (`activate`) creates the logger, status bar, and controller, registers commands + two listeners (config change, active-theme change), pushes everything to `context.subscriptions` (so there is intentionally no `deactivate`), then calls `controller.refresh("activation")`.

**The controller is the only place that writes themes.** [src/theme-controller.ts](src/theme-controller.ts) (`createThemeController`) holds the timer/disposal/one-shot-warning state in a closure and exposes `refresh / toggleNow / onActiveThemeChanged / dispose`. `refresh()` is an **explicit `switch (config.mode)`** — there is deliberately no mode-strategy abstraction (an earlier version had a leaky one; explicit dispatch is simpler for four modes):

- `manualToggle` → does nothing automatic and touches no host settings (so installing the extension never changes the user's config); the toggle command flips it.
- `systemPreference` → delegates to VS Code: sets `window.autoDetectColorScheme` + the `workbench.preferred*ColorTheme` settings ([src/workbench.ts](src/workbench.ts)) and lets the editor follow the OS. Extensions cannot read the OS appearance directly, so we must delegate.
- `sunriseSunset` / `manualSchedule` → resolve a `{ kind, nextTransitionMs }` from the pure core, apply the theme, and arm a `setTimeout` capped at 1h (so sleep/DST/clock-drift recover on the next tick).

### The pure scheduling core (read before touching schedule logic)

[src/schedule.ts](src/schedule.ts) imports **no `vscode`** — only `suncalc` + `date-fns`. All inputs (now, coordinates, configured times) are passed in. This is what makes the tricky logic unit-testable in plain Node. `resolveSunriseSunset` falls back to the sun's altitude during polar day/night (no sunrise to wait for); `resolveSchedule` evaluates boundaries across yesterday/today/tomorrow so inverted/wrapping times work. Coordinate resolution is impure (network/`globalState`) and lives separately in [src/location.ts](src/location.ts); the controller resolves coords there and feeds them into the pure functions.

### Status bar icon font

The sun/moon glyphs are a bundled icon font at `media/theme-toggle.woff`, registered via `contributes.icons` in [package.json](package.json) and referenced as `$(theme-toggle-sun)` / `$(theme-toggle-moon)` (VS Code status bar labels accept icon ids, not raw SVG). The `.woff` is a **committed build artifact**; the one-off generator was removed, so regenerating the glyphs means restoring it from git history and re-adding `svgicons2svgfont` / `svg2ttf` / `ttf2woff`.

### Distribution

`engines.vscode` is kept conservative (`^1.74.0`) so the extension activates on forks (Cursor, Windsurf, VSCodium) that lag upstream. esbuild bundles to CJS (`external: ["vscode"]`); only `dist/`, `media/`, and the marketplace icon ship (see `.vscodeignore`).

## Testing conventions

- **Vitest only covers `vscode`-free modules** — in practice just `schedule.ts` (and its pure helpers). The `vscode` module does not exist in the Node test environment, so anything importing it cannot be unit-tested here.
- Tests inject `now`/coords/times as arguments and pin observable behavior (kind + `nextTransitionMs`, boundary instants, polar fallback, malformed-time fallback). Local-time `Date`s in tests match how the implementation builds boundaries, so wall-clock diffs are exact regardless of timezone.
- Anything touching the `vscode` API (controller, commands, status bar) is verified manually in the Extension Development Host (F5).

## Code rules

### Project-specific

- **Prefer popular packages over hand-rolled utilities** — `date-fns` for all date math (no manual `Date` arithmetic), `suncalc` for sunrise/sunset.
- **No `vscode` import in `src/schedule.ts`** — it must stay pure/testable. Impure concerns (settings, network, `globalState`, the output channel) live in `config.ts` / `location.ts` / `log.ts` / `workbench.ts`.
- **`config.ts` is the only place that reads/writes our `themeToggle.*` settings; `workbench.ts` is the only place that touches the host's `workbench.*` / `window.*` settings.** Keep the two namespaces separate.
- Formatting is Oxfmt's job (double quotes, semicolons, 100-col, sorted imports) — run `pnpm run fmt`, don't fix style by hand.

### TypeScript

- **No `any`** — use `unknown` + type guards (see `readCoordinates`/`isFiniteNumber` in `location.ts`).
- **No `as` assertions** in `src/` (`as const`/`satisfies` are fine) — narrow with guards instead.
- **No `interface`** — default to `type` (use `&` for composition).
- **No `enum`** — string-literal unions (`ThemeKind`, `SwitchMode`).
- **No `class`** — factory functions returning an object with closure state (`createThemeController`, `createStatusBar`).
- **No `export default`.** Export only what another module imports (e.g. `getInstalledThemes` is unexported — only `getThemesForKind` is public).
- **`function` keyword** for top-level declarations; **return types only on exported functions** — let TypeScript infer local ones (keep them on type-guard predicates).
- **kebab-case** filenames.
- **Comments are block-format only** — `/** … */` on declarations, `/* … */` elsewhere; no `//` lines. Each module opens with a short `/* … */` purpose header. Write a comment only for rationale the code can't express, never to restate it.
- **Deliberate exception to get-reflect's rules:** this is a CJS esbuild bundle (VS Code loads extensions as CommonJS), so relative imports do **not** use `.js` suffixes.

### Error handling

- Always `catch (err: unknown)`.
- Never swallow errors silently — log via the output-channel `log` ([src/log.ts](src/log.ts)) and comment why a recovery is safe (see `location.ts`'s stale-cache fallback on a failed IP lookup).
