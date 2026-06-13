# Changelog

## [1.1.2](https://github.com/dm3yb/vscode-theme-toggle/compare/v1.1.1...v1.1.2) (2026-06-13)


### Bug Fixes

* publish to Open VSX ([d40101f](https://github.com/dm3yb/vscode-theme-toggle/commit/d40101fdea4c588f5228e1de33b6e756d57b6baf))
* publish to Open VSX ([11ba0f3](https://github.com/dm3yb/vscode-theme-toggle/commit/11ba0f3a7efffbd0f8bf0cb7018b75492571111a))

## [1.1.1](https://github.com/dm3yb/vscode-theme-toggle/compare/v1.1.0...v1.1.1) (2026-06-13)


### Bug Fixes

* publish to the VS Code Marketplace ([3e90aa7](https://github.com/dm3yb/vscode-theme-toggle/commit/3e90aa74157c4d608f308798c270a2910b2fd038))

## [1.1.0](https://github.com/dm3yb/vscode-theme-toggle/compare/v1.0.0...v1.1.0) (2026-06-13)


### Features

* add esbuild configuration and VSCode setup for extension development ([541963e](https://github.com/dm3yb/vscode-theme-toggle/commit/541963ecc3fead617d3c08498aeb99192b21ed86))
* add new media assets including icon images and theme toggle font ([f3fc60e](https://github.com/dm3yb/vscode-theme-toggle/commit/f3fc60e50cdbfd624b5587d3f35921ca763ddccd))
* add Vitest configuration and initial tests for schedule functionality ([1378104](https://github.com/dm3yb/vscode-theme-toggle/commit/1378104e0b3d2e506e2f6b6a5ef2898093bf5024))
* implement core functionality for theme toggling with command palette integration ([7176d8b](https://github.com/dm3yb/vscode-theme-toggle/commit/7176d8ba09925b4378f6da84cf29126f74475f41))

## 1.0.0 — 2026-06-14

Initial release.

- Four switching modes: **Manual Toggle**, **System Preference**, **Sunrise / Sunset**, and **Manual Schedule** (with inline time prompts).
- Configurable **light** and **dark** themes, picked from any installed theme.
- **Status bar** item with custom sun/moon icons (bundled icon font) and **Command Palette** commands.
- Optional notification when the theme changes automatically; master enable/disable switch.
- Sunrise/sunset computed locally with `suncalc` from manual coordinates or a one-time, day-cached IP geolocation fallback.
- Works in VS Code and VS Code–based editors (Cursor, Windsurf, VSCodium).
