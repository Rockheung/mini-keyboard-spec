# MINI KeyBoard HID 프로토콜

USB HID vendor-defined 인터페이스를 통한 키 매핑·LED·레이어 설정 프로토콜.

---

## 1. USB 장치 식별

Composite USB device. 두 개의 HID 인터페이스:

| Interface | Usage Page | 역할 |
|-----------|-----------|------|
| `0` | `0xFF00` (vendor-defined) | **설정 채널** (본 프로토콜이 통신하는 곳) |
| `1` | `0x01` (Keyboard) + `0x0C` (Consumer) | 실제 키 입력 |

### VID/PID 예

| VID | PID | 비고 |
|-----|-----|------|
| `0x1189` | `0x8890` | mi_01 인터페이스에 설정 채널, Protocol Type 0 |
| `0x1189` | `0x8810, 0x8830~0x8834, 0x8840` | mi_00 인터페이스, Protocol Type 1 |
| `0x514C` | `0x8851` | Interface 0에 설정 채널, Protocol Type 1 (실기기 검증됨) |

> Windows 설정 프로그램은 device path에 `mi_00` / `mi_01` 문자열로 인터페이스를 구분하며,
> Linux/macOS에서는 hidapi의 `interface_number`로 동일하게 구분 가능.

---

## 2. HID Output Report 구조

```
┌────────────┬──────────────────────────────────────────┐
│ ReportID   │ Payload                                  │
│ 1 byte     │ 64 bytes                                 │
└────────────┴──────────────────────────────────────────┘
총 65 bytes
```

- **Report ID**: `0`, `2`, `3` 중 하나. 공식 프로그램은 **3 → 0 → 2** 순으로 재시도하고
  성공한 값을 기억해 재사용. 검증된 실기기는 `3`이 바로 성공.
- **Payload 유효 길이**:
  - Protocol Type 0: 첫 8바이트만 사용
  - Protocol Type 1: 첫 46바이트까지 사용
- 전송 크기는 항상 65바이트 (나머지는 0 padding).

---

## 3. 명령 Opcode (payload[0])

| `payload[0]` | `payload[1]` | 의미 |
|--------------|--------------|------|
| `0xA1` | layer (1~3) | 레이어 전환 |
| `0xAA` | `0xAA` | Flash 저장 (일반) |
| `0xAA` | `0xA1` | Flash 저장 (LED 설정) |
| `0xFE` | 키번호 | 키 재매핑 (Protocol Type 1) |
| `1~16` | 유형 | 키 재매핑 (Protocol Type 0, 구형) |

---

## 4. 키 매핑 패킷 상세

### 4.1 Protocol Type 1 (46B, opcode 0xFE)

매핑 한 건을 한 번의 write로 완결:

```
offset  | 의미
--------|--------------------------------------------------
[0]     | 0xFE                        (opcode)
[1]     | 키번호 (1 ~ N)              (어느 물리 버튼, 1-based)
[2]     | 레이어 (1~3)
[3]     | 키유형 (KeyType)            (0, 1: 일반키 / 2: Mouse / 3: Multimedia / 8: LED)
[4]     | reserved                    (0으로 두어도 동작 확인됨)
[5]     | reserved                    (0으로 두어도 동작 확인됨)
[6..8]  | 0
[9]     | 조합키 개수 (0 ~ 18)
[10]    | Modifier byte (첫 번째)
[11]    | HID Usage   (첫 번째)
[12]    | Modifier byte (두 번째)
[13]    | HID Usage   (두 번째)
...
[44]    | Modifier byte (18번째)
[45]    | HID Usage   (18번째)
```

**조합키 개수 계산**: `[10..11]`부터 순차적으로 채우며 non-zero 쌍이 있을 때마다 `[9]`를 1 증가.
공식 프로그램은 이 카운팅을 자동 처리.

### 4.2 Protocol Type 0 (8B, 구형)

조합키를 **한 쌍씩 여러 번** write:

```
offset  | 의미
--------|--------------------------------------------------
[0]     | 키번호 (layer==1이면 keytype nibble 그대로)
        | layer>1이면: (layer << 4) | keytype
[1]     | keytype nibble
[2]     | 조합 개수 (총 쌍 수)
[3]     | 현재 인덱스 (0부터)
[4]     | Modifier byte
[5]     | HID Usage
```

각 조합 쌍마다 write → 마지막에 `[0xAA, 0xAA]` flash 저장.

### 4.3 KeyType 값

| `KeyType & 0xF` | 의미 | 추가 바이트 |
|-----------------|------|-------------|
| 0, 1 | 일반 키보드 키 | Modifier + HID Usage |
| 2 | 마우스 버튼/휠 | 2바이트 (버튼 비트마스크 + 휠) |
| 3 | 멀티미디어 (Consumer) | 5바이트 |
| 8 | LED | 1바이트 (LED 모드) |

