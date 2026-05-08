# My-Lathe — Custom LinuxCNC Lathe GUI

A custom PyQt5 GUI for a 2-axis CNC lathe running LinuxCNC. This project can also run in **offline preview mode** on Windows (or any system without LinuxCNC installed) for development and demonstration.

---

## Running the Offline GUI (Windows)

### Prerequisites

1. **Python 3.10+** — Download from https://www.python.org/downloads/
   - During install, check **"Add Python to PATH"**

2. **PyQt5** — Install via pip:
   ```
   pip install PyQt5
   ```

That's it. No other dependencies are required for offline mode.

### Launching

Open a terminal (Command Prompt or PowerShell) and run:

```
cd my-lathe\gui
python lathe_gui.py
```

The GUI will detect that LinuxCNC modules are unavailable and automatically start in **offline preview mode** with simulated demo data. You'll see this message in the console:

```
LinuxCNC modules not found — running in offline preview mode
```

### What Works in Offline Mode

- Full GUI layout and styling (DRO, tabs, sidebar)
- Position graph with G-code simulation playback
- Conversational programming (profile editor, G-code generation)
- Tool table editing
- G-code editor with syntax highlighting
- Theme and font rendering

### What Doesn't Work Offline

- Live machine control (jog, spindle, coolant)
- Real-time position feedback
- HAL pin monitoring
- Program execution on the machine

---

## Project Structure

```
my-lathe/
├── gui/
│   ├── lathe_gui.py        — Main GUI application (entry point)
│   ├── conv_profile.py     — Mazatrol-style profile conversational programming
│   ├── conversational.py   — Step-based conversational (OD Turn, Bore, Thread, Cutoff)
│   ├── theme.py            — Colors, stylesheet, fonts
│   ├── widgets.py          — DRO, status indicators
│   ├── position_graph.py   — Position graph and simulation
│   ├── tabs.py             — All tab widgets
│   └── ...                 — Supporting modules
├── my-lathe.ini            — LinuxCNC machine configuration
├── my-lathe.hal            — HAL wiring
├── tool.tbl                — Tool table
└── README.md               — This file
```

---

## Deploying to a LinuxCNC Machine

1. Copy the entire `my-lathe/` folder to your LinuxCNC config directory
   (typically `~/linuxcnc/configs/my-lathe/`)
2. In `my-lathe.ini`, set:
   ```ini
   [DISPLAY]
   DISPLAY = gui/lathe_gui.py
   ```
3. Launch LinuxCNC and select the `my-lathe` configuration

---

## Notes

- All X-axis values shown in the GUI are **diameter** (standard lathe convention)
- Units are **inches**
- The GUI uses Inter (UI) and JetBrains Mono (DRO/code) fonts with system fallbacks






