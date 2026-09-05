# GPU repair log

Tracking broken GPUs, what is wrong with them, and what has been tried, so
nothing gets lost between sessions.

## Boards

Both boards are Zotac RTX 3090 Trinity cards. They look identical, so the only reliable
way to tell them apart is the serial number.

| Board | Serial       | Status                  | Notes                              |
|-------|--------------|-------------------------|------------------------------------|
| A     | 0089973886   | Dead, no power at all   | [boards/A-0089973886.md](boards/A-0089973886.md) |
| B     | 0091224540   | Short on 5V rail        | [boards/B-0091224540.md](boards/B-0091224540.md) |

## Common findings

See [common.md](common.md) for issues seen on more than one board.

## Notes

General reference under `notes/`:

- [notes/fuses.md](notes/fuses.md): where the input fuses are and how to test them.
- [notes/gs9216.md](notes/gs9216.md): the small-rail buck regulator, pinout and how to test it.
- `notes/datasheets/`: saved datasheets.

## Conventions

- One file per board under `boards/`, named `<letter>-<serial>.md`.
- Each board file has a dated log. Append, do not rewrite history.
- Record measurements (rail voltages, resistance to ground) with the date and
  the conditions (powered, unpowered, which connector).
