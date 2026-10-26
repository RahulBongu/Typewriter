# THE IMPACT No. 01 — Mechanical Typewriter

------- OPEN IN LOCAL HOST AND VIEW IN FULL SCREEN FOR BETTER EXPERIENCE--------
A single-file, browser-based 3D mechanical typewriter simulator. Every key, typebar, carriage step, bell, ribbon spool and platen roll is modelled and animated in Three.js; all sound is synthesized with the Web Audio API; documents stay on the user's machine and export to DOCX. Built for writers, tinkerers and anyone who wants the feel of a mechanical typewriter without the hardware.

![THE IMPACT No. 01](/docs/screenshot-write.png)
*Write view: the keyboard, platen, carriage and virtual paper with typed text.*

![Mechanism view](/docs/screenshot-mechanism.png)
*Mechanism view: the typebar basket, escapement and carriage exploded for inspection.*

![Full screen](/docs/screenshot-fullscreen.png)
*Full-screen immersive mode with the centered full-screen toggle.*

## Features

**3D machine and typing**
- Four-row round-rimmed keyboard (43 keys), individual typebars, platen, carriage, paper bail, ribbon spools and vibrator, spacebar, shift, return lever and bell, built from primitives.
- Real keypress animation: key depress, typebar strike through the ribbon, carriage step left; bell near the right margin; carriage return slides back and feeds a line.
- Virtual paper with per-strike ink jitter, monospace type, margins, line spacing, automatic page breaks; backspace moves the carriage without erasing; overstrike allowed.

**Paper and themes**
- Three paper themes — cream, blueprint, carbon — with matching ribbon/ink/grain; switchable without losing content; DOCX font color matches the theme.

**Camera views**
- Four views: Write, Mechanism, Inspect (hover labels), Sound. Guided 5-step tour of keyboard, typebars, escapement, carriage and ribbon.
- Explode and cutaway sliders, smooth and reversible. Collapsible side panels; full-screen immersive mode.

**Mechanism tools**
- Inspect hover labels; explode/cutaway; guided tour with camera fly-to, highlight and captions.

**Sound**
- Fully synthesized acoustics: key click, linkage knock, typebar strike, platen thud, rebound tick, carriage escapement, space thud, shift clunk, backspace tick, bell, return ratchet/slide/stop.
- Per-keystroke humanization (pitch, gain, timing jitter, per-key weight, stereo pan by carriage column, burst scaling), early reflection, voice cap.
- Paper sounds for Undo, Redo, Clear and New Sheet. Telegraph/Morse audio. Sound view with volume, reverb, mute, key intensity and live waveform.

**Export**
- One-click local DOCX export (multi-page, theme color, page breaks), no network upload or account.

## Quick start

Open `index.html` in a modern browser (double-click it). The page loads Three.js and JSZip from a CDN, so an internet connection is required on first load; the document itself stays local.

Optional local server:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

Audio is created lazily: the first key press or click calls `ensureAudio()` and resumes the AudioContext. Browsers block autoplay, so no sound plays until that first interaction.

## Controls and shortcuts

| Action | Input |
|---|---|
| Type character | Letter/number/punctuation keys |
| Space | Word space (all modes) |
| Enter | Carriage return |
| Tab | 4 spaces |
| Backspace | Move carriage left (no erase) |
| Undo | Ctrl/Cmd+Z |
| Redo | Ctrl/Cmd+Shift+Z (also Ctrl/Cmd+Y) |
| Export DOCX | Ctrl/Cmd+S or Export button |
| Reset view | Home or double-click canvas |
| Full screen | Alt+Enter or the centered toggle |
| Cancel / close | Esc |
| Tour step | Arrow Left / Arrow Right (during tour) |
| Orbit | Drag on the canvas |
| Zoom | Mouse wheel |
| Manual Morse keying | Shift+Space or press-and-hold the brass key |

Toolbar buttons: New Sheet, Undo, Redo, Clear, Export DOCX. Control desk: theme selector (cream/blueprint/carbon), telegraph toggle, WPM (5–30), explode/cutaway sliders, volume, reverb, mute, audition buttons, key intensity.

## Views and modes

- **Write** — straight-on front view of keyboard, typebar basket and paper.
- **Mechanism** — angled view of the typebar basket, escapement and carriage.
- **Inspect** — hover parts for labels.
- **Sound** — waveform, volume, reverb, mute, key intensity, audition buttons.
- **Guided tour** — 5 steps (keyboard, typebars, escapement, carriage, ribbon) with camera fly-to, highlight and captions; Next/Back/Exit; arrow keys; explode/cutaway auto-adjust and restore.
- **Explode / Cutaway** sliders — smooth, reversible.
- **Paper themes** — cream, blueprint, carbon; switchable without losing content.
- **Telegraph/Morse mode** — brass Morse key in 3D, typed text converted to Morse, Play Sheet, manual keying (Shift+Space / on-screen key), fold-out A–Z/0–9/punctuation chart with current letter highlighted.
- **Full screen** — immersive mode; top bar reveals within 40 px of the top edge; centered toggle enters/exits.

## Sound

