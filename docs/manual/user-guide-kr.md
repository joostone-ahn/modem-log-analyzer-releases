# DM Viewer 사용자 가이드

**버전:** v1.1.0

---

## 목차

1. [프로그램 실행](#1-프로그램-실행)
2. [USB 실시간 캡처](#2-usb-실시간-캡처)
3. [파일 로드](#3-파일-로드)
4. [메시지 뷰어](#4-메시지-뷰어)
5. [AT Command (Power Off/On)](#5-at-command-power-offon)
6. [트러블슈팅](#6-트러블슈팅)
7. [종료 방법](#7-종료-방법)

---

## 1. 프로그램 실행

1. [Releases](https://github.com/joostone-ahn/dm-viewer-releases/releases)에서 `DM-Viewer.exe`를 다운로드합니다
2. `DM-Viewer.exe`를 더블클릭합니다
3. 브라우저가 자동으로 열립니다: http://localhost:8340

> **최초 실행 시:** WSL2, usbipd-win, scat, tshark가 자동 설치됩니다. 재부팅이 필요할 수 있습니다.  
> **이후 실행:** 즉시 시작됩니다. USB 디바이스가 감지되면 자동으로 연결됩니다.

⚠️ **주의:** 최초 실행 시 exe 옆에 `dm-viewer-linux` 파일이 자동 생성됩니다. 이 파일을 삭제하지 마세요.

---

## 2. USB 실시간 캡처

### 2.1 디바이스 준비

스마트폰은 USB를 통해 진단(DIAG) 포트를 노출해야 합니다. 이는 기기마다 다르므로 모든 기기에 적용 가능한 일반적인 방법을 제시하기 어렵습니다.

**Samsung:**
- 전화 다이얼러에서 `*#0808#` 입력 → DM이 포함된 USB 모드 선택 (예: DM+MODEM+ADB)
- 한국 내수 모델: 다이얼러에서 `3197123580` 입력, 비밀번호 `996412` 또는 `776432` 또는 `0821`

**Apple iPhone (Qualcomm 베이스밴드):**
- 설정 → Carrier Settings → Baseband Manager → Logging Settings → **Qualcomm**: Passive, External Hardware (QXDM) 선택

> **현재 검증된 기기:** Samsung Galaxy Z Fold7, iPhone 17 Pro (모두 Qualcomm 칩셋)  
> Samsung Exynos 기반 단말(.sdm 포맷)은 미검증 상태입니다.

### 2.2 Windows: 실행 전 USB 포트 확인

`run-wsl.bat` 실행 전에 Windows에서 USB 디바이스가 인식되는지 반드시 확인합니다.

1. 단말에서 DIAG 모드 설정 (2.1 참고)
2. USB 케이블을 연결합니다
3. **장치 관리자** 확인: `Win+X` → 장치 관리자 → 포트(COM & LPT) 또는 범용 직렬 버스 장치에서 단말이 잡히는지 확인
4. 포트가 보이면 `run-wsl.bat`을 관리자 권한으로 실행합니다

> 포트가 안 잡힌 상태에서 `run-wsl.bat`을 실행하면 usbipd bind가 실패합니다.  
> 포트가 안 보이면 USB 케이블 교체, 다른 포트 시도, 또는 단말의 DIAG 모드 재설정을 먼저 하세요.  
> 실행 중 USB를 뽑았다가 다시 연결한 경우, `run-wsl.bat`을 종료(Ctrl+C)하고 포트가 잡힌 것을 확인한 뒤 다시 실행해야 합니다.

### 2.3 연결

1. 스마트폰을 USB로 연결합니다
2. 브라우저에서 **Connect** 버튼을 클릭합니다
3. 디바이스가 감지되면 캡처가 자동으로 시작됩니다
4. 상태 표시가 녹색 ●으로 변경되고 메시지가 실시간으로 표시됩니다

### 2.4 캡처 중 제어

| 버튼 | 동작 |
|------|------|
| Disconnect | 캡처 중지 |
| Power Off/On | 모뎀 전원 토글 (AT+CFUN=0/1) |
| Save | 캡처 데이터를 pcap 파일로 다운로드 |
| Auto-scroll | 새 메시지 도착 시 자동 스크롤 ON/OFF |
| Clear | 화면 메시지 삭제 (캡처는 계속됨) |

### 2.5 연결 해제

**Disconnect** 버튼을 클릭하면 캡처가 중지됩니다.

---

## 3. 파일 로드

### 3.1 업로드

1. **Open File** 버튼 클릭, 또는 파일을 화면에 드래그 앤 드롭합니다
2. 파싱 완료 후 메시지 목록이 표시됩니다

### 3.2 지원 포맷

| 확장자 | 설명 |
|--------|------|
| `.pcap` | Wireshark 캡처 파일 (tshark로 직접 파싱) |
| `.hdf` | Qualcomm HDF 로그 (scat으로 pcap 변환 후 파싱) |
| `.qmdl` | Qualcomm QMDL 로그 |
| `.dlf` | Qualcomm DLF 로그 |
| `.sdm` | Samsung Shannon DM 로그 (미검증) |

### 3.3 Save (PCAP 다운로드)

파일 파싱 완료 후 툴바에 **Save** 버튼이 나타납니다.

| 업로드 파일 | Save 동작 |
|------------|-----------|
| `.pcap` | 업로드한 원본 pcap 파일을 그대로 다운로드 |
| `.hdf` / `.qmdl` / `.dlf` / `.sdm` | scat으로 변환된 pcap 파일을 다운로드 |

**활용:**
- DM 로그 파일을 pcap으로 변환하여 Wireshark에서 직접 열어볼 수 있습니다
- 변환된 pcap을 NR RRC Visualizer에 업로드하여 PHY 파라미터 분석이 가능합니다
- 팀원에게 공유 시 Wireshark 호환 포맷으로 전달할 수 있습니다

---

## 4. 메시지 뷰어

### 4.1 화면 구성

| 영역 | 설명 |
|------|------|
| 좌측 패널 | 메시지 목록 테이블 |
| 우측 패널 | 선택한 메시지의 상세 ASN.1 디코딩 |
| 패널 구분선 | 드래그하여 좌/우 패널 비율 조절 |

### 4.2 메시지 테이블

| 컬럼 | 설명 |
|------|------|
| # | 프레임 번호 |
| Time | 타임스탬프 (HH:MM:SS.mmm) |
| Protocol | NR-RRC, LTE-RRC, NAS-5GS, NAS-EPS |
| Channel | DL-DCCH, UL-DCCH, BCCH-DL-SCH 등 |
| Message | 메시지 이름 |

프로토콜별 색상 구분:
- **NR-RRC**: 파란색
- **LTE-RRC**: 초록색
- **NAS-5GS**: 보라색
- **NAS-EPS**: 주황색

### 4.3 필터링

상단 입력란에 텍스트를 입력하면 프로토콜, 채널, 메시지 이름 기준으로 필터링됩니다.

### 4.4 상세 패널

메시지를 클릭하면 우측에 전체 디코딩 트리가 표시됩니다.
- **Ctrl+F** (Cmd+F): 상세 내용 내 검색
- **▲▼**: 검색 결과 이전/다음 이동
- **Copy**: 상세 내용 클립보드 복사

### 4.5 키보드

| 키 | 동작 |
|----|------|
| `↑` / `↓` | 이전/다음 메시지 선택 |

---

## 5. AT Command (Power Off/On)

캡처 중 **Power Off** 버튼을 클릭하면 `AT+CFUN=0` 명령으로 모뎀 RF를 끕니다.  
버튼이 **Power On**으로 변경되며, 클릭하면 `AT+CFUN=1`로 다시 켭니다.

모뎀 재시작 후 Attach/Registration 시퀀스를 처음부터 캡처할 수 있습니다.

---

## 6. 트러블슈팅

### 6.1 USB 디바이스 미감지

**공통:**
- USB 케이블 확인 (충전 전용 케이블은 데이터 전송 불가)
- DIAG 포트 활성화 확인 (2.1 디바이스 준비 참고)
- 다른 프로그램(QXDM, Wireshark 등)이 USB를 점유 중이면 해제

**Windows:**
- `run-wsl.bat`을 반드시 관리자 권한으로 실행
- usbipd가 정상 설치되었는지 확인: CMD에서 `usbipd list` 실행

### 6.2 Windows: BIOS 가상화 미활성화

**증상:** `setup-wsl.bat` 실행 시 `HCS_E_HYPERV_NOT_INSTALLED` 에러

**해결:**
1. PC 재시작 → BIOS 진입 (F2 또는 Del)
2. Intel VT-x 또는 AMD SVM 항목을 Enabled로 변경
3. 저장 후 재시작

또는: 설정 → 시스템 → 복구 → 고급 시작 → UEFI 펌웨어 설정에서 변경

**확인 방법:** 작업 관리자 → 성능 → CPU → "가상화: 사용" 표시 확인

### 6.3 Windows: 설치 중 재부팅 필요

**증상:** `setup-wsl.bat` 실행 후 "재부팅이 필요합니다" 메시지

WSL2에 필요한 VirtualMachinePlatform은 Windows 커널 기능이므로 최초 활성화 시 재부팅이 필수입니다.  
재부팅 후 `setup-wsl.bat`을 다시 실행하면 이어서 설치됩니다.

### 6.4 Windows: 브라우저에서 페이지 연결 불가

**증상:** 서버는 실행되었다고 표시되지만 브라우저에서 `ERR_CONNECTION_REFUSED` 또는 페이지가 열리지 않음

**원인:** WSL2는 가상 네트워크를 사용하며, Windows에서 localhost 접근은 WSL2 → Windows 포워딩에 의존합니다. 서버를 종료하지 않고 `run-wsl.bat`을 다시 실행하면 이 포워딩이 일시적으로 꼬일 수 있습니다.

**해결:**

1. 잠시 대기 후 (30초~1분) 브라우저 새로고침
2. 여전히 안 되면 CMD(관리자)에서 `wsl --shutdown` 실행 후 `run-wsl.bat` 재실행
3. 그래도 안 되면 Windows 재부팅

**예방:** 서버를 종료하려면 CMD 창에서 **Ctrl+C**를 사용하세요. CMD 창을 X 버튼으로 닫거나, 서버가 실행 중인 상태에서 `run-wsl.bat`을 다시 실행하면 문제가 발생할 수 있습니다.

### 6.5 Windows: USB 포트 미반환 (비정상 종료)

**증상:** DM Viewer 종료 후 QXDM 등에서 COM 포트가 보이지 않음

**원인:** CMD 창을 X 버튼으로 닫으면 USB unbind 과정이 생략되어 포트가 WSL에 잡힌 상태로 남음

**해결:**

1. `run-wsl.bat`을 다시 실행합니다 (시작 시 자동으로 이전 연결을 정리하고 다시 attach)
2. WSL에서 포트가 정상 인식되는 것을 확인
3. **Ctrl+C**로 정상 종료 → USB 포트가 Windows로 반환됨
4. 이후 QXDM 등에서 COM 포트 확인

**그래도 안 될 경우 (수동 정리):**
```cmd
usbipd detach --all
usbipd unbind --all
```
위 명령을 관리자 CMD에서 실행하면 모든 USB 바인딩이 해제되어 포트가 Windows로 돌아옵니다.

> 항상 **Ctrl+C**로 종료하면 이 문제가 발생하지 않습니다.

### 6.5 NR RRC 메시지 미표시 (scat.lua 관련)

**증상:** LTE-RRC는 표시되지만 NR-RRC 메시지만 안 나옴

**원인:** scat.lua 플러그인이 tshark에 로드되지 않으면 NR RRC 디코딩 불가

**macOS 해결:**
```bash
rm -rf .venv
bash run.command
```
venv 재생성 시 scat.lua가 자동으로 다시 설치됩니다.

**Windows 해결:**
`setup-wsl.bat`을 다시 실행하면 scat.lua가 재설치됩니다.

### 6.6 DIAG 세션 손상 (메시지 수신 중단)

**증상:** 캡처 중 갑자기 메시지가 더 이상 오지 않음 (scat은 살아있음)

**원인:** scat 비정상 종료 등으로 디바이스의 DIAG 세션이 손상됨

**해결:**
1. DM Viewer에서 **Disconnect**
2. 스마트폰 재부팅 (전원 Off → On)
3. USB 케이블을 물리적으로 뽑았다가 다시 연결
4. **Connect**로 재연결

### 6.7 scat 실행 실패 (최신 칩셋 미지원)

**증상:** 연결은 되지만 메시지가 하나도 안 나오고 scat stderr에 `Unknown NR RRC OTA Message packet version` 경고

**원인:** 설치된 scat 버전이 최신 Qualcomm 칩셋의 pkt_ver를 지원하지 않음

**해결:**
```bash
# macOS
rm -rf .venv
bash run.command

# Windows
# setup-wsl.bat 재실행 (scat upstream git에서 최신 버전 재설치)
```

### 6.8 포트 충돌 (UDP 4729 또는 TCP 8340)

**증상:** "Cannot bind UDP 4729" 또는 서버 시작 실패

**원인:** Wireshark, 이전 DM Viewer 프로세스 등이 포트를 점유 중

**macOS 해결:**
```bash
lsof -ti:8340 | xargs kill -9
lsof -ti:4729 | xargs kill -9
bash run.command
```

**Windows:** `run-wsl.bat`이 시작 시 자동으로 기존 프로세스를 정리합니다.

---

## 7. 종료 방법

### macOS
터미널에서 **Ctrl+C**

### Windows
CMD 창에서 **Ctrl+C** (USB 포트가 자동으로 Windows에 반환됨)

> ❌ CMD 창을 X 버튼으로 닫지 마세요. USB 포트 반환이 실패할 수 있습니다.

---

© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr> All rights reserved.

---

## 개정이력

| 버전 | 날짜 | 내용 |
|------|------|------|
| v1.0.0 | 2026-06-04 | 최초 배포 |
| v1.1.0 | 2026-06-04 | 런타임 USB 연결 관리 (Connect/Disconnect 시 USB attach/detach) |
