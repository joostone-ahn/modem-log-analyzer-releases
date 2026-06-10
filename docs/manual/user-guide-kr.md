# Modem Log Analyzer 사용자 가이드

**버전:** v1.6.3

---

## 목차

1. [프로그램 실행](#1-프로그램-실행)
2. [USB 실시간 캡처](#2-usb-실시간-캡처)
3. [파일 로드](#3-파일-로드)
4. [메시지 뷰어](#4-메시지-뷰어)
5. [트러블슈팅](#5-트러블슈팅)
6. [종료 방법](#6-종료-방법)

---

## 1. 프로그램 실행

### Windows (EXE)

1. [Releases](https://github.com/joostone-ahn/modem-log-analyzer-releases/releases)에서 최신 `Modem-Log-Analyzer-vX.X.X.exe`를 다운로드합니다
2. EXE를 **관리자 권한으로 실행**합니다 (우클릭 → 관리자 권한으로 실행 권장)
3. 스플래시 화면이 표시되며 환경을 자동 설정합니다
4. 완료 후 앱 윈도우가 자동으로 열립니다

#### 최초 설치 (WSL 미설치 PC)

새 PC에서는 EXE 실행 전에 `setup-wsl.bat`을 먼저 실행해야 합니다:

1. Release에서 `setup-wsl.bat`을 다운로드합니다
2. 우클릭 → **관리자 권한으로 실행**
3. WSL2 + Ubuntu 설치 완료 후 **재부팅** 안내 → PC 재부팅
4. 재부팅 후 `setup-wsl.bat`을 **한 번 더 실행** → 나머지 패키지 설치
5. "Setup complete!" 메시지 확인 → 이제 EXE를 실행하면 됩니다

> ⚠️ 재부팅 전에는 WSL이 활성화되지 않습니다. 반드시 재부팅 후 bat을 다시 실행하세요.

#### 이후 실행

- WSL 초기화 후 서버를 시작합니다 (약 10초)
- 이전 실행이 남아있으면 WSL을 리셋 후 재시작합니다 ("Resetting WSL..." 10~15초 소요)

⚠️ 최초 실행 시 EXE 옆에 `modem-log-analyzer-linux` 파일이 생성됩니다. 이 파일을 삭제하지 마세요.

### macOS (DMG)

> ⚠️ 사전 요구사항: [Homebrew](https://brew.sh)가 설치되어 있어야 합니다. 최초 실행 시 필요한 의존성(Wireshark, libusb 등)을 자동 설치합니다.

1. [Releases](https://github.com/joostone-ahn/modem-log-analyzer-releases/releases)에서 최신 `Modem-Log-Analyzer-vX.X.X-macOS.dmg`를 다운로드합니다
2. DMG 파일을 더블클릭하여 마운트합니다
3. `Modem Log Analyzer.app`을 Applications 폴더로 드래그합니다 (또는 원하는 위치에 복사)
4. 최초 실행 시 macOS Gatekeeper 경고가 표시됩니다:
   - `.app`을 **우클릭** → **열기** → **열기** 클릭
   - 이후 실행부터는 더블클릭으로 바로 열립니다
5. 스플래시 화면이 표시되며 의존성을 확인/설치합니다 (최초 1회, 2~3분 소요)
6. 준비 완료 후 앱 윈도우가 자동으로 열립니다

> 💡 Python, venv 등 사전 설치가 필요 없습니다 — standalone 앱입니다.

> 💡 `.app`을 Dock에 고정하면 빠르게 실행할 수 있습니다.

---

## 2. USB 실시간 캡처

### 2.1 디바이스 준비

스마트폰의 USB DIAG 포트를 활성화해야 합니다.

**Samsung:**
- 전화 다이얼러에서 `*#0808#` 입력 → DM이 포함된 USB 모드 선택 (예: DM+MODEM+ADB)
- 한국 내수 모델: 다이얼러에서 `3197123580` 입력, 비밀번호 `996412` 또는 `776432` 또는 `0821`

**Apple iPhone (Qualcomm 베이스밴드):**
- 설정 → Carrier Settings → Baseband Manager → Logging Settings → **Qualcomm**: Passive, External Hardware (QXDM) 선택

> **현재 검증된 기기:** Samsung Galaxy Z Fold7, iPhone 17 Pro (Qualcomm 칩셋)

### 2.2 연결 (Connect)

1. 단말을 USB로 PC에 연결합니다
2. **Windows:** 장치 관리자에서 포트가 인식되는지 확인합니다
3. 앱에서 **Connect** 버튼을 클릭합니다
4. 디바이스 감지 → USB를 WSL로 전달 → scat 시작 (약 8초)
5. 상태 표시가 녹색 ●으로 변경되고 메시지가 실시간으로 표시됩니다

> Connect 시 Windows의 USB 포트가 WSL로 넘어가므로 장치 관리자에서 사라집니다. 이는 정상입니다.

### 2.3 캡처 중 제어

| 버튼 | 동작 |
|------|------|
| Disconnect | 캡처 중지 + USB를 Windows로 반환 |
| Save | 캡처 데이터를 pcap 파일로 다운로드 |
| Auto-scroll | 새 메시지 도착 시 자동 스크롤 ON/OFF |
| Clear | 화면 메시지 삭제 (캡처는 계속됨) |

> **Power Off/On 버튼:** macOS에서만 표시됩니다. AT+CFUN=0/1 명령으로 모뎀 전원을 토글하여 Registration 시퀀스를 처음부터 캡처할 수 있습니다.

### 2.4 연결 해제 (Disconnect)

**Disconnect** 버튼을 클릭하면:
1. scat 캡처가 중지됩니다
2. USB 디바이스가 WSL에서 분리(detach + unbind)됩니다
3. Windows 장치 관리자에 포트가 다시 나타납니다
4. 다른 도구(QXDM 등)에서 포트를 사용할 수 있습니다

### 2.5 단말 교체

앱 재시작 없이 단말을 교체할 수 있습니다:

1. **Disconnect** 클릭
2. 장치 관리자에서 포트가 돌아온 것을 확인
3. 기존 단말을 분리하고 새 단말 연결
4. 장치 관리자에서 새 단말의 포트가 잡히는지 확인
5. **Connect** 클릭 → 새 단말로 캡처 시작

### 2.6 USB 비정상 분리 시 동작

캡처 중 USB가 물리적으로 빠지면:
- Watchdog이 2초 내에 감지합니다
- "USB disconnected" 에러가 표시됩니다
- 캡처가 자동 중지됩니다
- **Connect** 버튼이 다시 나타납니다
- 단말을 다시 연결하고 **Connect**를 누르면 됩니다

### 2.7 모드 전환 (USB ↔ File)

상단 툴바의 **USB Mode / File Mode** 토글로 전환합니다.

| 전환 방향 | 조건 | 동작 |
|-----------|------|------|
| USB(캡처 중) → File | 캡처 실행 중 | 차단됨 — 먼저 Disconnect 필요 |
| USB(데이터 有) → File | 데이터 존재 | 저장 다이얼로그 표시 (Save .pcap / Don't Save / Cancel) |
| USB(빈 상태) → File | 데이터 없음 | 즉시 전환 |
| File(결과 有) → USB | 파싱 결과 표시 중 | 확인 다이얼로그 표시 (Continue / Cancel) |
| File(빈 상태) → USB | 데이터 없음 | 즉시 전환 |

> 💡 모드 전환 시 **Save**를 선택하면 캡처 로그가 .pcap으로 다운로드됩니다.

---

## 3. 파일 로드

### 3.1 업로드

1. 상단에서 **File Mode** 탭을 선택합니다
2. **Open File** 버튼 클릭, 또는 파일을 화면에 드래그 앤 드롭합니다
3. 파싱 완료 후 메시지 목록이 표시됩니다

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
| `.pcap` | 원본 pcap 파일을 그대로 다운로드 |
| `.hdf` / `.qmdl` / `.dlf` / `.sdm` | scat으로 변환된 pcap 파일을 다운로드 |

**활용:**
- DM 로그를 pcap으로 변환하여 Wireshark에서 열 수 있습니다
- Wireshark에서 디코딩하려면 [scat.lua](https://raw.githubusercontent.com/fgsect/scat/master/wireshark/scat.lua) 플러그인이 필요합니다
- 적용: 다운로드 후 Wireshark 플러그인 디렉터리에 복사  
  - macOS/Linux: `~/.local/lib/wireshark/plugins/scat.lua`  
  - Windows: `%APPDATA%\Wireshark\plugins\scat.lua`

---

## 4. 메시지 뷰어

### 4.1 화면 구성

| 영역 | 설명 |
|------|------|
| 좌측 패널 | 메시지 목록 테이블 |
| 우측 패널 | 선택한 메시지의 상세 ASN.1 디코딩 |
| 패널 구분선 | 드래그하여 좌/우 비율 조절 |

### 4.2 메시지 테이블

| 컬럼 | 설명 |
|------|------|
| # | 프레임 번호 |
| Time | 타임스탬프 (HH:MM:SS.mmm) |
| Protocol | NR-RRC, LTE-RRC, NAS-5GS, NAS-EPS |
| Channel | DL-DCCH, UL-DCCH, BCCH-DL-SCH 등 |
| Message | 메시지 이름 |

프로토콜별 색상:
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

## 5. 트러블슈팅

### 5.1 "No DIAG devices found" (Connect 실패)

- USB 케이블 확인 (충전 전용 케이블은 데이터 전송 불가)
- DIAG 포트 활성화 확인 (2.1 참고)
- Windows 장치 관리자에서 포트가 잡히는지 먼저 확인
- 다른 프로그램(QXDM, Wireshark)이 USB를 점유하고 있으면 해제

### 5.2 연결 성공했지만 메시지가 안 나옴

**scat 미지원 칩셋:**
- 최신 EXE로 업데이트하세요 (scat upstream 최신 버전이 번들됨)

**DIAG 세션 손상 (삼성 단말에서 자주 발생):**
- 단말을 **재부팅** (전원 Off → On) 하세요
- 재부팅 후 USB 재연결 → **Connect**

**NR-RRC만 안 나오는 경우:**
- scat.lua 플러그인 문제. 최신 EXE를 재설치하세요

### 5.3 EXE 실행 시 "Waiting for server" timeout

이전 프로세스가 비정상 종료된 경우:

1. 작업 관리자에서 모든 `Modem-Log-Analyzer` 프로세스 종료
2. EXE 재실행

그래도 안 되면:
1. 관리자 CMD에서 `wsl --shutdown` 실행
2. EXE 재실행

### 5.4 Disconnect 후 장치 관리자에 포트가 안 나타남

보통은 정상적으로 Disconnect하면 포트가 돌아옵니다. 안 나타나면:

**방법 1: `reset-usb.bat` 사용 (권장)**

Release에 포함된 `reset-usb.bat`을 **관리자 권한으로 실행**합니다.
- WSL에 잡혀있는 모든 USB 디바이스를 Windows로 반환합니다
- WSL도 함께 종료하여 깨끗한 상태로 복구합니다

**방법 2: 수동 명령**

```cmd
:: 관리자 CMD에서 실행
usbipd unbind --all
```

### 5.5 비정상 종료 후 USB 복구

EXE를 강제 종료하거나 PC가 갑자기 꺼진 경우, USB가 WSL에 잡혀 있어 다른 프로그램(QXDM 등)에서 사용할 수 없습니다.

**`reset-usb.bat`을 관리자 권한으로 실행**하면 즉시 복구됩니다.

> 💡 `reset-usb.bat`은 Release 페이지에서 EXE와 함께 다운로드할 수 있습니다.

### 5.6 디버그 로그 수집

문제 발생 시 EXE 옆에 생성되는 `modem-log-analyzer.log` 파일을 확인하세요.
- 매 실행마다 새로 생성됩니다 (이전 로그는 덮어씌워짐)
- 서버 시작, USB 연결, 캡처, 에러 등 모든 내부 동작이 기록됩니다
- 이슈 보고 시 이 파일을 함께 전달해 주세요

### 5.7 단말 교체 후 Connect 실패

1. 장치 관리자에서 새 단말이 보이는지 확인
2. 안 보이면 USB를 뽑았다 다시 꽂기
3. 보이면 **Connect** 재시도

---

## 6. 종료 방법

앱 윈도우를 닫으면 프로그램이 종료됩니다.

- **macOS:** `⌘Q` 또는 창 닫기 버튼 (빨간 ●) 클릭
- **Windows:** 우측 상단 **X** 버튼 클릭 또는 `Alt+F4`

창을 닫으면:
1. 캡처 중이면 자동 중지
2. USB 디바이스가 Windows로 반환됨 (detach + unbind)
3. 서버가 종료됨

> **참고:** Windows에서 작업 관리자를 통해 프로세스를 종료해도 동일하게 동작합니다.

> **재시작:** 종료 후 다시 사용하려면 EXE(Windows) 또는 .app(macOS)을 다시 실행하세요.

---

**© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr>. Licensed under GPLv2.**
