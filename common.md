# Common findings

Issues observed on more than one board.

## Missing components (both boards)

On both boards, two capacitors and a resistor array had fallen off. Same
location on each board, which suggests a manufacturing or handling weakness
rather than a one-off. The caps were re-soldered and the array was replaced
with two discrete 1.5 kohm resistors, on both boards, on 2026-09-05 before
any testing began.

Location: front of the board, bottom right, beside a small inductor. The
regulator IC for that inductor is on the back, directly behind. On a
similar layout (Inno3D 3090, GPU Solutions video "The Mysterious Short
Circuit Fix", https://www.youtube.com/watch?v=cXy12uUIkgk) that inductor is
the 5V rail. Different vendor, so use as a guide, not a map.

Both boards have a 5V fault at this spot:

- Board B: 5V shorted to ground. Re-fitted resistors read 1.5 kohm and
  0 ohm in place.
- Board A: 5V at 0V, regulator IC on the back gets warm when powered.
  Re-fitted resistors read 1.5 kohm and 1.0 kohm in place.

Both re-fitted capacitors read 0 ohm in place on both boards. So the 5V
net is shorted to ground on both cards, and in-place readings of any part
on that net are meaningless until it is lifted.

Both fitted resistors are 1.5 kohm. The 1.0 kohm in-place reading on Board
A is a 1.5 kohm in parallel with something else on the board, which is
normal. Board B reading 0 ohm at that same position is the only reading
that differs between the boards.

The original part was an array, so the two discretes had to be placed to
match the array's internal wiring. A mismatch (one resistor bridging two
pads that the array kept separate) would be a plausible way to short that
node on one board and not the other.
Whatever knocked them off hit the same regulator on both cards. The short
is either a cracked cap (one per board would be enough) or the regulator
IC itself. Board A's IC runs warm, which favours the IC there.

Known values so far:

| Part       | Value    | Notes                                        |
|------------|----------|----------------------------------------------|
| Resistor array | 2 x 1.5 kohm | Original part. Replaced with two discrete 1.5 kohm on both boards |
| Discrete R1 | 1.5 kohm | Reads 1.5 kohm in place on both boards      |
| Discrete R2 | 1.5 kohm | Reads 1.0 kohm in place on A (parallel path), 0 ohm on B |
| Cap 1      | ?        | Reads 0 ohm in place on both (net shorted)   |
| Cap 2      | ?        | Reads 0 ohm in place on both (net shorted)   |

TODO:
- Identify the reference designators of the missing parts.
- Photograph the area on both boards.
- Determine values (compare with the other board if any are still attached,
  or find a board view / schematic).
- Work out which rail or circuit they belong to. Current suspicion is that
  they are on the 5V circuit, since Board B has a 5V short.

## Slot 12V to core VRM reads about 50 kohm (both boards)

Unpowered, there is no continuity from the PCIe slot 12V fingers to the
core VRM high-side FET drains. Both boards read about 50 kohm, so this is
the board design, not a fault. Most likely the slot 12V feeds a separate
group of phases or goes through its own switch. Do not chase this.

The 8-pin 12V pins do beep through to the core VRM on both boards.

## Powered signature (2026-09-05)

Same setup for both: card on a powered riser, ATX PSU on the 8-pins, no PC.

| Point                                | Board A     | Board B        |
|--------------------------------------|-------------|----------------|
| 12V at both 8-pins                   | 12V         | 12V            |
| Bottom right inductor (5V rail)      | about 30 mV | about 35 mV    |
| Bottom left inductor (assumed 1.8V)  | about 300 mV| about 300 mV   |
| Top left GS9216 inductor             | 0V          | 0V             |
| 5V regulator IC on the back          | warm after a long run | barely, short run |

Both boards fail the same way: 12V arrives, 5V never comes up, and every
rail downstream stays dead. The 300 mV on the 1.8V inductor is identical,
so it is leakage into a rail whose regulator has no input, not a fault of
its own.

There is no real difference. Board A's "warm IC" was after a long run and
its 5V rail also sits at about 30 mV. Both boards have a hard short on the
5V net and a regulator sitting in current limit. Identical fault.

## Powered signature after the 5V rework (2026-09-09, evening)

Setup: card in x299, slot-6 riser root port B, host booted. A known-good
Gigabyte 3090 links x4 in the same position, so slot power and PERST#
release are proven. The BIOS hides a root port whose link never trains,
so a Zotac board here simply does not appear in lspci: no link at all.

