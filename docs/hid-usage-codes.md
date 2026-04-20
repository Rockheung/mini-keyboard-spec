# HID Usage 코드 테이블

이 키보드의 모든 키 매핑 값은 **표준 USB HID Usage Page 0x07 (Keyboard/Keypad)**와
완전히 동일합니다. USB-IF HID Usage Tables 공식 문서(`HUT 1.12`)의 해당 페이지와 호환됩니다.

## 알파벳

| 키 | Usage | 키 | Usage |
|----|-------|----|-------|
| A | 4 | N | 17 |
| B | 5 | O | 18 |
| C | 6 | P | 19 |
| D | 7 | Q | 20 |
| E | 8 | R | 21 |
| F | 9 | S | 22 |
| G | 10 | T | 23 |
| H | 11 | U | 24 |
| I | 12 | V | 25 |
| J | 13 | W | 26 |
| K | 14 | X | 27 |
| L | 15 | Y | 28 |
| M | 16 | Z | 29 |

## 숫자 (메인 행)

| 키 | Usage | Shift 조합 결과 |
|----|-------|-----------------|
| 1 | 30 | ! |
| 2 | 31 | @ |
| 3 | 32 | # |
| 4 | 33 | $ |
| 5 | 34 | % |
| 6 | 35 | ^ |
| 7 | 36 | & |
| 8 | 37 | * |
| 9 | 38 | ( |
| 0 | 39 | ) |

## 제어 키

| 키 | Usage |
|----|-------|
| Enter | 40 |
| Esc | 41 |
| Backspace | 42 |
| Tab | 43 |
| Space | 44 |
| `- _` | 45 |
| `= +` | 46 |
| `[ {` | 47 |
| `] }` | 48 |
| `\ \|` | 49 |
| `; :` | 51 |
| `' "` | 52 |
| `` ` ~ `` | 53 |
| `, <` | 54 |
| `. >` | 55 |
| `/ ?` | 56 |
| CapsLock | 57 |

## 펑션 키

| 키 | Usage | 키 | Usage |
|----|-------|----|-------|
| F1 | 58 | F7 | 64 |
| F2 | 59 | F8 | 65 |
| F3 | 60 | F9 | 66 |
| F4 | 61 | F10 | 67 |
| F5 | 62 | F11 | 68 |
| F6 | 63 | F12 | 69 |

## 편집/네비게이션

| 키 | Usage |
|----|-------|
| PrintScreen | 70 |
| ScrollLock | 71 |
| Pause/Break | 72 |
| Insert | 73 |
| Home | 74 |
| PageUp | 75 |
| Delete | 76 |
| End | 77 |
| PageDown | 78 |
| RightArrow | 79 |
| LeftArrow | 80 |
| DownArrow | 81 |
| UpArrow | 82 |

## 키패드 (Keypad)

| 키 | Usage |
|----|-------|
| NumLock | 83 |
| Kp / | 84 |
| Kp * | 85 |
| Kp - | 86 |
| Kp + | 87 |
| Kp Enter | 88 |
| Kp 1 | 89 |
| Kp 2 | 90 |
| Kp 3 | 91 |
| Kp 4 | 92 |
| Kp 5 | 93 |
| Kp 6 | 94 |
| Kp 7 | 95 |
| Kp 8 | 96 |
| Kp 9 | 97 |
| Kp 0 | 98 |
| Kp . | 99 |

## Modifier Usage (개별 키로 매핑할 때)

Modifier byte(비트마스크)가 아닌 **일반 Usage 코드로 매핑**할 때의 값:

| 키 | Usage |
|----|-------|
| LCtrl | 224 |
| LShift | 225 |
| LAlt | 226 |
| LWin (GUI) | 227 |
| RCtrl | 228 |
| RShift | 229 |
| RAlt | 230 |
| RWin (GUI) | 231 |

> 일반적으로는 Usage `0` + Modifier byte 조합으로 표현하는 쪽이 표준.
> (ex. Ctrl+C = modifier `0x01`, usage `0x06`.)
