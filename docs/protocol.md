# MINI KeyBoard HID Protocol

Specification for key remapping, LED, and layer control over a USB HID
vendor-defined interface.

---

## 1. USB Device Identification

Composite USB device. Two HID interfaces:

| Interface | Usage Page | Role |
|-----------|-----------|------|
| `0` | `0xFF00` (vendor-defined) | **Configuration channel** (where this protocol lives) |
| `1` | `0x01` (Keyboard) + `0x0C` (Consumer) | Actual keystroke output |

### Known VID/PIDs

| VID | PID | Notes |
|-----|-----|-------|
| `0x1189` | `0x8890` | Config channel on `mi_01` interface, Protocol Type 0 |
| `0x1189` | `0x8810, 0x8830~0x8834, 0x8840` | `mi_00` interface, Protocol Type 1 |
| `0x514C` | `0x8851` | Config channel on Interface 0, Protocol Type 1 (verified on hardware) |

> The official Windows configurator distinguishes interfaces by searching for
> `mi_00` / `mi_01` in the device path. On Linux/macOS, hidapi's
> `interface_number` gives the same information.

---

## 2. HID Output Report Structure

```
┌────────────┬──────────────────────────────────────────┐
│ Report ID  │ Payload                                  │
│ 1 byte     │ 64 bytes                                 │
└────────────┴──────────────────────────────────────────┘
Total: 65 bytes
```

- **Report ID**: one of `0`, `2`, `3`. The official app tries **3 → 0 → 2**
  and remembers the first success. On verified hardware, `3` works on the
  first attempt.
- **Effective payload length**:
  - Protocol Type 0: first 8 bytes used
  - Protocol Type 1: first 46 bytes used
- Always transmit 65 bytes total; remaining bytes should be zero-padded.

---

## 3. Command Opcodes (payload[0])

| `payload[0]` | `payload[1]` | Meaning |
|--------------|--------------|---------|
| `0xA1` | layer (1~3) | Switch active layer |
| `0xAA` | `0xAA` | Flash commit (normal) |
| `0xAA` | `0xA1` | Flash commit (LED settings) |
| `0xFE` | key number | Key remap (Protocol Type 1) |
| `1~16` | type | Key remap (Protocol Type 0, legacy) |

---

## 4. Key Remap Packet — Details

### 4.1 Protocol Type 1 (46B, opcode 0xFE)

Entire remap in a single write:

```
offset  | meaning
--------|--------------------------------------------------
[0]     | 0xFE                        (opcode)
[1]     | key number (1 ~ N)          (which physical button, 1-based)
[2]     | layer (1 ~ 3)
[3]     | KeyType                     (0, 1: normal key / 2: mouse / 3: multimedia / 8: LED)
[4]     | reserved                    (zero works on verified hardware)
[5]     | reserved                    (zero works on verified hardware)
[6..8]  | 0
[9]     | combo count (0 ~ 18)
[10]    | Modifier byte (1st)
[11]    | HID Usage   (1st)
[12]    | Modifier byte (2nd)
[13]    | HID Usage   (2nd)
...
[44]    | Modifier byte (18th)
[45]    | HID Usage   (18th)
```

**Combo count**: the official app fills `[10..11]` onward sequentially and
increments `[9]` for each non-zero pair. You must populate `[9]` yourself.

### 4.2 Protocol Type 0 (8B, legacy)

Write one combo pair per packet:

```
offset  | meaning
--------|--------------------------------------------------
[0]     | key number (if layer==1: keytype nibble as-is)
        | if layer>1: (layer << 4) | keytype
[1]     | keytype nibble
[2]     | total combo count
[3]     | current index (0-based)
[4]     | Modifier byte
[5]     | HID Usage
```

Send one write per combo pair, then `[0xAA, 0xAA]` to flash-commit.

### 4.3 KeyType Values

| `KeyType & 0xF` | Meaning | Extra bytes |
|-----------------|---------|-------------|
| 0, 1 | Normal keyboard key | Modifier + HID Usage |
| 2 | Mouse button / wheel | 2 bytes (button bitmask + wheel) |
| 3 | Multimedia (Consumer) | 5 bytes |
| 8 | LED | 1 byte (LED mode) |