| Point                              | Reading                       |
|------------------------------------|-------------------------------|
| 12V at the 8-pins                  | 12V                           |
| 5V, bottom right inductor          | 5V (was 30 mV before rework)  |
| 3.3V, small cap behind the fingers | 3.08V                         |
| 1.8V, bottom left inductor         | 1.8V (was 300 mV)             |
| Top left GS9216 inductor           | 0V, both boards               |
| Core caps behind the die           | 0V                            |
| Memory caps                        | 0V                            |
| GS9216 output to ground, unpowered | about 5 ohm (not a short)     |

One board was first measured with a front waterblock fitted: the 1.8V
inductor read 300 mV until the block came off, then 1.8V. Whether the
block was shorting the 1.8V net or the rail simply had not started on that
power-up was not separated. Both boards were then tried bare in the same
slot with a fan on the die: same readings, neither links.

So the 5V fix moved the chain one stage: 5V and 1.8V now run. The chain
stops at the top left GS9216, whose rail is still unidentified but, with
1.8V confirmed at the bottom left, is most likely PEX. A dead PEX rail is
by itself enough for no link and for core/memory staying off. Identical on
both boards, which points at a common enable gate rather than two dead
chips. Next: the GS9216 pin sequence in notes/gs9216.md (AIN, VCC, EN,
output) and a continuity trace of where EN comes from.

## GS9216 EN network (2026-09-09, evening, board in x299)

GS9216 (top left, front) powered: pin 7 AIN 12V, pin 21 VCC 5V, pin 2 EN
1.5V, pin 3 PFM 1.3V, output 0V. The chip is alive; EN sits just under the
1.6V rising threshold, so it never starts. That is the whole remaining
fault on this board.

Continuity from pin 2, unpowered: direct beep to ONE END of ONE of the two
discrete 1.5 kohm resistors fitted in place of the array, the resistor
nearest the top edge of the board. No beep to the 3.3V cap, 5V inductor,
12V pins, or pin 1 PGOOD. Resistance from pin 2: 10k to the 1.8V
inductor (lowest, so EN's pull-up is about 10k from the 1.8V rail, i.e.
PEX is sequenced after 1.8V by design), 11k to ground, 12.6k to 5V, 15k
to the 3.3V cap, 17k to pin 3.

So EN should sit at 1.8V and is being loaded down by 0.3V, and the only
direct connection to it is the fitted 1.5k. Working theory: the discrete
resistors were placed across pads the original array kept separate, so
one of them now loads EN. Same replacement on both boards explains the
identical 0V on both GS9216s.

Next: powered volts at both ends of both discretes; continuity from the
far end of the EN resistor; lift the EN end of that resistor and re-read
EN (expect 1.8, then the GS9216 inductor should come up). Fallback: force
EN with 10k from pin 21 to pin 2. The 5V regulator IC (back, behind the
bottom right inductor) marking is still unrecorded; if the EN resistor's
far end lands on one of its pins, the array was wiring EN into the 5V
power-good.

## The chip behind the bottom right inductor is the MCU, not a regulator (2026-09-09)

Marking HT32F52241: Holtek Cortex-M0+ microcontroller, 2.0 to 3.6V supply,
sold by GPU repair shops as a graphics-card housekeeping MCU. On this board
it is the sequencer (and Spectra RGB). Earlier notes calling it "the 5V
regulator IC" are wrong; the 5V buck's controller is elsewhere.

Powered readings tonight, board in x299:

- MCU decoupling caps (two largish caps beside it, back): 0V. THE MCU HAS
  NO SUPPLY. This is the remaining fault: no MCU, no PEX enable, no chain.
- The fitted 1.5k nearest the top edge: one end 5V, other end 2.7V. Its
  5V-side... correction: the end that beeps is GS9216 PIN 1 (PGOOD), not
  pin 2. So the array was the 5V pull-ups for power-good lines into the
  MCU, and the rework is NOT in the GS9216 EN path. EN's 1.5V is the
  signature of a pin driven from an unpowered MCU.
- The other fitted 1.5k: 1V on the edge side, 0V (or no reading) on the
  other.

Next: MCU supply net resistance to ground (kohm = healthy, ohms = MCU
shorted from the 5V-short episode, when it ran warm); continuity from the
MCU caps to the two re-fitted caps, 5V inductor, slot 3.3V; find the 3.3V
LDO feeding the MCU and read its input (expect 5V) and output (expect
3.3V). If the MCU itself is dead, a replacement needs Zotac's firmware:
check Board B's MCU supply net and whether it is alive before anything.

## U15 is a missing IC, not a resistor array (2026-09-09, late)

