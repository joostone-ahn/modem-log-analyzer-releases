# Modem Log Analyzer User Guide

**Version:** v1.6.2

---

## Table of Contents

1. [Running the Program](#1-running-the-program)
2. [USB Real-time Capture](#2-usb-real-time-capture)
3. [File Loading](#3-file-loading)
4. [Message Viewer](#4-message-viewer)
5. [Troubleshooting](#5-troubleshooting)
6. [How to Exit](#6-how-to-exit)

---

## 1. Running the Program

### Windows (EXE)

1. Download the latest `Modem-Log-Analyzer-vX.X.X.exe` from [Releases](https://github.com/joostone-ahn/modem-log-analyzer-releases/releases)
2. **Run as Administrator** (right-click → Run as administrator recommended)
3. A splash screen appears while the environment is configured
4. The app window opens automatically

#### First-time Setup (WSL not installed)

On a fresh PC, run `setup-wsl.bat` before using the EXE:

1. Download `setup-wsl.bat` from the Release page
2. Right-click → **Run as administrator**
3. WSL2 + Ubuntu installation completes → **reboot** when prompted
4. After reboot, run `setup-wsl.bat` **one more time** → remaining packages install
5. "Setup complete!" message confirms readiness → now run the EXE

> ⚠️ WSL is not active until after the reboot. You must reboot and re-run the bat.

#### Subsequent Runs

- WSL reset + server start (approx. 10 seconds)
- If a previous instance exists, it is automatically terminated and restarted

⚠️ **Note:** On first run, a `modem-log-analyzer-linux` file is created next to the exe. Do not delete this file.

### macOS (DMG)

> ⚠️ Prerequisite: [Homebrew](https://brew.sh) must be installed. Required dependencies (Wireshark, libusb, etc.) are auto-installed on first launch.

1. Download the latest `Modem-Log-Analyzer-vX.X.X-macOS.dmg` from [Releases](https://github.com/joostone-ahn/modem-log-analyzer-releases/releases)
2. Double-click the DMG file to mount it
3. Drag `Modem Log Analyzer.app` to the Applications folder (or any location)
4. On first launch, macOS Gatekeeper will show a warning:
   - **Right-click** the .app → **Open** → click **Open**
   - Subsequent launches work with a normal double-click
5. A splash screen appears and installs dependencies (first time only, 2-3 min)
6. The app window opens automatically when ready

> 💡 No Python, venv, or other prerequisites needed — it's a standalone app.

> 💡 Pin the .app to the Dock for quick access.

---

## 2. USB Real-time Capture

### 2.1 Device Preparation

The smartphone must expose a diagnostic (DIAG) port over USB.

**Samsung:**
- Enter `*#0808#` in the phone dialer → Select any USB mode containing DM (e.g., DM+MODEM+ADB)
- Korean models: Enter `3197123580` in the dialer, password `996412`, `776432`, or `0821`

**Apple iPhone (Qualcomm baseband):**
- Settings → Carrier Settings → Baseband Manager → Logging Settings → **Qualcomm**: Passive, External Hardware (QXDM)

> **Tested devices:** Samsung Galaxy Z Fold7, iPhone 17 Pro (Qualcomm chipsets)

### 2.2 Connection (Connect)

1. Connect the device to PC via USB
2. **Windows:** Verify the port appears in Device Manager
3. Click the **Connect** button in the app
4. Device detection → USB transfer to WSL → scat start (approx. 8 seconds)
5. Status indicator turns green ● and messages appear in real time

> During Connect, the USB port moves from Windows to WSL. It disappears from Device Manager — this is normal.

### 2.3 Controls During Capture

| Button | Action |
|--------|--------|
| Disconnect | Stop capture + return USB to Windows |
| Save | Download captured data as pcap file |
| Auto-scroll | Toggle auto-scroll on new messages |
| Clear | Clear on-screen messages (capture continues) |

> **Power Off/On button:** Only visible on macOS. Sends AT+CFUN=0/1 to toggle modem power, allowing you to capture the full Registration sequence from the beginning.

### 2.4 Disconnection (Disconnect)

Clicking **Disconnect**:
1. Stops the scat capture
2. Detaches and unbinds the USB device from WSL
3. The port reappears in Windows Device Manager
4. Other tools (QXDM, etc.) can now use the port

### 2.5 Device Switching

Switch devices without restarting the app:

1. Click **Disconnect**
2. Verify the port returns in Device Manager
3. Unplug the current device and connect a new one
4. Verify the new device port appears in Device Manager
5. Click **Connect** → Capture starts on the new device

### 2.6 USB Physical Disconnect During Capture

If USB is physically unplugged during capture:
- Watchdog detects the disconnect within 2 seconds
- "USB disconnected" error is displayed
- Capture stops automatically
- **Connect** button reappears
- Reconnect the device and click **Connect** to resume

### 2.7 Mode Switching (USB ↔ File)

Use the **USB Mode / File Mode** toggle in the toolbar.

| Direction | Condition | Behavior |
|-----------|-----------|----------|
| USB (capturing) → File | Capture running | Blocked — disconnect first |
| USB (data exists) → File | Data present | Save dialog shown (Save .pcap / Don't Save / Cancel) |
| USB (empty) → File | No data | Immediate switch |
| File (results shown) → USB | Parsed data displayed | Confirm dialog shown (Continue / Cancel) |
| File (empty) → USB | No data | Immediate switch |

> 💡 Selecting **Save** on mode switch downloads the captured log as .pcap.

---

## 3. File Loading

### 3.1 Upload

1. Select the **File Mode** tab at the top
2. Click **Open File**, or drag and drop a file onto the screen
3. After parsing, the message list is displayed

### 3.2 Supported Formats

| Extension | Description |
|-----------|-------------|
| `.pcap` | Wireshark capture file (parsed directly by tshark) |
| `.hdf` | Qualcomm HDF log (converted to pcap via scat) |
| `.qmdl` | Qualcomm QMDL log |
| `.dlf` | Qualcomm DLF log |
| `.sdm` | Samsung Shannon DM log (untested) |

### 3.3 Save (PCAP Download)

After file parsing, a **Save** button appears in the toolbar.

| Uploaded File | Save Action |
|---------------|-------------|
| `.pcap` | Downloads the original pcap as-is |
| `.hdf` / `.qmdl` / `.dlf` / `.sdm` | Downloads the pcap converted by scat |

**Use cases:**
- Convert DM logs to pcap for viewing in Wireshark
- Decoding in Wireshark requires the [scat.lua](https://raw.githubusercontent.com/fgsect/scat/master/wireshark/scat.lua) plugin
- Installation: download and copy to Wireshark plugin directory  
  - macOS/Linux: `~/.local/lib/wireshark/plugins/scat.lua`  
  - Windows: `%APPDATA%\Wireshark\plugins\scat.lua`

---

## 4. Message Viewer

### 4.1 Layout

| Area | Description |
|------|-------------|
| Left panel | Message list table |
| Right panel | Detailed ASN.1 decoding of selected message |
| Panel divider | Drag to adjust left/right ratio |

### 4.2 Message Table

| Column | Description |
|--------|-------------|
| # | Frame number |
| Time | Timestamp (HH:MM:SS.mmm) |
| Protocol | NR-RRC, LTE-RRC, NAS-5GS, NAS-EPS |
| Channel | DL-DCCH, UL-DCCH, BCCH-DL-SCH, etc. |
| Message | Message name |

Protocol colors:
- **NR-RRC**: Blue
- **LTE-RRC**: Green
- **NAS-5GS**: Purple
- **NAS-EPS**: Orange

### 4.3 Filtering

Type text in the top input field to filter by protocol, channel, or message name.

### 4.4 Detail Panel

Click a message to display the full decoding tree on the right.
- **Ctrl+F** (Cmd+F): Search within detail content
- **▲▼**: Navigate to previous/next match
- **Copy**: Copy detail content to clipboard

### 4.5 Keyboard

| Key | Action |
|-----|--------|
| `↑` / `↓` | Select previous/next message |

---

## 5. Troubleshooting

### 5.1 "No DIAG devices found" (Connect Fails)

- Check USB cable (charge-only cables cannot transfer data)
- Verify DIAG port is enabled (see section 2.1)
- On Windows, verify the port appears in Device Manager before clicking Connect
- Close other programs (QXDM, Wireshark) that may hold the USB port

### 5.2 Connected But No Messages

**Unsupported chipset:**
- Update to the latest EXE version (bundles latest scat upstream)

**DIAG session corruption (common on Samsung devices):**
- **Reboot the device** (power off → on)
- After reboot, reconnect USB → click **Connect**

**Only NR-RRC missing:**
- scat.lua plugin issue. Reinstall the latest EXE.

### 5.3 "Waiting for server" Timeout on Launch

Previous process terminated abnormally:

1. End all `Modem-Log-Analyzer` processes in Task Manager
2. Re-run the EXE

If still failing:
1. Run `wsl --shutdown` in an Administrator CMD
2. Re-run the EXE

### 5.4 Port Not Visible in Device Manager After Disconnect

Normally, Disconnect returns the port. If not:

**Method 1: Use `reset-usb.bat` (recommended)**

Run `reset-usb.bat` from the Release package **as Administrator**.
- Returns all USB devices from WSL back to Windows
- Also shuts down WSL for a clean state

**Method 2: Manual command**

```cmd
:: Run in Administrator CMD
usbipd unbind --all
```

### 5.5 USB Recovery After Abnormal Exit

If the EXE was force-closed or the PC shut down unexpectedly, USB devices may remain attached to WSL and unavailable to other Windows programs (QXDM, etc.).

**Run `reset-usb.bat` as Administrator** to recover immediately.

> 💡 `reset-usb.bat` is available on the Release page alongside the EXE.

### 5.6 Collecting Debug Logs

If you encounter issues, check the `modem-log-analyzer.log` file next to the EXE.
- Created fresh on each launch (previous log is overwritten)
- Records all internal operations: server start, USB connection, capture, errors
- Please include this file when reporting issues

### 5.7 Connect Fails After Switching Devices

1. Check that the new device is visible in Device Manager
2. If not, unplug and replug the USB cable
3. If visible, try **Connect** again

---

## 6. How to Exit

Close the app window to exit the program.

- **macOS:** `⌘Q` or click the close button (red ●)
- **Windows:** Click the **X** button at the top-right or press `Alt+F4`

When the window is closed:
1. If capturing, capture stops automatically
2. USB device is returned to Windows (detach + unbind)
3. Server is terminated

> **Note:** On Windows, you can also terminate the process via Task Manager.

> **Restart:** To use again after exit, re-run the EXE (Windows) or .app (macOS).

---

**© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr>. Licensed under GPLv2.**
