# Dalgak

**중계 서버와 계정 없이, 기기끼리 직접 연결하는 개인용 P2P 원격 데스크톱** (Windows · Linux)

- **다운로드 페이지**: https://n837.github.io/DalgakVNC/
- **최신 릴리스**: https://github.com/n837/DalgakVNC/releases/latest
- **변경 내역 / 설치 안내**: https://n837.github.io/DalgakVNC/

---

## 주요 특징

### 1. 서버·계정 없는 순수 P2P 직접 연결
- **같은 네트워크 자동 발견**: 같은 Wi-Fi·공유기에 있는 Dalgak PC를 자동으로 찾아 목록에 표시합니다(UDP `38451`).
- **상호 TLS 1.2 직접 통신**: 화면·입력·파일·소리가 모두 기기 간 암호화 연결(TCP `38450`)로 직접 오가며, 중앙 중계 서버를 거치지 않습니다.
- **집 밖 연결 지원**: 원할 때만 공유기 포트 자동 열기(UPnP / PCP / NAT-PMP)와 IPv6, 일회용 초대 문자열(`dalgak://v1/...`)로 외부망에서도 직접 연결할 수 있습니다.
- **반대 방향 도와주기**: 포트가 막힌 네트워크(CGNAT 등)의 PC를 도울 때는 도와주는 쪽이 기다리고, 도움받는 쪽에서 먼저 연결해 수락할 수 있습니다.
- **짝지은 기기 홈 카드**: 한 번 짝지은 기기는 홈에 카드로 남아 온라인 배지 확인, 즐겨찾기, 원클릭 연결, SSH 중계, 원격 켜기(Wake-on-LAN)를 바로 쓸 수 있습니다.

### 2. 빠른 원격 화면 · 원격 소리
- **Windows 고속 파이프라인**: DXGI Desktop Duplication, 연속 갱신(ContinuousUpdates), GPU H.264 인코딩 및 DXVA 하드웨어 디코딩, D3D11 SwapChain 직접 렌더링으로 CPU 부하와 전송량을 크게 줄였습니다.
- **Linux 양방향 H.264**: 시스템 `ffmpeg` / `libavcodec`·`libswscale`이 있으면 Windows ↔ Linux 양방향 H.264 전송을 자동 사용하고, 없으면 Tight+JPEG로 자동 폴백합니다.
- **원격 소리(오디오) 전송**: Windows WASAPI 루프백과 Linux PipeWire/PulseAudio를 IMA-ADPCM 4:1 압축·무음 억제로 전송합니다. 원격 화면 상단의 **소리** 토글로 켜고 끌 수 있습니다(기본 꺼짐).

### 3. 실사용 편의 기능
- **파일 주고받기**: 조작이 허용된 세션에서 파일을 송수신할 수 있습니다(Windows 수신 파일에 인터넷 출처 표시 `Zone.Identifier` 자동 적용).
- **선택적 클립보드 공유**: 연결 수락 창에서 직접 허용한 경우에만 텍스트 클립보드를 공유합니다.
- **원클릭 자동 업데이트**: 앱 안에서 새 버전을 확인하고 서명·해시 검증 후 바로 업데이트할 수 있습니다.

---

## 보안 설계

- **기기 인증서 핀닝**: 각 기기가 자체 ECDSA P-256 인증서를 생성하며, 인증서 DER의 SHA-256 해시를 기기 ID로 검증합니다. 개인키와 짝지은 기기 목록은 Windows DPAPI(`CurrentUser`) 및 Linux `0600` 권한으로 보호됩니다.
- **일회용 코드 증명**: 평문 코드를 전송하지 않고 `HMAC-SHA256`으로 양측 기기 ID에 바인딩해 검증합니다. 코드는 2분마다 자동 갱신되며(초대는 10분), 5회 오입력 시 즉시 교체되고 반복 실패 IP는 일정 시간 차단됩니다.
- **명시적 수락과 최소 권한**: 연결 요청 창의 기본 선택은 **거절**이며 60초 안에 수락하지 않으면 자동 만료됩니다. ‘수락 없이 연결’은 받는 PC에서 직접 허용한 기기에만 적용되며, 이 경우 클립보드 공유는 항상 차단됩니다.
- **오프라인 서명 검증 업데이트**: 앱 내 자동 업데이트는 `n837/DalgakVNC`의 GitHub Releases만 허용하며, 오프라인 소유자 키의 **RSA-3072 + SHA-256** 서명과 파일 크기·해시가 모두 일치할 때만 설치합니다.

> **안전 주의**: 전화·메시지로 원격 연결 설치나 코드·초대 전달을 요구하는 낯선 사람에게는 절대 코드나 초대를 알려 주지 마세요. 연결 중에도 상단 바 또는 **받기 중지**로 언제든 즉시 종료할 수 있습니다.

---

## 지원 환경 및 설치

### Windows (x64 · .NET Framework 4.8 내장)
1. [다운로드 페이지](https://n837.github.io/DalgakVNC/)에서 **`Dalgak-Setup.exe`**(설치 파일) 또는 **`PersonalVnc.Windows.zip`**(무설치 포터블)을 받습니다.
2. 설치 파일은 현재 사용자 계정에 설치되며 관리자 권한을 요구하지 않습니다. (코드 서명이 없는 개인 배포 프로그램이므로 Windows SmartScreen 경고가 뜨면 게시 내용과 SHA-256을 확인한 뒤 실행하세요.)
3. 제거는 **Windows 설정 → 설치된 앱**에서 할 수 있습니다.

### Linux (Debian · Ubuntu · Raspberry Pi OS 등 GTK3 데스크톱)
1. [다운로드 페이지](https://n837.github.io/DalgakVNC/)에서 **`Dalgak.Linux.tar.gz`**를 받습니다.
2. 압축을 푼 뒤 설치 스크립트를 실행합니다:
   ```bash
   tar -xzf Dalgak.Linux.tar.gz
   cd dalgak-linux-*
   ./install.sh
   ```
3. 앱 메뉴의 **Dalgak** 또는 터미널에서 `dalgak`으로 실행합니다. 제거는 `~/.local/share/dalgak/uninstall.sh` 또는 앱 메뉴의 **Dalgak 제거**를 사용하세요.

---

## 약관 · 개인정보 · 오픈소스 고지

- [이용약관](https://n837.github.io/DalgakVNC/terms.html)
- [개인정보 처리방침](https://n837.github.io/DalgakVNC/privacy.html)
- [오픈소스 라이선스 고지](https://n837.github.io/DalgakVNC/licenses/)

---

## 저장소 및 저작권 안내

이 공개 저장소(`n837/DalgakVNC`)는 Dalgak의 공식 다운로드 페이지(GitHub Pages)와 릴리스 설치 파일(GitHub Releases) 배포 전용 저장소입니다. 소스 코드는 비공개로 관리됩니다.

Copyright (c) 2026 n837. All rights reserved.  
Dalgak은 개인 프로젝트이며, 저작권자의 사전 서면 허락 없이 이 소프트웨어의 복제·수정·재배포·판매를 금합니다. 포함된 제3자 오픈소스 구성 요소는 각자의 라이선스를 따릅니다.
