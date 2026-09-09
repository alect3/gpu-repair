# Rail reference

Typical unpowered resistance to ground on an RTX 3090, meter on the lowest
ohms range. Subtract the meter's own floor (probes touched together).

| Rail             | Typical          | Measured, both boards (2026-09) |
|------------------|------------------|---------------------------------|
| Core (NVVDD)     | 1 to 5 ohm       | 1 to 5 ohm                      |
| Memory (MSVDD, FBVDD) | 5 to 30 ohm | normal, value not recorded      |
| PEX              | 20 to 100 ohm    | normal, value not recorded      |
| 1.8V             | hundreds of ohm  | normal, value not recorded      |
| 5V               | hundreds to kohm | was shorted on both, now normal |
| 3.3V             | hundreds to kohm | normal, value not recorded      |
| 12V              | kohm             | normal, value not recorded      |

A dead short reads at the meter floor and stays there in both polarities.
A healthy low rail reads a little above the floor and drifts upwards over a
few seconds as the bulk capacitors charge.

The core rail is low because the GPU die is a large parallel load. Do not
mistake it for a short. Compare against the other board instead.

## Where the small regulators are (front view)

| Position     | Rail             | Regulator                          |
|--------------|------------------|------------------------------------|
| Bottom right | 5V               | inductor only; the IC behind it on the back is the HT32F52241 MCU, not the regulator |
| Bottom left  | 1.8V (assumed)   | not yet identified                 |
| Top left     | unknown, 1.8V or PEX | GS9216, see gs9216.md          |
| Top row, full size choke | probably PEX | full phase                 |