[DEPLOY_README.md](https://github.com/user-attachments/files/27538105/DEPLOY_README.md)
# My-Lathe — LinuxCNC Deployment Guide

Custom PyQt5 GUI for a 2-axis CNC lathe running LinuxCNC with Mesa 7i96s + 7i85s.


## Hardware Summary

- Mesa 7i96s (Ethernet) + 7i85s daughter card
- Closed-loop steppers on X and Z with Sino linear encoder feedback
- Manual spindle with 1000 PPR rotary encoder (threading/CSS only)
- 2x MPG handwheels, jog buttons, limit/home switches
- Quick change tool post (manual tool change)
- Units: inches


===============================================================================
## Installation on the LinuxCNC Machine
===============================================================================

### 1. Copy the project folder

Copy the entire my-lathe/ directory to your LinuxCNC config location.
Adjust the USB mount point to match your system (check with: lsblk)

  ┌─────────────────────────────────────────────────────────────────────┐
  │  cp -r /media/usb/my-lathe ~/linuxcnc/my-lathe                     │
  └─────────────────────────────────────────────────────────────────────┘

Final location should be:  /home/linuxcnc/linuxcnc/my-lathe/

NOTE: If your Linux username is not "linuxcnc", update these paths in
my-lathe.ini:
  - [DISPLAY] DISPLAY = /home/YOUR_USER/linuxcnc/my-lathe/gui/lathe_gui.py
  - [DISPLAY] PROGRAM_PREFIX = /home/YOUR_USER/linuxcnc/nc_files
  - [RS274NGC] SUBROUTINE_PATH = /home/YOUR_USER/linuxcnc/nc_files/subroutines


### 2. Set file permissions

  ┌─────────────────────────────────────────────────────────────────────┐
  │  chmod +x ~/linuxcnc/my-lathe/gui/lathe_gui.py                     │
  │  chmod +x ~/linuxcnc/my-lathe/shutdown.sh                          │
  └─────────────────────────────────────────────────────────────────────┘


### 3. Create the G-code directories

  ┌─────────────────────────────────────────────────────────────────────┐
  │  mkdir -p ~/linuxcnc/nc_files                                       │
  │  mkdir -p ~/linuxcnc/nc_files/subroutines                           │
  └─────────────────────────────────────────────────────────────────────┘


### 4. Install Python dependencies

  ┌─────────────────────────────────────────────────────────────────────┐
  │  sudo apt update                                                    │
  │  sudo apt install python3-pyqt5 python3-pyqt5.qtopengl             │
  └─────────────────────────────────────────────────────────────────────┘

The linuxcnc and hal Python modules are already present on any LinuxCNC
installation — no extra install needed.


### 5. Install fonts (optional but recommended)

Inter UI font:

  ┌─────────────────────────────────────────────────────────────────────┐
  │  sudo apt install fonts-inter                                       │
  └─────────────────────────────────────────────────────────────────────┘

JetBrains Mono (for DRO/code display) — download .ttf files from
https://www.jetbrains.com/mono/ then:

  ┌─────────────────────────────────────────────────────────────────────┐
  │  mkdir -p ~/.local/share/fonts                                      │
  │  cp JetBrainsMono*.ttf ~/.local/share/fonts/                        │
  │  fc-cache -fv                                                       │
  └─────────────────────────────────────────────────────────────────────┘

Alternative: place .ttf files in ~/linuxcnc/my-lathe/gui/fonts/ and the
GUI will load them automatically at startup.


===============================================================================
## Launching LinuxCNC
===============================================================================

From the LinuxCNC config picker:
  LinuxCNC scans ~/linuxcnc/ for .ini files. Select "My-Lathe" from the list.

From the command line:

  ┌─────────────────────────────────────────────────────────────────────┐
  │  linuxcnc ~/linuxcnc/my-lathe/my-lathe.ini                         │
  └─────────────────────────────────────────────────────────────────────┘


===============================================================================
## First-Run Checklist
===============================================================================

1. TEST WITH AXIS FIRST — Edit my-lathe.ini, temporarily change DISPLAY to
   axis, and verify hardware works (encoders read, steppers move, limits
   trigger). This isolates hardware issues from GUI issues.

   To use AXIS temporarily, change this line in my-lathe.ini:
   DISPLAY = axis

2. Check Mesa communication:

  ┌─────────────────────────────────────────────────────────────────────┐
  │  ping 10.10.10.10                                                   │
  │  dmesg | grep hm2                                                   │
  └─────────────────────────────────────────────────────────────────────┘

3. Verify HAL pins (run these while LinuxCNC is running):

  ┌─────────────────────────────────────────────────────────────────────┐
  │  halcmd show pin | grep encoder                                     │
  │  halcmd show pin | grep stepgen                                     │
  │  halcmd show pin | grep gpio                                        │
  └─────────────────────────────────────────────────────────────────────┘

4. Home both axes and confirm encoder feedback matches commanded position.

5. Switch to custom GUI — change DISPLAY back to the full path:
   DISPLAY = /home/linuxcnc/linuxcnc/my-lathe/gui/lathe_gui.py
   Then relaunch LinuxCNC.


===============================================================================
## Fallback to Stock AXIS GUI
===============================================================================

If the custom GUI has issues, revert with one line change in my-lathe.ini.
Change:

  DISPLAY = /home/linuxcnc/linuxcnc/my-lathe/gui/lathe_gui.py

To:

  DISPLAY = axis

All HAL, motion control, and hardware continue to work regardless of which
GUI is running.


===============================================================================
## Clean Shutdown
===============================================================================

Use the shutdown script to ensure work offsets and tool table are saved:

  ┌─────────────────────────────────────────────────────────────────────┐
  │  ~/linuxcnc/my-lathe/shutdown.sh                                    │
  └─────────────────────────────────────────────────────────────────────┘

To also power off the PC after stopping LinuxCNC:

  ┌─────────────────────────────────────────────────────────────────────┐
  │  ~/linuxcnc/my-lathe/shutdown.sh poweroff                           │
  └─────────────────────────────────────────────────────────────────────┘


===============================================================================
## How It Works
===============================================================================

The INI file tells LinuxCNC to:

1. Load my-lathe.hal — configures Mesa board, stepgens, linear encoders,
   PID loops, spindle encoder, limit switches, jog buttons, MPG handwheels,
   cycle start/stop, and analog pots
2. Load custom.hal — your additions (coolant, spindle brake, etc.)
3. Launch gui/lathe_gui.py as the display
4. Load postgui.hal — HAL connections that need the GUI running first

The GUI connects to LinuxCNC via linuxcnc.stat() and linuxcnc.command()
— it reads machine state and sends commands but does not create HAL pins.


===============================================================================
## File Reference
===============================================================================

  my-lathe.ini          Main LinuxCNC machine configuration
  my-lathe.hal          HAL wiring (Mesa, stepgens, encoders, PID, switches, MPGs)
  custom.hal            Your custom HAL additions (loaded after my-lathe.hal)
  postgui.hal           Post-GUI HAL connections (currently empty)
  my-lathe.var          Work offsets / parameters (LinuxCNC reads/writes this)
  tool.tbl              Tool table (LinuxCNC reads/writes this)
  shutdown.sh           Clean shutdown script
  gui/lathe_gui.py      Custom PyQt5 GUI (the DISPLAY process)
  gui/theme.py          Colors, stylesheet, font helpers
  gui/position_graph.py Position graph with G-code simulation
  gui/conv_profile.py   Mazatrol-style profile conversational programming
  gui/conversational.py Step-based conversational (not currently wired in)
  gui/gcode_editor.py   G-code editor with line numbers
  gui/widgets.py        DRO, status indicators, spindle/override panels
  gui/widgets_fence.py  Go-To fence soft limit widget
  gui/hal_providers.py  HAL pin data providers (live + offline)
  gui/hal_monitor_utils.py  HAL monitor utilities
  gui/line_numbers.py   G-code line number resolution
  gui/thread_data.py    Thread pitch/TPI data tables
  gui/tabs/             All tab modules (Manual, Program, MDI, Tool, etc.)
  gui/fonts/            Bundled font files (loaded automatically)
  gui/tests/            Unit tests (not needed on the machine)
  wiring-map.md         Physical wiring reference


===============================================================================
## Files You Can Safely Omit on the Machine
===============================================================================

These are development artifacts:

  - .hypothesis/ directories
  - .pytest_cache/
  - __pycache__/
  - gui/tests/
  - gui/lathe_gui_original.py.bak
  - gui/my-lathe-demo.zip
  - wiring-map.xlsx (keep .md version for reference)


===============================================================================
## Network Configuration
===============================================================================

The Mesa 7i96s communicates via Ethernet at 10.10.10.10. Your LinuxCNC PC's
network interface connected to the Mesa board must be configured with a
static IP on the same subnet:

  IP Address: 10.10.10.1
  Netmask:    255.255.255.0
  Gateway:    (none)

This should be a dedicated NIC — do not route internet traffic through it.
