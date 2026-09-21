# Breathwork Studio — README

A single‑file, offline‑friendly web app for guided breathwork. It includes eight research‑backed techniques, an auto‑sequenced Smart Session, and a local analytics dashboard — all running in the browser with no build step.

---

## Overview

Breathwork Studio is designed for calm, focus, and energy regulation. It provides real‑time visual and audio guidance for each breathing pattern, logs every session locally via IndexedDB, and offers a hands‑free Smart Session that builds a complete routine based on your chosen duration.

It is TV‑remote friendly, supports keyboard shortcuts, and defaults to a light theme with ambient drone on and phase chime off.

---

## Features

### 8 Breathing Techniques

| # | Name | Pattern | Description |
|---|------|---------|-------------|
| 1 | **Box Breathing** | 4 · 4 · 4 · 4 | Navy SEAL method for calm under pressure. |
| 2 | **Triangular Breathing** | 4 · 4 · 4 | Three‑sided rhythm for steady, spacious awareness. |
| 3 | **4‑7‑8 Breathing** | 4 · 7 · 8 | Dr. Weil’s relaxing breath — the natural tranquilizer. |
| 4 | **Resonant Breathing** | 5 · 5 | Slow, coherent breathing at ~5.5 breaths per minute. |
| 5 | **Physiological Sigh** | 2 · 1 · 6 | Fastest known way to reset stress — in one breath. |
| 6 | **Cyclic Hyperventilation** | 30× · Hold | Energizing breaths followed by a deep retention hold. |
| 7 | **Alternate Nostril** | In L · Ex R · In R · Ex L | Balances the left and right hemispheres. |
| 8 | **Kapalabhati** | Rapid Exhale | Rapid, forceful exhales for mental clarity. |

Each card includes benefits, a “how it works” explanation, and a Start button.

### Smart Session

- Choose a duration: **5, 10, 15, or 20 minutes**.
- The app automatically builds a scientifically sound routine using a weighted role‑based algorithm.
- Roles: `settle`, `energize`, `focus`, `calm`, `close`.
- Transitions between techniques happen automatically — no user input required.
- A progress bar shows the current block and overall time.

### Analytics Dashboard

- Logs every session (≥ 15 seconds) locally in **IndexedDB**.
- Displays:
  - All‑time total minutes/hours breathed.
  - Total sessions and average session length.
  - Current day streak.
  - 14‑day bar chart of daily practice time.
  - Time spent per technique (horizontal bars).
  - Recent session history with date, duration, and techniques used.
- Data is stored only on your device. A “Clear data” button wipes the local database.

### Settings & Customization

- **Theme**: Light (default) / Dark.
- **Language**: English / Hindi.
- **Audio**:
  - Phase chime (default **off**).
  - Ambient drone (default **on**).
  - Volume slider (default 55%).
- **Visual**: Frame border toggle (default on).

### YouTube Subscribe Bar

A floating, smooth‑animated button at the bottom links to the channel:  
[https://youtube.com/@radhe.jaiguru](https://youtube.com/@radhe.jaiguru?si=t7Y9O6IFvjOyD_y2)

---

## Getting Started

1. Download or copy the single HTML file (`index.html`).
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari).
3. No server, build tools, or dependencies required — everything is self‑contained except for Google Fonts and Tailwind CDN (optional, but included via CDN for styling).