---

## 5. Modifier Byte

**표준 USB HID Keyboard Modifier와 100% 동일**:

| Bit | 값 | 키 |
|-----|------|------|
| 0 | `0x01` | Left Ctrl |
| 1 | `0x02` | Left Shift |
| 2 | `0x04` | Left Alt |
| 3 | `0x08` | Left GUI (Win/Cmd) |
| 4 | `0x10` | Right Ctrl |
| 5 | `0x20` | Right Shift |
| 6 | `0x40` | Right Alt |
| 7 | `0x80` | Right GUI |

여러 modifier는 OR하여 결합. 예: `Ctrl+Shift` = `0x03`.

---

## 6. HID Usage 코드 (키)

**표준 USB HID Usage Page 0x07 (Keyboard/Keypad) 그대로 사용.**

| 키 | Usage | 키 | Usage |
|----|-------|----|-------|
| A | 4 | Enter | 40 |
| B | 5 | Esc | 41 |
| C | 6 | Backspace | 42 |
| ... | ... | Tab | 43 |
| Z | 29 | Space | 44 |
| 1 | 30 | F1~F12 | 58~69 |
| 2 | 31 | PrtSc | 70 |
| ... | ... | ScrollLock | 71 |
| 0 | 39 | Pause | 72 |

전체 테이블: [hid-usage-codes.md](hid-usage-codes.md).

### Shift 조합 (숫자키 상단 특수문자)

Shift 비트(`0x02`)를 Modifier에 OR하고 해당 숫자 Usage 전송:

- `!` = Shift(`0x02`) + Usage 30 (1)
- `@` = Shift(`0x02`) + Usage 31 (2)
- ...

---

## 7. 멀티미디어 키 (Consumer Usage Page 0x0C)

KeyType = 3, payload에 Consumer Usage 값:

| 버튼 | 값 |
|------|-----|
| Play/Pause | 205 |
| Stop | 182 (추정) |
| Next | 181 |
| Mute | 226 |
| Volume Up | 233 |
| Volume Down | 234 (추정) |

> 일부 값은 공식 프로그램에서 추출했으나 실기기 재현 검증 전. Consumer Usage Page 표준 참고.

---

## 8. LED 명령

- KeyType = 8
- payload[2]에 LED 모드 값 (0~5 범위로 보이는 색상 프리셋)
- Flash 저장은 `[0xAA, 0xA1]` (일반 `[0xAA, 0xAA]`와 다름)

색상 프리셋 UI 레이블: Red, Orange, Yellow, Green, Cyan, Blue, Purple.

---

## 9. 전체 매핑 워크플로우

1. **USB 장치 열기**: 본인 개체의 VID/PID, Interface 0 (usage page `0xFF00`).
2. **매핑 패킷**: Protocol Type 1 기준 opcode `0xFE` 패킷 전송 (위 4.1).
3. **Flash 저장**: `[0xAA, 0xAA]` 패킷 전송.
4. 완료. 재부팅·재연결해도 유지됨.

레이어 전환은 중간에 `[0xA1, layer]` 단독 전송.

---

## 10. 실기기 검증 결과

**검증 장치**: VID `0x514C` / PID `0x8851` 2개체, macOS 환경, hidapi 사용.

확정된 사항:

- **ReportID = 3** — retry 없이 첫 시도에 성공. retry 로직은 fallback으로 유지 권장.
- **키 번호는 1-based** — KEY1 = `payload[1] = 1`. 문서 4.1의 표 그대로 적용.
- **payload[4], [5] = 0** 으로 두고 일반 키 매핑 성공.
- **일반 키 매핑(key_type=1)** + **Modifier + HID Usage 쌍** — 실제 동작 확인.
- **Flash commit `[0xAA, 0xAA]`** — 전송 후 즉시 영구 반영.

**미검증 (문서 값만)**:

- 레이어 전환 (`[0xA1, layer]`)
- Mouse 매핑 (key_type=2)
- Multimedia 매핑 (key_type=3)
- LED 매핑 (key_type=8, flash commit `[0xAA, 0xA1]`)

---

## 구현 참고

최소 매핑 전송 의사 코드:

```
// 매핑 패킷: KEY1을 'A'로
buf = zeros(65)
buf[0]  = 3        // Report ID
buf[1]  = 0xFE     // opcode
buf[2]  = 1        // 키 번호
buf[3]  = 1        // 레이어
buf[4]  = 1        // KeyType (일반)
buf[10] = 1        // 조합 개수
buf[11] = 0x00     // modifier
buf[12] = 0x04     // HID Usage = 'A'
device.write(buf)

sleep(50ms)

// Flash commit
buf = zeros(65)
buf[0] = 3
buf[1] = 0xAA
buf[2] = 0xAA
device.write(buf)
```