The engine is Web Audio only — no external audio files. Oscillators, filtered noise (from a pre-generated buffer), biquad filters, gain envelopes and a synthesized reverb impulse. Each keystroke is 5 layered parts (key click, linkage knock, typebar strike with resonant partials, platen thud, rebound tick) with pitch/gain/timing humanization, per-key weight, stereo pan by carriage column and burst scaling. Carriage step is a two-tick escapement; space/shift/backspace have distinct sounds. Paper sounds (tear, crumple, slap, roller ratchet, slide, knob click) accompany Undo/Redo/Clear/New Sheet. Telegraph/Morse uses a sine tone. A `SOUND_BANK` object (empty by default) can hold base64 OGG/MP3 clips; if a group is non-empty it plays through the same humanizing chain.

To add your own licensed recordings, populate `SOUND_BANK` near the top of the audio section, e.g. `{ keys: ['data:audio/ogg;base64,...'], space: [...], ... }`. Clips should be short (under ~40 KB) and licensed for use.

## Export

Export DOCX produces a `.docx` (JSZip + OOXML) with Courier New 11 pt, exact line spacing (36 lines per page), multi-paragraph output, page breaks (including blank middle pages), and theme-matching font color. Files are named `THE-IMPACT-YYYY-MM-DD.docx`. Everything happens locally.

## How it works

- **Scene/machine groups:** `typewriter`, `baseGroup`, `shellGroup`, `keyboardGroup`, `basketGroup`, `carriageGroup`, `ribbonGroup`, `escapementGroup`, `bellGroup`, `telegraphGroup`, plus `desk` and lights.
- **Typebar/carriage animation:** per-frame `updateMechanism()` drives key travel, typebar swing, vibrator, spools, escapement, platen roll, return lever, bell, telegraph lever and page transitions; carriage X is damped toward `carriageTarget`.
- **Paper canvas texture:** a 1000×1300 (×2 `PAPER_SCALE`) canvas is drawn into a `CanvasTexture`; dirty-rect partial uploads via `copyTextureToTexture`, with full-upload fallback; `redrawPaper()` repaints all glyphs.
- **Paper feed and strike pin:** `paperFeedCurrent` damps toward `doc.line * LINE_STEP_WORLD`; the pin (fixed horizontally) dips on strike; glyph draw is deferred ~110 ms to the typebar contact moment.
- **Document model:** pages of 36 lines of cells (glyph arrays for overstrike); undo/redo via deep-cloned snapshots (max 120); counters and page turns.
- **Audio chain:** `ensureAudio` → master gain → compressor → analyser; dry/wet (convolver) reverb; `fxIn` feeds master.

## Customization

| Constant / setting | What it does | Default |
|---|---|---|
| `MACHINE_OFFSET_X` | Shifts the machine on the desk | `0.08` |
| `STRIKE_Y` | World height of the typing line | `1.0` |
| `PAPER_SCALE` | Paper canvas render scale | `2` |
| `CHAR_W` | Canvas column width (px) | `18` |
| `LINE_H` | Canvas line height (px) | `44` |
| `COLS` | Columns per line | `46` |
| `ROWS` | Lines per page (derived) | `24` |
| `BELL_COL` | Bell column | `40` |
| `CANVAS_W` / `CANVAS_H` | Paper canvas base size | `1000` / `1300` |
| `VIEW_PRESETS` | Camera presets per view | see code |
| `TELEGRAPH_PRESET` | Telegraph camera preset | see code |
| `SOUND_BANK` | Optional recorded clips | `{}` |
| `keyIntensity` | Keystroke layer scale | `1.0` |

## Project layout

Single file `index.html` (~4591 lines). Major sections: styles/CSS (1–400); HTML skeleton (400–620); constants and themes (620–700); document model and undo/redo (700–760); paper canvas and redraw (760–1000); document actions (1000–1100); Three.js init and materials (1100–1450); desk and typewriter build (1450–2230); views, camera, interaction (2230–2460); mechanism animation loop (2460–2870); audio engine (2870–3470); Morse/telegraph (3470–3700); UI wiring (3700–4130); boot (4130–end).

## Browser and device notes

- Requires WebGL (Three.js). No WebGL → loading error overlay.
- Fullscreen API used with `webkit`-prefixed fallback; CSS immersive fallback where unavailable (e.g. iPhone Safari) — untested on device.
- Mobile: on-screen textarea input, dock panels, Morse drawer.
- Audio requires a user gesture before starting (browser autoplay policy).

## Troubleshooting

- **Blank screen / WebGL errors:** check the console; the loader shows an error overlay if Three.js or JSZip fail to load from the CDN.
- **No sound:** audio starts only after the first key press or click; check mute and volume.
- **Text not showing:** verify the paper canvas dirty-rect path; `redrawPaper()` forces a full upload.
- **CDN blocked:** host the two pinned libraries locally and update the `<script src>` URLs.
- **Full screen not working:** browser may not support the Fullscreen API; the CSS immersive fallback still hides the chrome.

## LICENSE

<!-- TODO: fill in your license here (e.g. MIT). -->

## Known limitations / roadmap

- `SOUND_BANK` is empty by default; recorded clips must be added and licensed manually.
- iPhone Safari fullscreen uses a CSS fallback (untested on device).
- The paper texture is 2× but the dirty-rect path is the only upload route for typed glyphs; a full redraw covers any miss.
- No persistence: documents live in memory only. TODO: optional local save/load. TODO: add a print preview. TODO: verify mobile layout on real devices. TODO: add automated visual regression tests. (Items marked TODO are not implemented in the code.)
