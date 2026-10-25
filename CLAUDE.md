# HSWET Variable Load Board

KiCad project `variable load.kicad_pro`. Two schematic sheets, the root sheet and `dac.kicad_sch`. This file guides placement and routing.

## What the board does

The board is an active electronic load for the Collegiate Wind Competition. It sits in the load box on the far side of the PCC from the turbine. Ten MOSFET channels each sink a current set by `VREF`. Each channel is an op amp loop that holds the voltage across its 0.05 ohm sense resistor equal to `VREF`, so each channel draws `VREF / 0.05`.

`VREF` comes from an MCP4822 DAC, divided by 8 and buffered. The DAC gets its data from the turbine control box over SPI through three 6N137 optocouplers. A 3.48M resistor from `VTURB` into the divider gives a minimum load of about 20 ohm when the control box is off.

Expected turbine power is about 120W, roughly 6A at 20V. The channels are sized for up to 5A each.

## Parts

Derive channel grouping from the netlist. Each MOSFET source net connects to exactly one 0.05 ohm resistor and one op amp −IN pin. Each gate net connects to one 100 ohm resistor, one 100k resistor and one 470pF capacitor.

| Refs | Part | Footprint |
|---|---|---|
| Q1 to Q10 | IXTX110N20L2 linear MOSFET, PLUS247 | `TO-247-3_Vertical` |
| R1, R2, R5, R6, R19, R20, R23, R24, R28, R30 | 0.05 ohm sense, 3W | `R_2512_6332Metric` |
| R7, R8, R11 to R14, R17, R18, R31, R33 | 100 ohm gate resistor | `R_0805_2012Metric` |
| R3, R4, R9, R10, R15, R16, R21, R22, R29, R32 | 100k gate pulldown | `R_0805_2012Metric` |
| C5, C6, C9 to C14, C18, C20 | 470pF C0G loop compensation | `C_0805_2012Metric` |
| IC1, IC2 | OPA4197, channels 1 to 8 | `SOIC-14_3.9x8.7mm_P1.27mm` |
| IC3 | OPA4197, channels 9 and 10, VREF buffer, one spare section | `SOIC-14_3.9x8.7mm_P1.27mm` |
| C7, C8, C19 | 1uF op amp supply decoupling | `C_0805_2012Metric` |
| U1 | R-78HE5.0-0.3, 12V to 5V | SIP3 |
| C1 | 10uF 25V, U1 input | `C_1206_3216Metric` |
| C2, C3, C4 | 100nF and 10uF on `5V_LOAD` | `C_0805_2012Metric` |
| U2, U3, U4 | 6N137 optocouplers | `DIP-8_W7.62mm` |
| C15, C16, C17 | 100nF opto decoupling | `C_0805_2012Metric` |
| R25, R26, R27 | 1k opto output pull ups | `R_0805_2012Metric` |
| U5 | MCP4822 DAC | MSOP-8 |
| R34 | 10k pulldown on `VREF_DAC` | `R_0805_2012Metric` |
| R37, R36 | 6.98k and 1k divider, 0.1% | `R_0805_2012Metric` |
| R35 | 3.48M low wind floor, will be swapped during tuning | `R_1206_3216Metric` |
| C21 | 10uF divider filter | `C_0805_2012Metric` |
| J1 | `VTURB` and GND in, Phoenix MKDS 3/2 | `TerminalBlock_Phoenix_MKDS-3-2-5.08_1x02_P5.08mm_Horizontal` |
| J2 | 12V and GND in | Same as J1 |
| J3 | Link from control box, `5V_LED`, `SCK_K`, `SDI_K`, `CS_K` | `JST_XH_B4B-XH-A_1x04_P2.50mm_Vertical` |

## Stackup

Four layers.

| Layer | Use |
|---|---|
| F.Cu | Parts and signal routing |
| In1.Cu | Solid GND plane over the whole board. No cuts or slots. |
| In2.Cu | `VTURB` zone under the power section only, GND fill everywhere else at lower priority |
| B.Cu | GND pour over the whole board |

Board outline about 220 × 130mm. Use 2 oz outer copper if the fab offers it.

## Placement

1. Put Q1 to Q10 in one row along the bottom board edge, metal back facing outward, about 20mm apart. One heatsink clips onto all of them. Keep the area beyond that edge free of parts.
2. Put each 0.05 ohm resistor directly behind its MOSFET source pin, as close as the pads allow.
3. Put each 100 ohm gate resistor and 100k pulldown right at its MOSFET gate pin.
4. Center each op amp over its channels. IC1 over its four, IC2 over its four, IC3 over channels 9 and 10 at the right end of the row.
5. Put each 470pF next to its op amp, not next to the MOSFET.
6. Put J1 at the top left corner. Put J2 and U1 in a narrow strip down the left edge below J1, leaving room for the `VTURB` path from J1 down to the MOSFET row.
7. Put the signal parts in the top right corner, as far from the MOSFETs as possible. That covers J3, U2 to U4, U5, R34, the divider R35, R36, R37 and C21. Put the divider next to IC3, since IC3 buffers it.
8. Put every decoupling capacitor next to its IC supply pin.

## Net classes

| Class | Nets | Track | Clearance |
|---|---|---|---|
| HighCurrent | `VTURB`, MOSFET source nets | Zones, or 3mm minimum where a track is unavoidable | 0.5mm |
| Supply | `12V`, `5V_LOAD`, `5V_LED` | 0.5mm | 0.2mm |
| Default | Everything else | 0.25mm | 0.2mm |

Vias 0.4mm drill and 0.8mm pad by default. Use arrays of at least six vias wherever current changes layers.

## Routing rules

1. `VTURB` runs as a zone on In2.Cu from J1 down the left side and across the MOSFET row. Stop it before the signal corner. Join each MOSFET drain to it through the drain pin plus extra vias in a top layer pour around the drain pad.
2. Each MOSFET source to its sense resistor is a short, wide top layer pour.
3. Each sense resistor ground pad drops straight into the GND planes through several vias at the pad.
4. Route each op amp −IN as its own thin track from the source side pad of its sense resistor. Do not tap the wide source pour partway along. This is a Kelvin connection and it matters for accuracy.
5. Route `VREF` as one track along the op amp row on F.Cu, with GND on both sides and no high current copper underneath on In2.Cu.
6. Keep gate tracks short and keep them away from the drain copper.
7. Add a row of GND stitching vias between In1.Cu and B.Cu along the MOSFET row, about 5mm apart.
8. Keep the SPI nets, `SCK_K`, `SDI_K`, `CS_K`, `SCK_L`, `SDI_L` and `CS_L`, inside the signal corner.

## Do not

- Do not split, slot or partition the GND planes. The solid plane replaces a star ground.
- Do not run `VTURB` copper under the signal corner on any layer.
- Do not route signal tracks between MOSFET pins or through the sense resistor area.
- Do not change footprints, values or net names without asking.
- Do not move parts off the MOSFET edge row or rotate the MOSFETs away from the heatsink edge.

## Done means

- DRC passes with zero errors and zero unconnected items.
- All zones refilled after the last edit.
- Every channel has the same layout pattern, so the ten loops behave alike.
- The board edge beyond the MOSFET row is clear for the heatsink.
