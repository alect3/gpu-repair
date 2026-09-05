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
