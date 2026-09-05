# Common findings

Issues observed on more than one board.

## Missing components (both boards)

On both boards, two capacitors and a resistor had fallen off. Same location
on each board, which suggests a manufacturing or handling weakness rather
than a one-off. Both sets were re-soldered on 2026-09-05 before any
testing began.

Board A reads clean on 5V with the parts back on. Board B has a 5V short,
and the re-fitted resistor on B reads 0 ohm where A reads 1.5 kohm. So the
resistor is 1.5 kohm and sits on the 5V circuit.

Known values so far:

| Part      | Value    | Notes                                  |
|-----------|----------|----------------------------------------|
| Resistor  | 1.5 kohm | Measured in place on Board A           |
| Cap 1     | ?        |                                        |
| Cap 2     | ?        |                                        |

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
