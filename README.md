# Two dead Zotac RTX 3090 Trinity boards

A bench log for two identical Zotac RTX 3090 Trinity cards that arrived dead.
Both turned out to have the same fault. This page gives the conclusion first,
then the fault chain, then how we got there. The raw, dated measurements
live in the files linked at the bottom.

## Short version

- **Symptom.** Card is dark: no fans, no lights, never appears in `lspci`.
  The host PC boots fine with it installed, so the card is not shorting
  under power. It just never enables.
- **Physical damage.** On both boards, the same three parts had been knocked
  off the front of the PCB beside a mounting hole, by a standoff: two small
  capacitors and a 1 mm four-pad IC, silkscreen **U15**. Same spot, same
  parts, on both cards.
- **What U15 is.** A 5V load switch. Its output feeds the control supply
  (VCC, pin 21) of the GS9216 buck regulator that makes the 1.8V rail.
  With U15 gone, the 1.8V regulator has 12V at its power input but no
  control supply, so it never runs. Without 1.8V and its power-good, the
  PEX regulator's enable never goes high, and with no PEX rail the GPU
  core, memory and PCIe link never come up. One missing part, whole card
  dead.
- **A self-inflicted detour.** U15 was first mistaken for a resistor array,
  and two 1.5 kohm resistors were fitted on its pads on both boards. After
  that re-soldering the 5V rail read as a short on both cards, which cost a
  few sessions. Redoing the joints on 2026-09-09 cleared it; the exact cause
  of the short was never pinned down. The resistors have since been lifted
  from Board B.
- **Fix.** A 0 ohm link across U15's input pad (5V) and output pad, on
  each board. U15's enable was tied high through a pull-up anyway, so a
  hard link does what the switch did. Then a boot test.

Status as of 2026-09-10:

| Board | Serial     | State                                                            | Log |
|-------|------------|------------------------------------------------------------------|-----|
| A     | 0089973886 | 5V short cleared. The two 1.5k resistors are still on the U15 pads and must come off. Then fit the 0R link. | [boards/A-0089973886.md](boards/A-0089973886.md) |
| B     | 0091224540 | 5V short cleared, resistors lifted, U15 pads bare. Ready for the 0R link and a boot test. | [boards/B-0091224540.md](boards/B-0091224540.md) |

Neither board has been boot-tested with the fix yet. The chain below is
confirmed up to the point where the 1.8V regulator's VCC is dead; whether
that is the *only* remaining fault is what the boot test will show.

## The fault chain

Power-up on this board, as pieced together from measurements:

```
12V (8-pin and slot)
  |
  +--> 5V buck (inductor bottom right, front)
  |      |
  |      +--> HT32F52241 MCU (housekeeping / sequencer, runs on slot 3.3V)
  |      |
  |      +--> U15, 5V load switch  <-- MISSING on both boards
  |             |
  |             +--> VCC (pin 21) of the 1.8V GS9216 (bottom left, front)
  |
  +--> 1.8V GS9216 power stage (AIN/VIN at 12V, waits on VCC)
         |
         +--> 1.8V rail --> power-good --> EN of the PEX GS9216 (top left, front)
                                              |
                                              +--> PEX rail --> core, memory, link
```

Measured signature with U15 missing (both boards, once the 5V short was
cleared):

| Point                               | Reading | Meaning                                   |
|-------------------------------------|---------|-------------------------------------------|
| 12V at the 8-pins                   | 12V     | input fine                                |
| 5V inductor                         | 5V      | 5V buck fine                              |
| MCU supply                          | 3.1V    | MCU powered, idle                         |
| 1.8V inductor                       | 300 mV  | regulator not running, leakage only       |
| PEX GS9216: AIN                     | 12V     | chip has power                            |
| PEX GS9216: VCC                     | 5V      | chip is alive                             |
| PEX GS9216: EN                      | 1.5V    | just under the 1.6V threshold, never starts |
| PEX inductor                        | 0V      | no PEX rail                               |
| Core and memory caps                | 0V      | nothing downstream                        |

