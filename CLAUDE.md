# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CalmCPAP is a privacy-first CPAP data visualization tool that runs entirely in the browser. It parses ResMed AirSense 11 SD-card ZIP exports and renders interactive charts for therapy signals (pressure, leak, flow, respiratory rate, SpO2, events, etc.). No backend, no accounts, no data leaves the browser.

## Architecture

**Single-file application**: The entire app lives in `index.html` (~5700 lines, ~219 KB) — HTML, CSS, and vanilla JavaScript with no build system or framework. External dependencies are loaded via CDN:
- **jszip@3.10.1** — ZIP extraction
- **plotly-2.30.0** — interactive charting

**State management**: A global `state` object holds the loaded ZIP, parsed metadata, indexed sessions (`sessionsByDate` Map keyed by ISO date), and the current session's signals/events/stats.

**Data pipeline**:
1. ZIP upload → `handleZip()` extracts entries via jszip
2. `readIdentification()` / `readCurrentSettings()` parse device JSON metadata
3. `indexSessionsFromSDStructure()` scans `DATALOG/YYYYMMDD/*.edf` folders, parses EDF headers to build session index
4. `readSelectedSignalsFromEDF()` parses binary EDF format, maps signal labels to 10 signal families, scales to physical units
5. Plotly renders interactive traces per signal tab

**EDF parsing**: The app reads European Data Format (EDF) binary files using ArrayBuffer/DataView. Signal label normalization (`normalizeLabel`) maps vendor-specific labels to canonical signal keys.

**10 signal families**: leak, pressure, flow, snore, respRate, tidalVolume, epr, pulse, spo2, annotations — defined in `WORKFLOW_SIGNAL_DEFS`.

**9 data source families**: PLD (primary 2s metrics), BRP (waveform), EVE/CSL (events), SA2 (pulse/oxygen), STR.edf (summary), plus JSON metadata — defined in `SOURCE_FAMILY_DEFS`.

**Event types**: Obstructive Apnea (10), Hypopnea (11), Central Apnea (12), Unclassified Apnea (13), RERA (14), Device Respiratory Event (16), Arousal, plus recording markers.

## Running the App

Open `index.html` directly in a browser, or use a local server:
```bash
python -m http.server 8000
```

**Demo mode** via URL hash parameters:
```
#demo=1&demoSet=1&skipOnboarding=1&tab=overview
```
- `demo=1` — load embedded synthetic data
- `demoSet=1..5` — select specific demo dataset (omit for random)
- `skipOnboarding=1` — bypass intro modal
- `tab=overview|pressure|leak|flow|snore|resprate|tidalvolume|epr|events|compare`

**Sample real data**: `SampleTestData/jan_2026_3days.zip` (ResMed SD-card export)

## Screenshot Automation

```powershell
powershell -ExecutionPolicy Bypass -File scripts\capture-marketing-screenshots.ps1
```
Captures the 4 README marketing screenshots (overview, pressure, compare, events tabs) using headless Edge/Chrome.

## Key Conventions

- No build step, no package manager, no transpilation — edit `index.html` directly
- All CSS uses custom properties (variables) for theming; dark theme is default
- URL hash-based routing controls active tab and demo state
- 5 embedded demo datasets use a seeded PRNG (Mulberry32) for reproducible synthetic data
- Git branches: `main` (stable), `dev` (active development)
- Licensed under GNU GPLv3
