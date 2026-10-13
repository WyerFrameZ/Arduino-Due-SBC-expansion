# Arduino-Due-SBC-expansion
# SBC Shield for Arduino Due

**Turn an Arduino Due into a tiny retro computer.** Plug in a VGA monitor or an old TV, a PS/2 keyboard and headphones.

<p align="center"><img src="docs/turntable.gif" alt="SBC Shield for Arduino Due – 3D turntable" width="720"></p>

- **Fully passive:** no extra microcontroller, just resistors, two transistors and connectors.
- **Through-hole parts only:** easy to solder by hand.
- **No jumpers or switches:** plug it onto the Due and go.
- **Unused Due headers stay reachable:** three cut-outs in the board edge leave D8–D13, SDA1/SCL1, AREF, the serial ports TX1–3/RX1–3 with SDA/SCL (D14–D21) and A0–A7 free for jumper wires.
- **Arduino Mega / Due form factor** (101.6 × 53.3 mm), KiCad 10 project, Gerbers ready for JLCPCB.

> **Status:** rev 1 – untested prototype. Build one and check every port before ordering more.
>
> ⚠️ **Arduino Due only.** The resistor values and pins match the Due's 3.3 V I/O and the DueVGA library. On other boards the outputs won't work.

---

## Features

| Port | Mode | Resolution | Colours | Library |
|---|---|---|---|---|
| **VGA** | colour | 320×240 | 256 (RRRGGGBB) | [DueVGA](https://github.com/stimmer/DueVGA) |
| **VGA** | monochrome | up to 800×600 | black & white | DueVGA |
| **VIDEO** (RCA) | PAL composite | 320×240 | 256 | DueVGA |
| **VIDEO** (RCA) | NTSC composite | 320×200 | 256 | DueVGA |
| **PS/2** | keyboard | – | – | [PS2Keyboard](https://github.com/PaulStoffregen/PS2Keyboard) |
| **PHONES** (3.5 mm) | stereo line out | 12-bit DAC | – | `analogWrite(DAC0/DAC1, …)` |
| **Free Due pins** | edge cut-outs | – | – | D8–D21, SDA/SCL, A0–A7 |

VGA and VIDEO share pins 36–41. DueVGA drives one output at a time; you choose VGA or composite in software.

## Board

![SBC Shield – 3D view](docs/render_3d.png)

| Ports: VGA, PS/2, VIDEO, PHONES | Back side |
|---|---|
| ![Ports](docs/render_ports.png) | ![Back](docs/render_3d_back.png) |

| Top | Bottom |
|---|---|
| ![Top](docs/render_top.png) | ![Bottom](docs/render_bottom.png) |

Schematic: [docs/schematic.pdf](docs/schematic.pdf)

## Pins used

| Due pin | Function |
|---|---|
| 34–41 | VGA colour (R 39–41, G 36–38, B 34–35) |
| 36–41 | composite video DAC (same pins, separate resistor ladder) |
| 42 / 43 | VGA VSYNC / HSYNC |
| SPI header MOSI (via the 2x3 socket J15 on the bottom) | monochrome VGA |
| 3 / 4 | PS/2 CLK / DATA |
| DAC0 / DAC1 | audio left / right |
| 5V, GND, IOREF | power, keyboard level reference |

## How it works

| Block | Parts | Function |
|---|---|---|
| **VGA** | R1–R10 | Resistor DAC with the DueVGA wiring: 2k2 / 1k / 470 Ω on red and green, 820 / 390 Ω on blue, into the monitor's 75 Ω inputs. 82 Ω series resistors on HSYNC / VSYNC. |
| **MONO** | R11–R13, J15 | A 2x3 socket on the bottom plugs onto the Due's 6-pin SPI header and picks up MOSI and GND. MOSI drives R, G and B together through 3 × 300 Ω (= 100 Ω, as DueVGA specifies). In colour mode they stay separate. |
| **VIDEO** | R14–R19 | Composite DAC with the DueVGA wiring: pins 36–41 through 3k3, 1k6, 820, 390, 200 and 100 Ω into the TV's 75 Ω input. |
| **KEYBOARD** | Q1, Q2, R31–R34 | PS/2 is a 5 V, open-collector, two-way bus, and the Due's pins are 3.3 V only. Each line (CLK and DATA) gets a bidirectional 2N7000 level shifter: 10 kΩ pull-up to the Due's IOREF on one side, 4.7 kΩ to 5 V on the keyboard side. |
| **AUDIO** | R21, R24, C3, C4, R27, R28 | Each DAC output goes through 1 kΩ (protects the DAC) and 10 µF (removes the DC offset), with a 47 kΩ load to ground. |

## Quick start

```cpp
#include <VGA.h>            // DueVGA
#include <PS2Keyboard.h>

PS2Keyboard kbd;

void setup() {
  VGA.begin(320, 240, VGA_COLOUR);   // or VGA.begin(800, 600) for mono, or PAL / NTSC
  kbd.begin(4, 3);                   // DATA = 4, CLK/IRQ = 3
}

void loop() {
  if (kbd.available()) VGA.print((char)kbd.read());
}
```

Sound:

```cpp
analogWriteResolution(12);
analogWrite(DAC0, 2048);             // left channel, 0..4095
analogWrite(DAC1, 2048);             // right channel
```

## Building it

1. Order the PCB: upload `SBC_Carrier_JLCPCB.zip` to JLCPCB, 1.6 mm, blue solder mask, white silkscreen, 5 pcs.
2. Get the parts from the list in [MANUAL.md](MANUAL.md).
3. Solder the resistors first, then the capacitors and transistors, then the connectors.
4. Plug the **bottom headers** onto the Due:
   - the male headers J1, J4, J5 and J7;
   - the 2x3 socket J15 on the SPI header.

   Put the shield on top and solder them, so that everything lines up.
5. Trim the leads under the VGA and PS/2 connectors and put a piece of Kapton tape on the Due's USB and DC jack under them.

### Notes
- **Headphones:** the DAC output is line level. High-impedance headphones and amplified speakers are fine; low-impedance earbuds will be quiet.
- **Hot-plugging:** don't hot-plug the PS/2 keyboard.

## Repository layout

| Path | Content |
|---|---|
| `SBC_Carrier.kicad_*` | KiCad 10 project (schematic + PCB) |
| `lib/Carrier_Parts.pretty` | custom footprints (VGA, RCA, PS/2, partial Mega headers, Due SPI socket) |
| `lib/3d` | 3D models of the RCA and PS/2 connectors (referenced via `${KIPRJMOD}`) |
| `SBC_Carrier_JLCPCB.zip` | Gerbers + drill files for JLCPCB |
| `docs/` | renders and schematic PDF |
| `MANUAL.md` | parts list, build order, first power-up |

## Roadmap

- [ ] Build and test rev 1 (VGA colour, VGA mono, composite, keyboard, audio)
- [ ] Demo firmware: text terminal on VGA with keyboard input
- [ ] Small BASIC interpreter for the full "computer at power-on" experience
- [ ] Photos of the assembled board and screenshots from a real monitor

## Credits

- [DueVGA](https://github.com/stimmer/DueVGA) by stimmer – the VGA / composite resistor networks follow its wiring.
- [PS2Keyboard](https://github.com/PaulStoffregen/PS2Keyboard) by Paul Stoffregen.
- Designed by **WD** in Poland.

## License

TBD

