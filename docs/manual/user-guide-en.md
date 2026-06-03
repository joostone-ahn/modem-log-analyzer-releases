# DM Viewer User Guide

**Version:** v1.0.0

---

## Table of Contents

1. [Running the Program](#1-running-the-program)
2. [USB Real-time Capture](#2-usb-real-time-capture)
3. [File Loading](#3-file-loading)
4. [Message Viewer](#4-message-viewer)
5. [AT Command (Power Off/On)](#5-at-command-power-offon)
6. [Troubleshooting](#6-troubleshooting)
7. [How to Exit](#7-how-to-exit)

---

## 1. Running the Program

### 1.1 macOS

1. Double-click `run.command`
2. A terminal window opens and configures the environment automatically
3. The browser opens automatically: http://localhost:8340

> To run from terminal: `bash run.command`

**Installed automatically on first run:**
- tshark, libusb (via Homebrew)
- Python virtual environment + dependencies
- scat (from upstream git)
- scat.lua (Wireshark plugin)

**Prerequisites:**
- [Homebrew](https://brew.sh)
- Python 3.10+

### 1.2 Windows

**Step 1: Initial Setup (one-time)**

Run `setup-wsl.bat` as Administrator (right-click → Run as administrator)

Automatically installs:
- WSL2 + Ubuntu
- usbipd-win
- Internal WSL environment (Python, tshark, scat, etc.)

> A reboot is required on first installation. After rebooting, run `setup-wsl.bat` again.

**Step 2: Launch (every time)**

Run `run-wsl.bat` as Administrator

This connects the USB device to WSL and starts the server automatically.  
The browser opens automatically: http://localhost:8340

**Prerequisites:**
- Windows 10/11
- Administrator access
- BIOS virtualization (VT-x/AMD-V) enabled

---

## 2. USB Real-time Capture

### 2.1 Device Preparation

The smartphone must expose a diagnostic (DIAG) port over USB. This varies by device, so providing a universal solution for all devices is difficult.

**Samsung:**
- Enter `*#0808#` in the phone dialer → Select any USB mode entry containing DM (e.g., DM+MODEM+ADB)
- Korean models: Enter `3197123580` in the dialer, password is `996412`, `776432`, or `0821`

**Apple iPhone (Qualcomm baseband):**
- Settings → Carrier Settings → Baseband Manager → Logging Settings → **Qualcomm**: Passive, External Hardware (QXDM)

> **Tested devices:** Samsung Galaxy Z Fold7, iPhone 17 Pro (both Qualcomm chipsets)  
> Samsung Exynos-based devices (.sdm format) have not been tested yet.

### 2.2 Windows: Verify USB Port Before Launch

Before running `run-wsl.bat`, verify that the USB device is recognized in Windows.

1. Enable DIAG mode on the device (see 2.1)
2. Connect the USB cable
3. Check **Device Manager**: `Win+X` → Device Manager → Ports (COM & LPT) or Universal Serial Bus devices — confirm the device appears
4. Once the port is visible, run `run-wsl.bat` as Administrator

> If the port is not recognized, `run-wsl.bat` will fail to bind the USB device via usbipd.  
> If the port doesn't appear, try a different USB cable, a different port, or re-enable DIAG mode on the device.  
> If USB is disconnected and reconnected during capture, exit `run-wsl.bat` (Ctrl+C), verify the port appears again, then re-run.

### 2.3 Connection

1. Connect the smartphone via USB
2. Click the **Connect** button in the browser
3. Once the device is detected, capture starts automatically
4. Status indicator turns green ● and messages appear in real time

### 2.4 Controls During Capture

| Button | Action |
|--------|--------|
| Disconnect | Stop capture |
| Power Off/On | Toggle modem power (AT+CFUN=0/1) |
| Save | Download captured data as pcap file |
| Auto-scroll | Toggle auto-scroll on new messages |
| Clear | Clear on-screen messages (capture continues) |

### 2.5 Disconnection

Click **Disconnect** to stop the capture.

---

## 3. File Loading

### 3.1 Upload

1. Click **Open File**, or drag and drop a file onto the screen
2. After parsing, the message list is displayed

### 3.2 Supported Formats

| Extension | Description |
|-----------|-------------|
| `.pcap` | Wireshark capture file (parsed directly by tshark) |
| `.hdf` | Qualcomm HDF log (converted to pcap via scat, then parsed) |
| `.qmdl` | Qualcomm QMDL log |
| `.dlf` | Qualcomm DLF log |
| `.sdm` | Samsung Shannon DM log (untested) |

---

## 4. Message Viewer

### 4.1 Layout

| Area | Description |
|------|-------------|
| Left panel | Message list table |
| Right panel | Detailed ASN.1 decoding of selected message |
| Panel divider | Drag to adjust left/right panel ratio |

### 4.2 Message Table

| Column | Description |
|--------|-------------|
| # | Frame number |
| Time | Timestamp (HH:MM:SS.mmm) |
| Protocol | NR-RRC, LTE-RRC, NAS-5GS, NAS-EPS |
| Channel | DL-DCCH, UL-DCCH, BCCH-DL-SCH, etc. |
| Message | Message name |

Protocol color coding:
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

## 5. AT Command (Power Off/On)

During capture, click **Power Off** to send `AT+CFUN=0` (modem RF off).  
The button changes to **Power On** — click it to send `AT+CFUN=1` (modem RF on).

This allows you to capture the full Attach/Registration sequence from the beginning.

---

## 6. Troubleshooting

### 6.1 USB Device Not Detected

**Common:**
- Check the USB cable (charge-only cables cannot transfer data)
- Verify DIAG port is enabled (see section 2.1 Device Preparation)
- Close other programs (QXDM, Wireshark, etc.) that may be holding the USB port

**Windows:**
- Ensure `run-wsl.bat` is run as Administrator
- Verify usbipd is installed: run `usbipd list` in CMD

### 6.2 Windows: BIOS Virtualization Not Enabled

**Symptom:** `HCS_E_HYPERV_NOT_INSTALLED` error during `setup-wsl.bat`

**Solution:**
1. Restart PC → Enter BIOS (F2 or Del)
2. Enable Intel VT-x or AMD SVM
3. Save and restart

Alternatively: Settings → System → Recovery → Advanced startup → UEFI Firmware Settings

**Verification:** Task Manager → Performance → CPU → Confirm "Virtualization: Enabled"

### 6.3 Windows: Reboot Required During Setup

**Symptom:** "A reboot is required" message after running `setup-wsl.bat`

VirtualMachinePlatform is a Windows kernel feature that requires a reboot on first activation.  
After rebooting, run `setup-wsl.bat` again to continue installation.

### 6.4 Windows: Browser Cannot Connect to Page

**Symptom:** Server shows "starting on port 8340" but browser displays `ERR_CONNECTION_REFUSED`

**Cause:** WSL2 uses a virtual network, and Windows accesses localhost through WSL2 → Windows forwarding. If `run-wsl.bat` is executed again without stopping the previous server, this forwarding can temporarily break.

**Solution:**

1. Wait 30 seconds to 1 minute, then refresh the browser
2. If still not working, run `wsl --shutdown` in an Administrator CMD, then re-run `run-wsl.bat`
3. If that doesn't help, reboot Windows

**Prevention:** Always use **Ctrl+C** to stop the server. Avoid closing the CMD window with the X button or running `run-wsl.bat` again while the server is still running.

### 6.5 Windows: USB Port Not Released (Abnormal Termination)

**Symptom:** After closing DM Viewer, COM ports are not visible in QXDM or other tools

**Cause:** Closing the CMD window with the X button skips the USB unbind process, leaving the port attached to WSL

**Solution:**

1. Run `run-wsl.bat` again (it cleans up stale connections on startup and re-attaches)
2. Confirm the port is recognized normally in WSL
3. Exit with **Ctrl+C** → USB port is returned to Windows
4. Verify COM port appears in QXDM or other tools

**If the port still doesn't return (manual cleanup):**
```cmd
usbipd detach --all
usbipd unbind --all
```
Run these commands in an Administrator CMD to release all USB bindings and return ports to Windows.

> Always exit with **Ctrl+C** to prevent this issue.

### 6.5 NR RRC Messages Not Appearing (scat.lua Issue)

**Symptom:** LTE-RRC messages appear but NR-RRC messages do not

**Cause:** If scat.lua plugin is not loaded in tshark, NR RRC decoding is not possible

**macOS fix:**
```bash
rm -rf .venv
bash run.command
```
Recreating the venv reinstalls scat.lua automatically.

**Windows fix:**
Re-run `setup-wsl.bat` to reinstall scat.lua.

### 6.6 DIAG Session Corruption (Messages Stop Arriving)

**Symptom:** Messages suddenly stop arriving during capture (scat process is still running)

**Cause:** Abnormal scat termination corrupted the device's DIAG session

**Solution:**
1. Click **Disconnect** in DM Viewer
2. Reboot the smartphone (power off → on)
3. Physically unplug and reconnect the USB cable
4. Click **Connect** to reconnect

### 6.7 scat Fails to Decode (Latest Chipset Not Supported)

**Symptom:** Connection succeeds but no messages appear; scat stderr shows `Unknown NR RRC OTA Message packet version`

**Cause:** Installed scat version does not support the latest Qualcomm chipset's pkt_ver

**Solution:**
```bash
# macOS
rm -rf .venv
bash run.command

# Windows
# Re-run setup-wsl.bat (reinstalls scat from upstream git)
```

### 6.8 Port Conflict (UDP 4729 or TCP 8340)

**Symptom:** "Cannot bind UDP 4729" or server fails to start

**Cause:** Wireshark, a previous DM Viewer process, or other software is holding the port

**macOS fix:**
```bash
lsof -ti:8340 | xargs kill -9
lsof -ti:4729 | xargs kill -9
bash run.command
```

**Windows:** `run-wsl.bat` automatically cleans up existing processes on startup.

---

## 7. How to Exit

### macOS
Press **Ctrl+C** in the terminal

### Windows
Press **Ctrl+C** in the CMD window (USB port is automatically returned to Windows)

> ❌ Do NOT close the CMD window with the X button. USB port release may fail.

---

© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr> All rights reserved.
