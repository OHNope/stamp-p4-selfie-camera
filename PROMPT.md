# Stamp-P4 camera recorder — main board (kicad-auto demo)

The prompts below are the whole human input for this board. They are reconstructed from the
2026-10-08/09 run in `stamp-p4-cam-recorder/main-board`: the requirements are the ones recorded in
that run's `project_spec.json`, and the PCB-stage answers are the decisions the user gave in chat.
Two changes fold later revisions into the first pass: the M3 mounting holes are in the schematic
from the start, and every PCB-stage answer is given up front instead of when asked.

Vendor documents are in `sources/`; the JLCPCB stackup and manufacturing floors are in
`pcb-fabrication-profile.json`. Nothing else from the earlier run is an input.

## 1. Schematic — `/sch-generation`

> mode: new. Project `stamp-p4-cam-main` in this repository root. Run the full workflow through
> immutable publish, then `/sch-review`. Commit locally after each phase; never push.
>
> CAN-triggered camera recorder main board built around the M5Stack Stamp-P4.
>
> - **Controller**: M5Stack Stamp-P4 (S013, ESP32-P4NRW32, 16 MB flash / 32 MB PSRAM), SMD
>   castellated module soldered to the board. Author a project symbol from the vendor STAMP1 pin
>   table (65 pads, numbers = official footprint pads); footprint from `sources/Stamp-P4.kicad_mod`.
> - **No camera here**: MIPI-CSI is only on the module's top-side J2 BTB, used by a separate camera
>   adapter. LPG11 (pad 4) and LPG9 (pad 6) are the camera SDA/SCL shared with that BTB and stay NC.
> - **Power input** JAE IL-G-4P-S3L2-SA: 1 +5V, 2 GND, 3 external +3V3, 4 GND. +5V feeds the module
>   VIN (pad 14) and the CAN transceiver VDD.
> - **Logic rail**: the module's SOC_3.3V output (pad 28, SY8089, 600 mA budget) is +3V3.
> - **External 3.3 V** is a separate net +3V3_EXT, never bridged to +3V3; it feeds only the microSD
>   load switch. PWR_FLAGs on the externally supplied rails (+5V, +3V3_EXT, GND).
> - **microSD** Hirose DM3D-SF (push-pull, no eject spring), SDMMC 4-bit on slot 1 via the GPIO
>   matrix: G16 CLK with 22 Ω series, G17 CMD, G18–G21 D0–D3, G22 card detect with 10 k pull-up to
>   +3V3, G23 SD power enable. 10 k pull-ups on CMD/D0–D3 to the switched SD rail.
> - **SD power**: +3V3_EXT through high-side load switch Torex XC8107AC20MR-G (SOT-25: 1 VOUT,
>   2 VSS, 3 FLG, 4 CE active-high, 5 VIN) with a CE pull-down.
> - **CAN** to the logger/release board: JAE IL-G-3P-S3L2-SA (1 CAN_H, 2 CAN_L, 3 GND),
>   MCP2562-E-SN with VDD +5V, VIO +3V3, STBY low, Nexperia PESD2CAN24T-QR ESD. 120 Ω termination
>   selectable by a solder jumper.
> - **Buttons**: RESET on CHIP_EN (pad 34; module already has 10 k + 1 µF), BOOT on G35 (pads
>   36/48; module already has a 10 k pull-up).
> - **UART0 test points**: G37 U0TX (pad 30), G38 U0RX (pad 32), plus GND.
> - **One status LED** on a free module GPIO.
> - **Mounting**: four M3 mounting holes H1–H4 (MountingHole_3.2mm_M3, no net).
> - **Parts**: consult the component preference tables for every part; passives from the JLCPCB
>   basic seed where a seed value exists.
> - **Mechanical intent**: the power, CAN and microSD connectors open toward the nearest board edge,
>   cable/card inserted from outside. The module's own USB-C must stay reachable at a board edge.
>   The camera adapter overhangs the module's RMII/DSI edge, so parts near that edge stay low
>   (~1.5 mm).

## 2. Manual step — Update PCB from Schematic

After `sch-review` allows sch2pcb: open the project in KiCad, run *Tools → Update PCB from
Schematic*, save the board. This is the one GUI step.

## 3. PCB — `/pcb-generation`

> Board decisions:
>
> 1. Outline 60 × 45 mm rectangle, 1 mm corner radius.
> 2. H1–H4 near the four corners.
> 3. Fabrication profile JLCPCB 4-layer JLC04161H-7628, 1.6 mm, as given in
>    `pcb-fabrication-profile.json`.
> 4. CAN_H/CAN_L (MCP2562, 1 Mbps): coupled differential pair, equal length, default width/gap,
>    no impedance control.
> 5. The DM3D-SF no-conductive-trace area (catalog p.9) is an F.Cu keep-out; the KiCad footprint
>    already carries both areas as rule areas.
> 6. Camera adapter overhang beyond the module's RMII/DSI edge (~7.5 mm deep, ~24 mm wide, ~1.5 mm
>    above the board): keep a 26 × 8.5 mm region outside that edge free of components; tracks and
>    pours are allowed.
> 7. The module USB-C stays reachable at a board edge.
>
> Place, route, pour, review with `/pcb-review`, and publish the reviewed candidate to the active
> board with its project rules. Commit locally; never push.

## 4. Fabrication output

> Export Gerber + drill files for JLCPCB from the published board.
