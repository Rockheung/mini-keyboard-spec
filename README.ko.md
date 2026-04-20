# MINI KeyBoard 키 매핑 프로토콜 (사양 문서)

> English: [README.md](README.md)

중국산 미니 매크로 키보드(소위 "3key / 미니 키패드")의 **공식 Windows 설정 프로그램이
해당 개체에서 작동하지 않을 때**, 매핑을 직접 제어할 수 있도록 USB HID 프로토콜을 정리한 사양 문서입니다.

> 이 저장소는 **사양(spec) 문서만 공개**합니다. 실행 코드·역공학 원본은 포함하지 않습니다.
> 문서를 읽고 원하는 언어로 직접 구현하여 사용하세요.

## 대상 기기 특성

이 문서가 적용될 가능성이 높은 장치의 식별 포인트:

- USB HID composite device (인터페이스 2개)
- 인터페이스 0: vendor-defined HID (usage page `0xFF00`) ← **설정 채널**
- 인터페이스 1: 표준 Keyboard/Mouse/Consumer HID ← **실제 입력**
- Output Report 크기 = 65 bytes (ReportID 1 + payload 64)
- Product string에 `USB Keyboard` 등 비영문(중국어) 문자가 섞여 깨져 보이기도 함

**확인된 VID/PID 예:**
- `0x1189` / `0x8810, 0x8830~0x8834, 0x8840, 0x8890` — 공식 설정 프로그램이 탐색하는 목록
- `0x514C` / `0x8851` — 같은 펌웨어 계열의 다른 VID 빌드 (본 사양 실기기 검증 대상)

> VID가 다르더라도 Report 구조·opcode·키 테이블이 동일하면 본 사양이 그대로 적용됩니다.

## 문서

- [docs/protocol.md](docs/protocol.md) — HID 프로토콜 상세 (패킷 구조, opcode, 워크플로우)
- [docs/hid-usage-codes.md](docs/hid-usage-codes.md) — 키 → USB HID Usage 코드 테이블

## 구현 시 핵심 요약

1. Interface 0 (usage page `0xFF00`)을 연다.
2. **65바이트** output report에:
   - byte 0: Report ID (`3`을 우선 시도, 실패 시 `0` → `2` 순)
   - byte 1 이후: payload (아래 프로토콜 문서 참조)
3. 매핑 패킷(`opcode 0xFE`) 전송 → Flash commit 패킷(`0xAA 0xAA`) 전송.
4. 즉시 영구 반영됨 (flash write).

## 주의

- **Flash에 직접 쓰는 동작입니다.** 잘못된 패킷으로 설정이 망가질 수 있습니다. 개별 책임.
- 같은 모델이라도 **개체마다 VID/PID가 다를 수 있습니다.** 구현 전에 반드시 본인 개체의 VID/PID를 확인하세요.
- Chrome, ChatGPT Atlas 등 **Chromium 기반 앱이 WebHID로 장치를 점유**하면 OS HID 목록에서 누락될 수 있습니다. 재연결하거나 해당 앱을 종료.

## 면책

- 이 문서는 **USB HID 동작을 블랙박스로 관찰·정리한 사양서**입니다.
- 특정 제조사·브랜드·판매자와 **무관**하며, 비공식 문서입니다.
- 사양 사용 및 구현으로 발생하는 장치 손상·데이터 손실·법적 문제는 **구현자 본인 책임**입니다.
- 원본 제조사의 저작물(바이너리·소스·상표)은 포함하지 않습니다.

## 라이선스

문서: [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) (퍼블릭 도메인)
— 자유롭게 복제·수정·상업적 사용 가능.