---

## 5. Modifier Byte

**Identical to the standard USB HID Keyboard Modifier byte:**

| Bit | Value | Key |
|-----|-------|-----|
| 0 | `0x01` | Left Ctrl |
| 1 | `0x02` | Left Shift |
| 2 | `0x04` | Left Alt |
| 3 | `0x08` | Left GUI (Win / Cmd) |
| 4 | `0x10` | Right Ctrl |
| 5 | `0x20` | Right Shift |
| 6 | `0x40` | Right Alt |
| 7 | `0x80` | Right GUI |

Combine multiple modifiers with bitwise OR. Example: `Ctrl+Shift` = `0x03`.

---

## 6. HID Usage Codes (Keys)

**The device uses standard USB HID Usage Page 0x07 (Keyboard/Keypad) values
directly.**

| Key | Usage | Key | Usage |
|-----|-------|-----|-------|
| A | 4 | Enter | 40 |
| B | 5 | Esc | 41 |
| C | 6 | Backspace | 42 |
| ... | ... | Tab | 43 |
| Z | 29 | Space | 44 |
| 1 | 30 | F1~F12 | 58~69 |
| 2 | 31 | PrtSc | 70 |
| ... | ... | ScrollLock | 71 |
| 0 | 39 | Pause | 72 |

Full table: [hid-usage-codes.md](hid-usage-codes.md).

### Shift Combinations (top-row symbols)

OR the Shift bit (`0x02`) into the Modifier byte and send the corresponding
number's Usage:

- `!` = Shift (`0x02`) + Usage 30 (1)
- `@` = Shift (`0x02`) + Usage 31 (2)
- ...

---

## 7. Multimedia Keys (Consumer Usage Page 0x0C)

KeyType = 3, payload carries Consumer Usage values:

| Button | Value |
|--------|-------|
| Play / Pause | 205 |
| Stop | 182 (tentative) |
| Next | 181 |
| Mute | 226 |
| Volume Up | 233 |
| Volume Down | 234 (tentative) |

> Values extracted from the official app. Not yet verified on real hardware.
> Cross-reference the Consumer Usage Page standard if something misbehaves.

---

## 8. LED Commands

- KeyType = 8
- `payload[2]` holds the LED mode value (range 0~5, corresponding to color presets)
- Flash commit uses `[0xAA, 0xA1]` (different from the normal `[0xAA, 0xAA]`)

UI color preset labels: Red, Orange, Yellow, Green, Cyan, Blue, Purple.

---

## 9. Full Remap Workflow

1. **Open the USB device**: your unit's VID/PID, Interface 0 (usage page `0xFF00`).
2. **Send the remap packet**: opcode `0xFE` Protocol Type 1 packet (see 4.1).
3. **Flash commit**: send `[0xAA, 0xAA]`.
4. Done. Persists across reboots and replug.

To switch layers, send `[0xA1, layer]` as a standalone packet.

---

## 10. Hardware-Verified Findings

**Verified on**: two keyboards with VID `0x514C` / PID `0x8851`, on macOS,
using hidapi.

Confirmed:

- **Report ID = 3** — first attempt succeeds; retry fallback is still
  recommended for other firmware builds.
- **Key number is 1-based** — KEY1 corresponds to `payload[1] = 1`. Table 4.1
  applies as-is.
- **`payload[4], [5] = 0`** works for normal key remapping.
- **Normal key mapping (key_type=1)** with **Modifier + HID Usage pair** — confirmed working.
- **Flash commit `[0xAA, 0xAA]`** — takes effect immediately and persists.

**Not yet verified (from the spec only)**:

- Layer switching (`[0xA1, layer]`)
- Mouse mapping (key_type=2)
- Multimedia mapping (key_type=3)
- LED mapping (key_type=8, flash commit `[0xAA, 0xA1]`)

---

## Implementation Reference

Minimal pseudocode for a single remap:

```
// Remap packet: KEY1 → 'A'
buf = zeros(65)
buf[0]  = 3        // Report ID
buf[1]  = 0xFE     // opcode
buf[2]  = 1        // key number
buf[3]  = 1        // layer
buf[4]  = 1        // KeyType (normal)
buf[10] = 1        // combo count
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
