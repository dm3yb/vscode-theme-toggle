# Theme Toggle

Automatically switch between a **light** and a **dark** color theme — by sunrise/sunset, your operating system's appearance, a custom schedule, or on demand. Works in VS Code and VS Code–based editors (Cursor, Windsurf, VSCodium).

<!-- TODO: add a short demo GIF of the status bar toggle and the mode picker, e.g.:
![Theme Toggle demo](media/demo.gif)
-->

## Requirements

- VS Code **1.74** or later (or a compatible fork: Cursor, Windsurf, VSCodium).
- At least one light theme and one dark theme installed (the built-in *Default Light/Dark Modern* work out of the box).

## Installation

**VS Code** — install from the Visual Studio Marketplace: open the Extensions view (`Cmd/Ctrl+Shift+X`), search **Theme Toggle**, and click Install.

**Cursor / Windsurf / VSCodium** — these forks use [Open VSX](https://open-vsx.org/); search **Theme Toggle** in the Extensions view, or install the `.vsix` directly.

**From a `.vsix` file** (any editor):

```bash
code   --install-extension theme-toggle-<version>.vsix   # VS Code
cursor --install-extension theme-toggle-<version>.vsix   # Cursor
```

## Quick start

1. **Pick your themes.** Open the Command Palette (`Cmd/Ctrl+Shift+P`) and run **Theme Toggle: Select Light Theme**, then **Theme Toggle: Select Dark Theme** — choose from any installed theme.
2. **Toggle.** Click the **☀️ / 🌙** item in the status bar (bottom-right), or run **Theme Toggle: Toggle Light/Dark Now**.
3. **Go automatic (optional).** Run **Theme Toggle: Select Switching Mode** and choose System Preference, Sunrise / Sunset, or Manual Schedule.

That's it — out of the box the mode is **Manual Toggle**, so the extension never changes your settings until you ask it to.

## Switching modes

Choose one via **Theme Toggle: Select Switching Mode**:

- **Manual Toggle** — switches only when you click the status bar item or run the toggle command.
- **System Preference** — follows your OS light/dark appearance (delegates to VS Code's built-in `window.autoDetectColorScheme`).
- **Sunrise / Sunset** — light during the day, dark at night, from your local sunrise/sunset times.
- **Manual Schedule** — switches at times you choose. Selecting this mode prompts you for the light and dark times inline (e.g. light at 07:00, dark at 19:00); you can also edit them later in Settings.

Toggle automatic switching on/off at any time with **Theme Toggle: Enable/Disable Automatic Switching**, or show a toast on each automatic change via `themeToggle.showNotifications`.

## Commands

| Command                                            | Description                                          |
| -------------------------------------------------- | ---------------------------------------------------- |
| `Theme Toggle: Toggle Light/Dark Now`              | Flip between your light and dark themes immediately. |
| `Theme Toggle: Select Switching Mode`              | Choose how the theme is decided.                     |
| `Theme Toggle: Select Light Theme`                 | Pick the theme used for light mode.                  |
| `Theme Toggle: Select Dark Theme`                  | Pick the theme used for dark mode.                   |
| `Theme Toggle: Enable/Disable Automatic Switching` | Master switch for automatic modes.                   |

## Settings

| Setting                          | Default                | Description                                                   |
| -------------------------------- | ---------------------- | ------------------------------------------------------------- |
| `themeToggle.lightTheme`         | `Default Light Modern` | Theme for light mode.                                         |
| `themeToggle.darkTheme`          | `Default Dark Modern`  | Theme for dark mode.                                          |
| `themeToggle.mode`               | `manualToggle`         | Active switching mode.                                        |
| `themeToggle.enabled`            | `true`                 | Master switch for automatic switching.                        |
| `themeToggle.showNotifications`  | `false`                | Notify on automatic theme change.                             |
| `themeToggle.latitude`           | `null`                 | Latitude for sunrise/sunset (auto-detected via IP if empty).  |
| `themeToggle.longitude`          | `null`                 | Longitude for sunrise/sunset (auto-detected via IP if empty). |
| `themeToggle.schedule.lightTime` | `07:00`                | Time to switch to light (Manual Schedule). 24-hour `HH:MM`.   |
| `themeToggle.schedule.darkTime`  | `19:00`                | Time to switch to dark (Manual Schedule). 24-hour `HH:MM`.    |

## Sunrise / sunset & privacy

Sunrise and sunset times are computed **locally on your machine** with [`suncalc`](https://github.com/mourner/suncalc) — they need only your latitude/longitude and the date, and make no network calls.

To get your coordinates, the extension uses, in order:

1. `themeToggle.latitude` / `themeToggle.longitude`, if you set them — **fully offline, nothing leaves your machine.**
2. Otherwise, a **one-time IP-based geolocation lookup** to `ipapi.co`, cached for the rest of the day. This sends your IP address to that service. To avoid it entirely, set your coordinates manually.

No telemetry or analytics are collected.

## Troubleshooting

- **The theme isn't switching automatically.** Make sure automatic switching is on (`themeToggle.enabled`) and you're not in **Manual Toggle** mode. In **System Preference** mode, switching is driven by your OS appearance setting via VS Code's `window.autoDetectColorScheme`.
- **Sunrise / Sunset does nothing.** It needs a location. Set `themeToggle.latitude` / `themeToggle.longitude`, or ensure the one-time IP lookup can reach the network.
- **My manual theme keeps getting overridden.** An automatic mode is active and re-applying its theme. Switch to **Manual Toggle** or disable automatic switching.
- **See what happened.** Open the **Output** panel (`Cmd/Ctrl+Shift+U`) and select **"Theme Toggle"** from the dropdown — it logs every theme change and mode switch.

## License

MIT — see [LICENSE](LICENSE).