> **Note:** For full audio and IndexedDB support, make sure you are not in a restrictive private browsing mode.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Space` | Play / Pause; release an open breath hold |
| `0` | Back (also works as TV remote back) |
| `1` – `8` | Switch to technique 1–8 |
| `R` | Toggle recording mode |
| `S` | Open / close Settings |
| `D` | Open / close Dashboard |
| `L` | Toggle language (English / Hindi) |
| `B` | Toggle frame border |
| `Esc` | Exit recording, close panels, or go back |
| Arrow keys | Move focus between cards (TV remote friendly) |

---

## Smart Session Logic

The planner uses a weighted role system:

1. **Block count** is determined by total duration:
   - ≤ 5 min → 3 blocks
   - ≤ 10 min → 4 blocks
   - > 10 min → 5 blocks

2. **Roles and weights** (for 5 blocks):
   - `settle`: 12%
   - `energize`: 20%
   - `focus`: 22%
   - `calm`: 28%
   - `close`: 18%

3. **Technique selection**: For each role, a technique is chosen from a predefined pool, avoiding repeats when possible.

4. **Cycle quantization**: The target seconds for each block are converted into a whole number of cycles for the chosen technique. The algorithm ensures the last block fills exactly the remaining time, and trims overshoot from earlier blocks if needed.

5. **Seamless transition**: When a block completes its cycles, the app automatically switches to the next technique, resets the phase machine, and continues without pausing.

This guarantees a balanced, time‑accurate session that progresses from settling to energizing, focusing, calming, and closing.

---

## IndexedDB Schema

- **Database**: `breathwork-studio`
- **Version**: `1`
- **Object Store**: `sessions`
  - Key path: `id` (auto‑increment)
  - Indexes:
    - `dateKey` (string, e.g. `"2025-03-21"`)
    - `startedAt` (timestamp)
    - `mode` (`"single"` or `"smart"`)

### Session Record Structure

```js
{
  id: 1,
  startedAt: 1711046400000,      // Unix timestamp (ms)
  dateKey: "2025-03-21",         // YYYY-MM-DD
  durationSec: 312,              // total seconds
  mode: "smart",                 // "single" or "smart"
  completed: true,               // did the session finish naturally?
  techniques: [
    { id: "box", name: "Box", seconds: 120 },
    { id: "478", name: "4-7-8", seconds: 192 }
  ]
}
```

If IndexedDB is unavailable (e.g., private mode), the app falls back to an in‑memory array so the dashboard still works within the session, but data is not persisted.

---

## Technical Details

- **Single HTML file** with embedded CSS and JavaScript.
- **Animation engine**: SVG‑based ball, rings, and progress tracks. Each technique defines its own HUD layout, ring radii, and base path.
- **Phase engine**: Handles timed phases, open‑ended holds (waiting for Space/tap), and rapid techniques (short durations, no HUD countdown).
- **Audio engine**: Web Audio API. Chime uses a multi‑oscillator bell; drone is a filtered noise + sine oscillator that modulates with breath phase.
- **State management**: A central `state` object tracks language, theme, playback, smart session progress, and session logging accumulators.
- **History integration**: Uses `history.pushState` so browser back buttons (and TV remotes) behave as expected.

---

## Customization

### Adding a New Technique

1. Open the `TECHNIQUES` array in the script.
2. Add a new object with the required fields:
   - `id`, `name`, `accent`, `accent2`, `icon`, `short`
   - `en` and `hi` objects containing `name`, `tagline`, `desc`, `benefits`, `how`
   - `hud`, `rings`, `base`, and `phases` arrays
3. Optionally set `rapid: true` and `perPhaseOnly: true` for fast patterns.
4. Rebuild the page — the card and scene will be generated automatically.

### Changing Defaults

In the `init()` function you can change:

- Default theme: `setTheme('light')` → `setTheme('dark')`
- Default chime: `state.chime = false` → `true`
- Default drone: `state.drone = true` → `false`
- Default volume: `state.volume = 0.55`

### Styling

All styles are in the `<style>` block. CSS variables control phase colors, track colors, and backgrounds. You can override them in `:root` or per‑theme (`#stage.theme-light`, `#stage.theme-dark`).

---

## Browser Support

- Chrome / Edge / Brave (latest)
- Firefox (latest)
- Safari (latest)
- Opera (latest)

Requires:
- IndexedDB (falls back to memory if blocked)
- Web Audio API (audio features degrade gracefully)
- CSS Grid and Flexbox

---

## Credits

- Built as a single‑file web app.
- Fonts: Outfit, Inter, Tiro Devanagari Hindi (Google Fonts).
- Tailwind CSS via CDN for utility classes.
- YouTube channel: [@radhe.jaiguru](https://youtube.com/@radhe.jaiguru?si=t7Y9O6IFvjOyD_y2)

---

## License

This project is provided as‑is for personal and educational use. You are free to modify and adapt it for your own needs.

---

*Breathe with intention.*
