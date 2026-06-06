# Lockpick Solver

Lockpick Solver is a self-contained browser application for solving plate-based
lock puzzles. It visualizes every plate, lets you configure directional
connections between plates, and calculates the shortest valid sequence of
button presses that moves every pin to its center hole.

The application runs entirely in the browser. It does not require Python,
Node.js, a web server, an installation process, or an internet connection.

## Features

- Configure between 1 and 10 plates.
- Configure an odd number of holes from 3 to 15.
- Select the starting hole for every plate.
- Define directional movement connections separately for every plate.
- Visualize plates, holes, pins, center positions, and active positions.
- Calculate the shortest solution using breadth-first search.
- Display every move together with the resulting plate positions.
- Automatically arrange long solutions into responsive columns.
- Detect already solved and unsolvable configurations.
- Stop exceptionally large searches after 500,000 visited states.
- Use the included example configuration as a reference.
- Run locally without sending puzzle data anywhere.

## Quick Start

1. Download or copy the project folder.
2. Open `index.html` in a current web browser.
3. Configure the puzzle.
4. Click **Calculate solution**.

Opening the file directly is sufficient. For example, on Windows you can
double-click `index.html`.

## How to Configure a Puzzle

### 1. Set the number of plates

Use the **Number of plates** control at the top of the page.

The application supports 1 to 10 plates. Increasing the number of plates also
increases the number of possible states and may make the search considerably
more expensive.

### 2. Set the number of holes

Use the **Number of holes** control.

Only odd values between 3 and 15 are accepted. An odd number is required
because the solved state is defined as one unambiguous center hole:

```text
center = floor(number of holes / 2)
```

Internally, positions are zero-based. The interface displays them as
human-friendly values starting at 1.

For seven holes, the internal positions are `0` through `6`, while the
interface displays **Hole 1** through **Hole 7**. The target is internal
position `3`, displayed as **Hole 4**.

### 3. Select every starting position

Each plate has a **Starting position** selector on its left. The highlighted
pin in the plate visualization moves immediately when the selection changes.

The outlined center hole is the target position for that plate.

### 4. Define movement connections

Every plate can affect any other plate when it moves. The connection settings
on the right side of a row define what happens when that row's plate is
activated.

For each possible target plate, choose one of the following options:

| Setting | Effect |
| --- | --- |
| **none** | The target plate does not move. |
| **same** | The target plate moves in the same physical direction as the active plate. |
| **opposite** | The target plate moves in the opposite physical direction. |

Connections are directional. A connection from Plate 2 to Plate 3 does not
automatically create a connection from Plate 3 to Plate 2. Configure both
directions explicitly if both actions should affect one another.

### 5. Calculate the solution

Click **Calculate solution**. The application searches all reachable valid
states until it finds the target state.

The result contains:

- The minimum number of moves.
- The plate that must be pressed for every move.
- The direction in which it must be pressed.
- The complete resulting state after every move.
- The number of states checked.
- The calculation time.

## Direction Model

The puzzle uses mirrored controls:

- Pressing a plate to the **right** physically moves that plate to the left.
- Pressing a plate to the **left** physically moves that plate to the right.

Connections are evaluated relative to the physical movement of the active
plate:

- **same** follows its physical movement.
- **opposite** moves against its physical movement.

This behavior matches the supplied Python implementation.

## Example Configuration

Click **Load example** to restore the original six-plate example:

```text
Plates: 6
Holes: 7
Starting positions shown in the UI: 2, 1, 6, 5, 4, 7
Internal zero-based positions:       1, 0, 5, 4, 3, 6
```

The directional connections are:

| Active plate | Connected plate | Direction |
| --- | --- | --- |
| Plate 2 | Plate 3 | same |
| Plate 2 | Plate 5 | opposite |
| Plate 4 | Plate 2 | same |
| Plate 4 | Plate 5 | opposite |
| Plate 5 | Plate 2 | opposite |
| Plate 5 | Plate 3 | same |
| Plate 5 | Plate 6 | opposite |
| Plate 6 | Plate 5 | same |

With the current movement rules, the shortest solution for this example has
40 moves.

## Solver Algorithm

The solver uses breadth-first search (BFS), equivalent to the algorithm in the
original Python script.

Every state is represented by the current position of all plates:

```text
[plate 1 position, plate 2 position, ..., plate N position]
```

For each state, the solver tries both possible presses on every plate:

```text
plate 1 left
plate 1 right
plate 2 left
plate 2 right
...
```

A resulting state is rejected when any plate would move beyond its first or
last hole. Valid states are added to the search queue only once.

Because BFS explores states in increasing move count, the first discovered
target state is guaranteed to use the smallest possible number of moves.

To reduce memory usage, the solver stores a parent reference and the move used
to reach each state. The complete move sequence is reconstructed only after a
solution is found.

## Search Limits and Performance

The theoretical state space grows exponentially:

```text
possible states = holes ^ plates
```

Examples:

| Plates | Holes | Maximum theoretical states |
| ---: | ---: | ---: |
| 4 | 5 | 625 |
| 6 | 7 | 117,649 |
| 8 | 9 | 43,046,721 |
| 10 | 15 | 576,650,390,625 |

Connections and boundary restrictions may make many states unreachable, but
large configurations can still consume significant time and memory.

For browser stability, the application stops after 500,000 visited states. If
this limit is reached, reduce the number of plates or holes, or verify that the
connection model is correct.

The search runs in small asynchronous chunks. This allows the browser to
refresh the progress indicator instead of appearing completely frozen during
larger calculations.

## Understanding the Result

A result step such as:

```text
Press plate 3 to the right
Positions: 4 · 2 · 6 · 5 · 3 · 7
```

means:

1. Press Plate 3 toward the right.
2. The mirrored control moves Plate 3 physically left.
3. Any configured connections from Plate 3 are applied.
4. The listed one-based positions show the state after that complete move.

If the application reports **No solution found**, it has exhausted every
reachable valid state without finding a state where all plates are centered.

If it reports **The lock is already solved**, every plate started in its center
hole and no moves are required.

## Project Structure

```text
.
├── index.html   Application, styling, interface, and solver
└── README.md    Project documentation
```

The project intentionally uses a single HTML file so it can be shared and
opened without a build process.

## Browser Compatibility

The application uses standard HTML, CSS, and modern JavaScript. It should work
in current versions of:

- Google Chrome
- Microsoft Edge
- Mozilla Firefox
- Safari

JavaScript must be enabled.

## Privacy and Offline Use

All calculations happen locally inside the browser. The application:

- Does not send puzzle configurations to a server.
- Does not use analytics.
- Does not load external scripts, fonts, images, or stylesheets.
- Does not require an account.
- Continues to work without an internet connection.

## Development

No build step is required. Edit `index.html` and reload it in the browser.

For development through a local server, any static file server can be used.
For example, if Python is installed:

```powershell
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

The server is optional and is not required for normal use.

## Relationship to the Python Version

The JavaScript solver preserves the important behavior of the original Python
implementation:

- The target is the center hole on every plate.
- Press direction and physical direction are mirrored.
- Connected plates use `+1` for same and `-1` for opposite movement.
- Moves that cross a plate boundary are invalid.
- Breadth-first search returns a shortest solution.

The browser version adds interactive configuration, responsive visualization,
progress reporting, state limits, and formatted solution output.

## License

No license has been specified for this project.
