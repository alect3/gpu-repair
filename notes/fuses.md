# Input fuses

Where to look and how to test on the Zotac RTX 3090 Trinity.

**2026-09-05: no fuses found on either board.** Inspected the areas around
both 8-pin connectors and the edge connector. This board appears to rely on
the PSU for overcurrent protection. Kept the notes below in case a later
inspection (under the backplate, or on the back of the board) turns one up.

## Where they are

There is no single fuse. Each 12V input has its own protection:

- **8-pin PCIe connectors (two on this board).** The fuse for each sits on
  the board within a couple of centimetres of the connector, in series with
  the 12V pins. Look for a small rectangular SMD part, often white, grey or
  black, marked with a current rating or a letter code. Some are chip fuses
  that look like a resistor but are marked with an F or a number such as
  "10" or "12" rather than a resistor code.
- **PCIe slot 12V.** Same idea, near the edge connector on the 12V fingers
  (B1 to B3 and A2 to A3). This one is usually smaller.

The fuse is upstream of the current-sense shunts. The shunts are the
very low value resistors (marked R005, R002, 2m0 or similar) that the
board uses to measure input power. So the 12V path is:

    connector pin -> fuse -> shunt -> VRM high-side FET drains / bulk caps

Follow the wide 12V copper from a connector pin and the fuse is the first
series component you hit.

## How to test

Unpowered, meter on continuity or low-ohms:

1. Probe both ends of the fuse. A good fuse reads near zero ohms. An open
   fuse reads open or many kilohms.
2. Or go end to end: probe from a 12V pin on the 8-pin connector to the
   drain of a high-side FET on the core VRM. That checks fuse and shunt in
   one go.

If a fuse is open, do not just bridge it. Something downstream drew enough
current to blow it, so find that first. The 12V rail on both boards reads
normal to ground, so if a fuse is open here the short may have been
transient, or on a branch that is now isolated by the blown fuse.

## Record

| Board | Fuse | Reading | Date |
|-------|------|---------|------|
| A     | none found | n/a | 2026-09-05 |
| B     | none found | n/a | 2026-09-05 |

## If there are no fuses

Do the end-to-end check instead. Unpowered, continuity from each 12V pin on
the 8-pin connectors to the drain of a high-side FET on the core VRM, and
from the slot 12V fingers to the same point. That confirms the whole input
path is intact regardless of what is in it.
