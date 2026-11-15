# KMBox Bridge - Advanced Autopilot Firmware

> **📖 For overview, architecture, and quick setup, see the [main README](../README.md#kmbox-bridge---advanced-autopilot).**

Advanced autopilot firmware for **Adafruit Feather RP2350** with color-based target tracking, PIO UART communication, and LED status indicators.

## Detailed Wiring

| Feather RP2350 Pin | Direction | KMBox Pico Pin | Description |
|--------------------|-----------|----------------|-------------|
| GPIO0 (TX)         | ────►     | RX pin         | Bridge transmits commands |
| GND                | ────      | GND            | Common ground |

**Note:** One-way communication (bridge → KMBox). No RX connection needed on bridge.

## Protocol Details

### PC → Bridge (USB CDC)
- Frame header: `['F', 'R', width_low, width_high, height_low, height_high, 0x00, 0x00]`
- Frame data: RGB888 pixels (width × height × 3 bytes)
- Config commands: `['C', r_min, r_max, g_max, b_max, gain_x*10, gain_y*10, deadzone, min_blob]`

### Bridge → KMBox (PIO UART @ 115200 baud)
- Mouse move: `[0x01, dx_low, dx_high, dy_low, dy_high, buttons]`

## Python Client Usage

Install pyserial:
```bash
pip install pyserial
```

### Command Line
```bash
python bridge_client.py --list    # List available ports
python bridge_client.py --test    # Test connection  
python bridge_client.py           # Interactive mode
```

### As Library
```python
from bridge_client import KMBoxBridge

with KMBoxBridge() as kmbox:
    kmbox.move(100, 50)      # Relative mouse movement
    kmbox.click('left')       # Left click
    kmbox.key_press('a')      # Press 'a' key
```

### Direct Serial Access
The bridge provides transparent KMBox passthrough:

```bash
# macOS: screen /dev/cu.usbmodem* 115200
# Linux: screen /dev/ttyACM0 115200
```

Then type KMBox commands directly:
```
km.move(10,20)
km.click(1)
```

## Keyboard injection (`km.press`, `km.type`, `km.release`, `km.keys`)

The bridge can drive the **keyboard** of the target PC through the KMBox (Board 1), next to the mouse. Keycodes are USB
HID usage IDs (decimal or `0x` hex, e.g. `4` = `a`, `0x28` = Enter); modifiers are the HID modifier bitmask
(`0x01` LCtrl, `0x02` LShift, `0x04` LAlt, `0x08` LGUI/Win, `0x10`-`0x80` the right-hand versions).

| Command | Effect |
|---|---|
| `km.press(keycode[,modifiers])` | tap one key (press + release), e.g. `km.press(4,2)` types `A` |
| `km.type("text")` | type US-layout ASCII, one tap per character (12 ms apart) |
| `km.release()` | release every key (use it if a key ever gets stuck) |
| `km.keys(mods[,k1..k5])` | set the **complete** keyboard state in one report: modifier bitmask plus up to 5 held keys. The caller owns every key-down and key-up time, so human-like timing, rollover and long holds are possible. `km.keys(0)` releases everything |
| `km.human(0\|1\|2)` | KMBox humanization of mouse moves: 0 = off (raw), 1 = micro, 2 = full (the default after every boot of Board 1) |
| `km.version()` | `kmbox bridge v1.2 keys human failsafe` |

Notes
- Arguments are parsed strictly (decimal digits or `0x` hex, 0..255, no empty arguments, the `)` must end the line);
  malformed `km.press` / `km.keys` / `km.human` lines are answered with `ERR: ...` instead of being forwarded.
- Keyboard and mouse share **one** USB HID endpoint on Board 1. A report that arrives while it is busy is dropped,
  so verify delivery on the target (e.g. `GetAsyncKeyState`) and re-send; leave >= 2 ms between consecutive packets.
- Failsafe: when the host closes the serial port (or the controlling program is killed) the bridge sends an empty
  keyboard state (three times) and releases all mouse buttons, so a hard kill can no longer leave a key stuck down.
- Example (Python, pyserial): `ser.write(b"km.keys(2,4)\r\n"); time.sleep(0.09); ser.write(b"km.keys(0)\r\n")`
  holds LShift+`a` for 90 ms.

## Performance & Troubleshooting

**Performance:**
- Baud rate: 115200 bps (~200 KB/s)
- Latency: <1ms typical through bridge
- Throughput: ~11,500 commands/second theoretical

**Common Issues:**
- Bridge not detected: Check USB cable, try different port, re-flash firmware
- No KMBox response: Verify wiring (TX→RX, GND→GND) and matching baud rates
- Garbled data: Check GND connection, try shorter wires

## USB White-Labelling (OTP Branding)

The bridge RP2350 can be **permanently** branded so that BOOTSEL mode shows custom
identifiers instead of the default "Raspberry Pi" / "RP2350" / "RPI-RP2".

> ⚠️ **OTP writes are irreversible.** Once burned, the branding cannot be changed.

### What Changes

| Field | Default | White-Labeled |
|-------|---------|---------------|
| USB Manufacturer | Raspberry Pi | Hurricane |
| USB Product | RP2350 Boot | KMBox Bridge |
| Volume Label | RPI-RP2 | KMBOX-BRDG |
| UF2 Model | Raspberry Pi RP2350 | Hurricane KMBox Bridge |
| Board ID | RP2350 | KMBOX-BRIDGE |
| SCSI Vendor | RPI | HURRCNE |
| INDEX.HTM | raspberrypi.com | GitHub repo |

> **Note:** This only affects BOOTSEL mode. The running firmware's USB descriptors
> (CDC device name, etc.) are set at compile time in `CMakeLists.txt` and are already
> branded as "Hurricane".

### Usage

```bash
# From the repo root:
./build.sh white-label          # Interactive — prompts before writing
./build.sh white-label-verify   # Read back current OTP state

# Or directly:
cd bridge/
./white_label.sh                # Interactive
./white_label.sh --dry-run      # Preview without writing
./white_label.sh --verify       # Read current state
./white_label.sh --force        # No prompt (CI/automation)
```

### Custom Branding

Edit `bridge/white_label.json` to change any field before burning. See the
[RP2350 Datasheet §5.7](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf)
for full details on available fields and limits.
