# Connecting an ST-Link to the Raz DC25000 via USB-C

No soldering required. The SWD debug signals (SWDIO and SWCLK) are routed
directly to the **CC1 and CC2 pins** of the USB-C charging port. A cheap
USB-C breakout board exposes those pins as labeled header pins.

This trick works on the entire Raz/Kraze color-LCD family — it was first
documented by [ginbot86](https://github.com/ginbot86/ColorLCDVape-RE) and
[xbenkozx](https://github.com/xbenkozx/RAZ-RE), and popularized by the
[Rip It Apart blog post](https://ripitapart.com/2024/04/20/dispo-adventures-episode-1-reverse-engineering-and-running-windows-95-on-a-disposable-vape-with-a-colour-lcd-screen/).

---

## Why the CC Pins?

A standard USB-C device places 5.1 kΩ pull-down resistors on CC1 and CC2 so
chargers recognize it. The vape manufacturer reused those same two pins as the
SWD port — no extra test pads, no opening the case. The 5.1 kΩ resistors are
already on the board and don't interfere with SWD at the 1 MHz adapter speed
used by OpenOCD.

---

## What You Need

| Item | Where to get it |
|---|---|
| ST-Link V2 clone | Amazon / AliExpress (~$8) |
| USB-C breakout board | Amazon / AliExpress / Adafruit (~$5) — any board that breaks CC1 and CC2 out to header pins |
| 3× jumper wires (female–female) | Included with most ST-Link clones |

> **Tip:** Make sure the breakout board labels CC1 and CC2 separately. Boards
> that only break out VBUS, GND, D+, D− won't work.

---

## USB-C Pinout (relevant pins only)

```
USB-C receptacle, looking into the port on the vape:

        ┌─────────────────────────┐
        │  GND  CC1  CC2  GND    │
        │   ↑    ↑    ↑    ↑     │
        │  A12  A5   B5  B12     │
        └─────────────────────────┘

  VBUS and D+/D− are present but unused for debugging.
  Do NOT connect VBUS — the vape is self-powered by its LiPo.
```

---

## Wiring

Connect 3 jumper wires between the USB-C breakout board and the ST-Link:

| USB-C breakout pin | ST-Link pin | Signal |
|---|---|---|
| CC1 | SWDIO | SWDIO (PA13 on MCU) |
| CC2 | SWCLK | SWCLK (PA14 on MCU) |
| GND | GND | Ground |

> **CC1/CC2 assignment varies by device.** If OpenOCD fails to connect,
> swap the CC1 and CC2 jumper wires (try CC1→SWCLK, CC2→SWDIO) and retry.
> It takes about 5 seconds to test both orientations.

> **Leave all other ST-Link pins disconnected.** In particular, do NOT
> connect the ST-Link's 3.3 V output — the vape's LDO is already running
> and back-feeding 3.3 V into the ST-Link's power pin can damage both.

---

## Step-by-Step

1. Plug the USB-C breakout board into the vape's charging port.
2. Connect the 3 jumper wires as shown in the table above.
3. Plug the ST-Link into your PC's USB port.
4. Turn the vape on (press the button) — the LiPo must be powering the MCU.
5. In PowerShell (run as Administrator):
   ```powershell
   usbipd attach --wsl --busid 1-2
   ```
   Substitute the bus ID shown by `usbipd list` for your ST-Link.
6. From any example directory, run:
   ```bat
   flash_vape.bat   # or flash_slots.bat
   ```
   OpenOCD will connect at 1 MHz, halt the core, flash, and reboot.

---

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `Error: Could not find MEM-AP` | Vape is off or battery too low — press the button to wake it; or CC1/CC2 are swapped — try reversing them |
| `WAIT` or timeout during erase | ST-Link 3.3 V accidentally connected; disconnect it |
| Connects once, then fails | Vape entered sleep mode mid-flash; increase IWDG timeout in TCL or keep the button held |
| `PGERR` after write | Flash still locked — rerun `gen_direct_flash.py` and reflash |

---

## References

- [ginbot86/ColorLCDVape-RE](https://github.com/ginbot86/ColorLCDVape-RE) — original reverse-engineering of the Kraze HD7K / Raz TN9000 family; first to document SWD-on-CC-pins
- [xbenkozx/RAZ-RE](https://github.com/xbenkozx/RAZ-RE) — RAZ-specific RE work, flash map, and tooling
- [Rip It Apart — Dispo Adventures Ep. 1](https://ripitapart.com/2024/04/20/dispo-adventures-episode-1-reverse-engineering-and-running-windows-95-on-a-disposable-vape-with-a-colour-lcd-screen/) — blog writeup with photos of the custom debug cable
- [USB-C standard CC pin reference](https://www.allaboutcircuits.com/technical-articles/introduction-to-usb-type-c-which-pins-power-delivery-data-transfer/) — USB-C pin numbering background