The clinching test was on Board B with nothing on the U15 pads: the 1.8V
inductor sat at 300 mV. Earlier, with a 1.5k accidentally bridging U15's
input and output pads, the 1.8V regulator's VCC sat at 2.7V and the rail
limped up to 1.8V but never asserted power-good. Remove the bridge and 1.8V
collapses again. The 1.8V regulator only runs when something feeds its VCC
through the U15 output net.

## How we got there

A condensed timeline. The dead ends are kept because they explain why the
logs contradict themselves in places.

1. **2026-09-04.** Board A tried in a PC. Dark. PC unaffected.
2. **2026-09-05.** Both boards inspected. Same three parts missing on each,
   beside a mounting hole. Assumed to be two caps and a resistor array;
   caps re-soldered, "array" replaced with two discrete 1.5k. Rail
   resistances to ground all normal except 5V, which read as a short on
   both. First powered test on a riser: 12V in, 5V at 30 mV, 1.8V at
   300 mV, everything else 0V. Identical on both boards.
3. **2026-09-09.** Rework at the 5V spot cleared the short on both boards.
   Powered in an x299 host: 5V and 1.8V now up, PEX GS9216 alive but EN
   stuck at 1.5V, no PEX, no link. The chip behind the 5V inductor
   identified as the HT32F52241 MCU, not a regulator, and confirmed
   powered.
4. **2026-09-09, late.** Pre-rework photo re-examined: the "resistor
   array" was a 1 mm four-pad IC, U15. Pad map from probing: top-right
   5V in, bottom-right enable with a 470k pull-up to 3.3V, top-left
   output, bottom-left and centre ground. Several wrong guesses at its
   job followed (3.3V LDO for the MCU, 5V supervisor driving PEX EN),
   each ruled out by a measurement. See
   [common.md](common.md) for the full sequence.
5. **2026-09-10.** Resistors lifted from Board B. The U15 output net was
   traced to pin 21 (VCC) of a second GS9216 beside the 1.8V inductor.
   With the pads bare, the 1.8V inductor fell back to 300 mV. U15 is a
   5V load switch feeding the 1.8V regulator's VCC. Fix is a 0R link.

## Still open

- Boot test with the 0R link on Board B: expect 1.8V, PEX GS9216 EN above
  1.6V, the PEX inductor around 1.0 to 1.1V, core caps live, card in `lspci`.
- If EN still sits at 1.5V with 1.8V up, the enable logic between the 1.8V
  power-good and PEX EN is the next target. U811, a SOT-23-6 marked CE5
  beside the 1.8V GS9216, is the candidate.
- Board A: lift the resistors, fit the link, repeat.
- Original U15 part number is unknown. No boardview or schematic exists
  for this card. If the 0R link proves out, it is a permanent fix; a real
  load switch is only needed if something ever wants to sequence that rail.

## Repo layout

- [common.md](common.md): findings that apply to both boards, written as a
  running log with the wrong turns and their corrections left in.
- `boards/`: one dated log per board, named `<letter>-<serial>.md`. The
  boards look identical; serial is the only reliable way to tell them apart.
- [notes/rails.md](notes/rails.md): expected resistance to ground per rail,
  and where the small regulators sit on this PCB.
- [notes/gs9216.md](notes/gs9216.md): the GS9216 buck, pinout and a
  powered test sequence.
- [notes/fuses.md](notes/fuses.md): where input fuses would be and how to
  test them. This board has none.
- `notes/datasheets/`: the GS9238 family datasheet, used for the GS9216
  pinout.

## Conventions

- Each board file has a dated log. Append, do not rewrite history. If a
  conclusion turns out wrong, add a correction rather than editing the
  original entry.
- Record measurements with the date and the conditions: powered or not,
  which slot or riser, which host.
- Positions like "top left inductor" are as seen from the front of the
  card. Where a note defines its own orientation (the U15 pad map does),
  that definition wins for that note.
