# ScreenLogger

Periodic screen-scraping data logger for Windows. Reads a numeric value
displayed by any measurement software that has no export, logging or
communication interface – by taking screenshots of a user-defined screen
region and running OCR on it (or by copying the value via clipboard),
optionally clicking a button before each reading. Values are appended to
a CSV measurement log.

Typical use case: lab instrument software, vendor utilities or legacy
tools that show a live reading on screen but offer no way to record it.
<img width="575" height="781" alt="pylogger" src="https://github.com/user-attachments/assets/a95f9dec-27dd-40ef-b653-2d4a34e9b7b4" />

## Features

- Region picker: drag a rectangle over the displayed value on a dimmed
  fullscreen overlay
- Two capture modes:
  - **OCR** – screenshot of the region + Tesseract (works for any
    rendered text, including custom-drawn readouts)
  - **Clipboard** – triple-click into the value field + Ctrl+C (exact,
    no recognition errors, but requires the value to be selectable text)
- Optional mouse click at a chosen point before each reading (e.g. a
  Refresh / Read / Single button), with configurable settle delay;
  mouse position is restored afterwards
- Drift-free interval timer (readings run on a fixed monotonic grid)
- CSV log: `date;time;value;raw`, UTF-8 with BOM (opens directly in
  Excel), decimal comma normalized to a dot in the parsed value column
- Optional archiving of every captured region to `shots/` as evidence
  for the measurement protocol
- OCR preprocessing tunable from the GUI: 3x upscale, grayscale,
  invert, threshold, digits-only whitelist
- Single-shot test button with live preview of the captured region
- Multilingual UI: English, Czech, German, Polish (auto-detected from
  Windows UI language, switchable at runtime)
- Settings persisted to a JSON file next to the script
- DPI aware – coordinates are physical pixels, works with Windows
  display scaling
- Emergency stop: move the mouse to the top-left screen corner
  (pyautogui failsafe)

## Requirements

- Windows 10/11
- Python 3.10+
- Python packages:

      pip install mss pillow pytesseract pyautogui pywin32

- Tesseract OCR (only for OCR mode):

      winget install UB-Mannheim.TesseractOCR

  or the installer from
  https://github.com/UB-Mannheim/tesseract/wiki
  Default path `C:\Program Files\Tesseract-OCR\tesseract.exe` is
  pre-filled in the app; adjust it in the GUI if installed elsewhere.
  No extra language packs needed – digits are read with the default
  `eng` data.

## Usage

    python screen_logger.py

1. **Select region** – drag a tight rectangle around the displayed
   number (Esc cancels). Keep labels and surrounding text out of the
   region; a unit inside the region is fine (it stays in the `raw`
   column, the `value` column gets only the parsed number).
2. Optionally **Select point** and enable the pre-reading click if the
   software needs a button press before each readout. Set the delay to
   however long the new value takes to render.
3. Choose mode (OCR / clipboard), interval in seconds and the CSV path.
4. **Test read** – verify the parsed value and the preview before
   starting.
5. **Start**. The status line shows the record count and last value.

During a run: do not lock the workstation or let the display sleep, do
not move or cover the target window, and do not change display
resolution or scaling (coordinates are physical pixels – reselect the
region after any such change).

## Log format

    date;time;value;raw
    2026-07-27;14:03:05;23.47;23,47 V
    2026-07-27;14:03:10;23.51;23,51 V

The file is opened and closed on every write, so it can be read while a
measurement is running and nothing is lost on a crash. The header
language follows the UI language and is written only when the file is
created.

## OCR troubleshooting

| Symptom | Fix |
|---|---|
| Light digits on dark background not read | enable Invert |
| Unstable readings, occasional garbage | set threshold 100–180, tune with Test read |
| Surrounding text gets picked up | shrink the region tightly around the digits |
| 0/O, 1/l confusion | keep "Digits and signs only" enabled |
| Seven-segment / LCD fonts | threshold + invert; if still failing, try clipboard mode |

## Configuration

All settings live in `screen_logger_config.json` next to the script
(plain JSON, saved automatically on exit). Keep multiple copies for
different instruments and swap them in as needed.

## License

MIT
