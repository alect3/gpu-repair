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
