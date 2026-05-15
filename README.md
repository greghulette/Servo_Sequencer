# Servo Sequencer

Browser-based piano-roll editor for authoring R2-D2 panel/servo sequences. One HTML file, no build step, no dependencies.

## What it does

- **Piano-roll grid** for two boards: Body Servo (BS, 13 servos — utility arms, doors, drawers, CPU arm) and Dome Controller (DC, 10 servos — dome panels, pie panels).
- **Drag-paint editing** with named **mask groups** matching firmware `#define`s (`ALL_SERVOS_MASK`, `CPU_SERVOS`, `LARGE_DOORS`, etc.).
- **Audio analysis** — BPM detection (autocorrelation on the energy envelope), waveform display, beat markers, playhead.
- **Auto-generate from audio** — radix-2 FFT per beat, four frequency bands (sub/low/mid/high) mapped to servo groups, sensitivity slider.
- **One-click deploy** into a sibling [`Arduino-Code`](https://github.com/greghulette/Arduino-Code) repo via the File System Access API:
  - Adds `Seq<name>BS` / `Seq<name>DC` arrays to `libraries/Reeltwo/src/ServoSequencer.h`.
  - Injects a wrapper function + dispatch case into `Body_Servo_Controller/Body_Servo_Controller.ino` and `Dome_Controller/Dome_Controller.ino` (between stable marker comments).
  - Persists a reloadable project to `sequences/<name>.json` plus a `sequences/_manifest.json` index.

## Usage

Open `sequencer_tool.html` in Chrome or Edge 86+ — the deploy pipeline needs the File System Access API.

First **🚀 Apply to Controllers** click prompts you to pick the root of your `Arduino-Code` project folder. The handle is cached in IndexedDB, so subsequent deploys are one-click.

Layout-only preferences (sidebar width, export-panel height) persist in `localStorage`.

## Function index allocation

Auto-injected wrappers are assigned indices **28–97**. Indices **1–27** and **98–99** are reserved for hand-written cases. The 2-digit ceiling is a controller-parser limitation: both `.ino` files read `inputBuffer[3]-[4]` as the function index. Lifting it requires patching the parser in both controllers first.

Re-deploying an existing sequence keeps its previously assigned index (tracked in `_manifest.json`).

## Project layout the tool expects

```
<your Arduino-Code root>/
├── Body_Servo_Controller/
│   └── Body_Servo_Controller.ino
├── Dome_Controller/
│   └── Dome_Controller.ino
├── libraries/Reeltwo/src/
│   └── ServoSequencer.h
└── sequences/
    ├── _manifest.json        ← function-index registry
    └── <name>.json           ← per-sequence reloadable project
```
