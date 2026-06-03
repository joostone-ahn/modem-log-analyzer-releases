# 📡 DM Viewer

A web-based real-time DIAG log viewer for NR/LTE RRC and NAS messages.

![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-red)

---

## 🌐 Online Demo

**[Try Online Demo](https://huggingface.co/spaces/Joostone/dm-viewer)**

> Note: The demo runs in File Mode only. Upload .pcap or DM log files to decode. USB capture is available in the local version.

---

## 💡 Why This Tool?

DM Viewer connects directly to USB DIAG ports on macOS and Windows (WSL), capturing and decoding NR/LTE RRC and NAS messages in real-time in your browser. Saved log files (.pcap, .hdf, .qmdl, .dlf) can also be loaded and decoded.

Log analysis has always been manual — read messages, check against spec, make a judgment. Existing tools offer text export or copy-paste, but there's always an extra step before automation is possible. DM Viewer eliminates that step with a single pipeline — capture, decode, analyze — that connects directly to automation.

This pipeline enables automated compliance checks, PHY-layer visualization from raw config, and pass/fail logic for test procedures. Currently it covers RRC and NAS, but the plan is to expand into radio measurements, throughput, and RF conditions — enabling a much wider range of automated field analysis.

---

## 🖥️ Screenshot

![DM Viewer](docs/images/screenshot_dm_viewer.png)

---

## ✨ Key Features

- **Real-time Capture**: USB DIAG → scat → tshark loopback decode → instant SSE push
- **Cross-platform**: macOS (native) and Windows (WSL2 + usbipd)
- **Direct USB DIAG**: Access via scat + pyusb, no commercial tools needed
- **File Load**: Open .pcap / .hdf / .qmdl / .dlf files
- **Protocol Support**: NR-RCC, LTE-RRC, NAS-5GS, NAS-EPS
- **AT Command**: Power Off/On (AT+CFUN) without leaving the tool
- **Dark/Light Theme**: Manual toggle, saved in browser

---

## 🚀 Quick Start

Download the latest release from [Releases](https://github.com/joostone-ahn/dm-viewer-releases/releases).

### macOS

**Prerequisites**: [Homebrew](https://brew.sh), Python 3.10+

1. Download and extract `dm-viewer-vX.X.X-macos.zip`
2. Double-click `run/run.command` (or run `bash run/run.command` in terminal)
3. Browser opens at `http://localhost:8340`

> On first run, dependencies (tshark, scat, etc.) are installed automatically.

### Windows (WSL)

**Prerequisites**: Windows 10/11, Administrator access, BIOS virtualization enabled

1. Download and extract `dm-viewer-vX.X.X-windows.zip`
2. Run `run/setup-wsl.bat` as Administrator (one-time setup)
3. Run `run/run-wsl.bat` as Administrator (every time)
4. Browser opens at `http://localhost:8340`

> First-time setup requires a reboot. After reboot, run `setup-wsl.bat` again to complete.

---

## 📖 Documentation

- [User Guide (한국어)](https://github.com/joostone-ahn/dm-viewer-releases/blob/main/docs/manual/user-guide-kr.md)
- [User Guide (English)](https://github.com/joostone-ahn/dm-viewer-releases/blob/main/docs/manual/user-guide-en.md)

---

## 📋 Change History

| Version | Date | Changes |
|---------|------|---------|
| v1.0.0 | 2026-06-03 | Initial release — real-time USB capture, file load, AT command, cross-platform (macOS / Windows WSL) |

---

## 👤 Author

**JUSEOK AHN (안주석)**  
**Email**: ajs3013@lguplus.co.kr  
**Organization**: LG U+  
**Role**: Technical Specialist, Telecommunications Engineer

---

## 📄 License

© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr>. All rights reserved.

This software is provided free of charge for personal and internal use.
You may not modify, distribute, sublicense, or sell copies of this software
without explicit written permission from the author.

Relies on [tshark](https://www.wireshark.org/) (GPL-2.0) and [scat](https://github.com/fgsect/scat) (GPL-2.0) as external subprocesses.
