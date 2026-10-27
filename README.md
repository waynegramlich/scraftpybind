# `pysolidcraft`: Python Bindings for SolidCraft

[SolidCraft](https://github.com/tangency-labs/solidcraft) is an open
source CAD program in which every action is a command, reachable from
`solidcraft-cli`, a JSON control port and an MCP server.  `pysolidcraft`
turns each of those commands into a typed Python method, so a design
can be written as an ordinary Python program:

```python
from pysolidcraft import PySolidCraft, connect

# A GUI started with `solidcraft --control 8080`; or "headless".
with connect("8080") as transport:
    solidcraft = PySolidCraft(transport)

    # A 100 x 60 mm rectangle with a 20 mm hole, extruded 10 mm.
    solidcraft.sketch.create(plane="XY", name="Plate")
    rectangle = solidcraft.sketch.rectangle_center(
        center=(0, 0), corner=(50, 30)
    )
    solidcraft.sketch.circle_center(center=(0, 0), diameter=20)
    solidcraft.sketch.finish()
    solidcraft.solid.extrude(
        distance=10,
        profiles=[{"curves": rectangle["curves"]}],
        body_name="Plate",
    )
```

This is what `python3 pysolidcraft.py 8080` runs;
`examples/plate_with_holes.py` is a longer example.

## Status

An early, unofficial project, not affiliated with the SolidCraft
developers.  It is published to see whether other SolidCraft users want
Python bindings; comments are welcome in the issues.

## How It Works

* `PySolidCraft` holds one object per command namespace:
  `sketch.rectangle.center` is `PySolidCraft.sketch.rectangle_center()`.
  Parameters left at `None` are not sent, so SolidCraft uses its own
  defaults.
* A `Transport` carries the requests:
  * `ControlTransport` talks to a running SolidCraft GUI started with
    `solidcraft --control PORT`, so the design can be watched as it is
    built.
  * `HeadlessTransport` runs `solidcraft-cli mcp --headless`, a design
    session with no window (useful with `file.export`).
* `connect("8080")` or `connect("headless")` picks one.
* The command classes are generated: `pysolidcraft_generate.py` reads
  the command list from `solidcraft-cli` (or the cached
  `pysolidcraft_commands.json`) and rewrites the generated region of
  `pysolidcraft.py`.

## Requirements

* Python 3.11 or newer; nothing outside the standard library.
* SolidCraft: the GUI for `ControlTransport`, and `solidcraft-cli` on
  the `PATH` for `HeadlessTransport`.  The commands were generated from
  `solidcraft-cli` 0.20.0.

## Trying It

```
python3 pysolidcraft.py 8080        # A GUI started with --control 8080.
python3 pysolidcraft.py headless    # Writes pysolidcraft_plate.step.
```

## License

MIT No Attribution (`LICENSE`).  SolidCraft itself is MIT licensed.
