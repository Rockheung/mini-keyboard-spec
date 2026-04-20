# MINI KeyBoard Key Remapping Protocol (Spec)

> 한국어 버전: [README.ko.md](README.ko.md)

USB HID protocol specification for Chinese-made mini macro keyboards (commonly
sold as "3-key" / "mini keypad"), enabling direct key remapping when the
**official Windows configurator fails to recognize your specific unit**.

> This repository contains **the specification only** — no executable code or
> reverse-engineered source is included. Read the spec and implement it in the
> language of your choice.

## Target Device Fingerprint

Likely applies to your device if:

- USB HID composite device with two interfaces
- Interface 0: vendor-defined HID (usage page `0xFF00`) ← **configuration channel**
- Interface 1: standard Keyboard / Mouse / Consumer HID ← **actual input**
- Output Report size = 65 bytes (ReportID 1 + payload 64)
- Product string often shows something like `USB Keyboard` with trailing garbled
  (Chinese) characters

**Known VID/PID combinations:**

| VID | PID | Notes |
|-----|-----|-------|
| `0x1189` | `0x8810`, `0x8830~0x8834`, `0x8840`, `0x8890` | Targeted by the official Windows configurator |
| `0x514C` | `0x8851` | Same firmware family, different VID build — verified on real hardware |

> Even if your VID differs, if the report structure, opcodes, and key tables
> match, this spec should apply unchanged.

## Documents

- [docs/protocol.md](docs/protocol.md) — HID protocol details (packet structure, opcodes, workflow)
- [docs/hid-usage-codes.md](docs/hid-usage-codes.md) — Key → USB HID Usage code table

## TL;DR for Implementers

1. Open Interface 0 (usage page `0xFF00`).
2. Send a **65-byte** output report:
   - byte 0: Report ID (try `3` first, fall back to `0` → `2` on failure)
   - byte 1 onward: payload (see the protocol doc)
3. Send the remap packet (opcode `0xFE`), then the flash-commit packet (`0xAA 0xAA`).
4. Change is applied immediately and persists across reboots.

## Cautions

- **This writes to the device's flash.** Malformed packets can corrupt
  configuration. Use at your own risk.
- Even within the same model, **VID/PID may differ between units.** Always
  verify your unit's VID/PID before implementing.
- **Chromium-based apps** (Chrome, ChatGPT Atlas, etc.) may hold the device
  open via WebHID, causing it to disappear from the OS HID enumeration list.
  Replug the keyboard or quit the offending app.

## Disclaimer

- This is a **black-box observation and specification** of USB HID behavior.
- Unaffiliated with any manufacturer, brand, or reseller. Unofficial.
- No warranty. Any device damage, data loss, or legal issue arising from use
  or implementation is **your own responsibility**.
- No manufacturer's copyrighted material (binaries, source code, trademarks)
  is included in this repository.

## License

Documents are released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/)
(public domain) — free to copy, modify, and use commercially.