Pre-rework photo shows the boxed silkscreen region U15 beside the mounting
hole: an empty cap footprint (top), the brown cap (middle), and at the
bottom a small dark 4-pad body, which is U15 itself. Footprint is 2 + 2
pads with a centre ground pad: DFN-4 with exposed pad. The "resistor
array" was this IC. Both boards lost it (and the neighbour cap) to the
same standoff at that mounting hole; the original parts are gone on both.

Top right pad is 5V (input). Two discrete 1.5k resistors are currently on
its pads on both boards and must come off. Working theory: U15 is the
3.3V LDO for the HT32F52241 MCU (IN 5V, OUT to the MCU supply cap, EN,
GND + centre). The MCU has 0V on its caps, which is the remaining fault.

Next (office): lift both resistors on both boards; map the three unknown
U15 pads against ground, the MCU supply cap and both brown caps; MCU
supply cap to ground resistance (kohm = alive, ohm = shorted); then a
temporary jumper from the slot 3.3V cap to the MCU supply cap, card in
x299, and read MCU caps / GS9216 pin 2 / GS9216 inductor / core caps.
Fix: any 3.3V LDO (SOT-23-5 dead-bugged with three wires, or a 1x1 DFN-4
matching the pad map), one per board.

## U15 pad map (2026-09-09, night). U15 is NOT the MCU's regulator.

Corrections: the MCU (HT32F52241) IS powered, 3.1V on its own supply cap
(slot 3.3V, which reads 3.08 behind the fingers). The two brown caps at
U15 are on U15's INPUT net and read 5V powered; the earlier "MCU caps 0V"
was those caps probed on the ground end. The MCU supply cap is a smaller
cap beside the MCU and does not connect to any U15 pad.

U15 footprint: 2 + 2 pads with a centre ground pad (DFN-4 with EP),
"top" = PCIe-slot end of the card:

| Pad          | Reading                              | Role         |
|--------------|--------------------------------------|--------------|
| top right    | 5V, both brown caps on it            | IN           |
| bottom right | 2.9V powered; 470k to the 3.1 net,   | EN, weak     |
|              | nothing else (2.9 = 3.1 via 470k     | pull-up      |
|              | loaded by the meter)                 |              |
| top left     | 0V                                   | OUT (dead)   |
| bottom left  | ground                               | GND          |
| centre       | ground                               | GND / EP     |

So U15 was a small regulator or load switch from 5V, enabled by default,
whose output rail is now dead. Whatever sits on that rail is what the
sequencing is waiting on; the MCU holds GS9216 EN at the 1.5V divider
level. Next: from the top-left pad, resistance to ground (expect a real
load) and continuity to GS9216 pins 2/3 and its caps, the 3.08 cap, the
1.8V inductor, MCU pins, BIOS EEPROM VCC. Test: jumper the 3.1 net onto
the top-left pad in-slot and watch GS9216 EN / inductor. Fix if it works:
3.3V LDO, IN top-right, OUT top-left, GND centre, EN bottom-right; same
on Board B. Resistor on 5V that beeped to GS9216 pin 1: pin 1 is on the
5V net directly, so the GS9238-derived pinout is wrong for pin 1 at least.

## U15 identified functionally: a 5V supervisor driving GS9216 EN (2026-09-09, night)

Top-left (OUT) pad: 20k to ground, and a brief continuity beep to GS9216
pin 2 (EN). So U15's output drives the PEX enable node directly, and
bottom-right (470k pull-up to 3.3V) is its manual-reset / enable input.
U15 = voltage supervisor on the 5V rail with a push-pull output that goes
high when 5V is good. With it missing EN sits at the passive 1.5V divider
level and PEX never starts. Same on both boards. The MCU is not in this
path (powered, 3.1V, idle).

Original part number unknown (no boardview, parts lost on both boards).
Replacements, one per board:
- Proper: TI TPS3840PL46 (or PL42), push-pull active-low reset, 4-pin.
  VDD -> top-right (5V), RESET -> top-left (EN node), MR -> bottom-right
  (470k pull-up), GND -> centre. Check the silkscreen outline: 1 mm square
  = X2SON-4 drops on (match pin order first); 2 mm square = DFN 2x2, so
  dead-bug the SOT-23-5 version with wires.
- Equivalent: one 10k 0603 between top-right and top-left. EN follows the
  5V rail via the 10k/20k divider (crosses 1.6V when 5V reaches ~2.4V).
Confirmation test either way: GS9216 pin 2 > 1.6V, inductor ~1.0-1.1V,
core caps behind the die come up, card appears in lspci.
