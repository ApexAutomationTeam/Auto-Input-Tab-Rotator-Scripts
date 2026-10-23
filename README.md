# Auto Input & Tab Rotator

Small Python scripts that **simulate periodic keyboard input (Ctrl presses) and rotate Chrome tabs on a schedule**. You choose the timing, the number of key presses and the tab-switch interval, either as fixed values or as repeating series.

> Built by [Apex Automation Team](https://apexautomationteam.com/). See more builds, with videos of real automations running, on the **[Sample Works page](https://apexautomationteam.com/sample-works/)**.

---

## Overview

| | |
|---|---|
| **Type** | Desktop input automation (keyboard simulation + window/tab control) |
| **Language** | Python 3.8+ |
| **Platform** | Windows 10 / 11 (window focusing uses `pygetwindow`'s Windows API) |
| **Target app** | Google Chrome (tab switching via `Ctrl + Tab`) |
| **Dependencies** | `pyautogui`, `pygetwindow` |
| **Interface** | Command line, settings are edited at the top of each script |
| **Stop** | `Ctrl + C` in the terminal |

## What it does

1. **Presses Ctrl** a configurable number of times, at a configurable interval.
2. **Brings Chrome to the front** and sends `Ctrl + Tab` to move to the next tab.
3. Optionally runs a **one-time test mode** so you can see tab switching work within a minute.
4. Supports **series**: lists of times and click counts that cycle forever (step 1, step 2, ... back to step 1).

## Possible uses

- Rotating dashboards or monitoring tabs on a wall display or kiosk.
- Simulating light user input during UI / soak testing of your own apps.
- Demos and screen recordings that need hands-free tab changes.
- Keeping your own machine from idling while a long, supervised task runs.

## Responsible use

This tool only generates normal keyboard events. **You are responsible for how you use it.** Do not use it to misrepresent activity or working time to a platform, employer or client, or in any way that breaks the terms of a service you use. Many time-tracking and monitoring services prohibit simulated activity and can suspend accounts for it. The authors accept no liability for misuse.

---

## Scripts

All scripts live in [`scripts/`](scripts/). Pick the one that matches the behaviour you need.

| Script | Behaviour | Edit these settings |
|---|---|---|
| `01_basic_ctrl_and_tab.py` | Fixed Ctrl burst every N seconds, fixed tab switch every N seconds, plus a one-time test run | `CTRL_INTERVAL`, `CTRL_PRESS_COUNT`, `TEST_TAB_DELAY`, `TEST_TAB_CHANGES`, `NORMAL_TAB_INTERVAL` |
| `02_ctrl_series.py` | Ctrl presses only, as a series of `(minutes, press_count)` steps | `series` |
| `03_tab_series.py` | Fixed Ctrl bursts, tab switches on a series of minute intervals | `CTRL_INTERVAL`, `CTRL_PRESS_COUNT`, `tab_time_series` |
| `04_coupled_series.py` | Ctrl time and Ctrl count are paired step by step, tabs have their own series | `ctrl_time_series`, `ctrl_click_series`, `tab_time_series` |
| `05_decoupled_series.py` | Ctrl time, Ctrl count and tab time each cycle independently, lengths may differ | same three tuples |

### Time units

- Script `01` uses **seconds** (`8 min = 8 x 60 = 480`).
- Scripts `02` to `05` use **minutes** and convert to seconds internally. Decimals work (`0.5` = 30 seconds).

---

## Installation

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install pyautogui pygetwindow
```

## Usage

1. Open Chrome with the tabs you want to rotate and **leave the window in front** (not minimized).
2. Edit the settings at the top of your chosen script.
3. Run it from a terminal:

```bash
cd scripts
python 01_basic_ctrl_and_tab.py
```

4. Press `Ctrl + C` to stop.

> **Tip:** if you edit a script, close the old terminal window and start a fresh one before running again, otherwise the old process may still be running.

---

## Configuration examples

### Basic script (`01`)

```python
CTRL_INTERVAL = 20         # Ctrl burst every 20 seconds
CTRL_PRESS_COUNT = 3       # 3 presses per burst
TEST_TAB_DELAY = 60        # run the test after 1 minute
TEST_TAB_CHANGES = 3       # switch 3 tabs during the test
NORMAL_TAB_INTERVAL = 480  # then switch 1 tab every 8 minutes
```

For a 10 minute tab interval use `600`.

### Ctrl series (`02`)

```python
series = [
    (1, 2),    # every 1 min  -> Ctrl x2
    (3, 3),    # every 3 min  -> Ctrl x3
    (10, 5),   # every 10 min -> Ctrl x5
]
```

### Coupled series (`04`)

```python
ctrl_time_series  = (3, 2, 4)   # minutes
ctrl_click_series = (2, 3, 4)   # presses
tab_time_series   = (3, 2, 4)   # minutes, 1 tab per step
```

Result: 3 min -> Ctrl x2 + 1 tab, then 2 min -> Ctrl x3 + 1 tab, then 4 min -> Ctrl x4 + 1 tab, then repeat.

`ctrl_time_series` and `ctrl_click_series` **must be the same length**; the script exits with a clear message if they are not.

### Decoupled series (`05`)

```python
ctrl_time_series  = (3, 2)       # minutes
ctrl_click_series = (2, 3, 4)    # presses
tab_time_series   = (5, 10, 13)  # minutes
```

Each tuple loops on its own, so different lengths are fine.

---

## How it works

- `pyautogui.keyDown('ctrl')` / `keyUp('ctrl')` with short delays (0.1 s down, 0.2 s between presses) simulate a Ctrl tap.
- `pygetwindow.getWindowsWithTitle('Chrome')` finds a Chrome window, restores it if minimized and activates it.
- `pyautogui.hotkey('ctrl', 'tab')` moves to the next tab.
- Series scripts keep a step index and a "last run" timestamp per series and compare against `time.time()` in a one-second loop.

## Known limitations

- **Windows only** for window focusing; `pygetwindow.activate()` is not supported on macOS / Linux.
- The first window whose title contains "Chrome" is used. If several Chrome windows are open, keep the one you want in front and the others minimized.
- Scripts `01` and `03` check the tab timer only after each Ctrl burst, so tab switches can be late by up to `CTRL_INTERVAL` seconds.
- Focusing Chrome takes the foreground from whatever app you are using at that moment.
- PyAutoGUI's fail-safe is on by default: slamming the mouse into a screen corner stops the script with an exception.
- Only `Ctrl + Tab` (next tab) is sent; there is no previous-tab or tab-selection logic.

## Keeping the PC awake (optional)

If the machine sleeps or locks, no input can be sent. On Windows, check **Control Panel > Power Options > Change plan settings > Advanced settings** (Sleep, Display, Power buttons and lid, Battery) and **Settings > Accounts > Sign-in options > Require sign-in** so the PC does not sleep or lock during your session.

---

## More from Apex Automation Team

Automation, web scraping, QA and back-office systems. Watch real builds run on the **[Sample Works page](https://apexautomationteam.com/sample-works/)** or get in touch at [contact@apexautomationteam.com](mailto:contact@apexautomationteam.com).

## License

Add your preferred license here (for example MIT) before publishing.
