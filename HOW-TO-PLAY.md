# How to Play — 3D Four-in-a-Row

## The Goal

Connect **four tokens in a straight line** on a 4×4×4 three-dimensional grid. The line can run in any direction — horizontal, vertical, diagonal within a layer, or diagonally through multiple layers.

There are **76 possible winning lines** in total, so there's a lot to watch out for!

---

## Game Board

The board is made up of **4 layers**, each containing a **4×4 grid**. Think of it as four floors of a building stacked on top of each other.

You can view the board in two ways:

- **2D view** — All four layers shown side by side (or stacked vertically — press **M** to toggle). Layer 1 is the bottom, Layer 4 is the top.
- **3D view** — An interactive 3D cube you can grab and rotate. Use the slice slider at the bottom to pull the layers apart for a clearer view.

---

## Taking a Turn

Players take turns placing their token into an empty cell. Simply **click on any empty cell** to place your token there. Unlike traditional Connect Four, pieces do not fall due to gravity — you can place a token in any empty cell on any layer.

Each player is represented by a randomly assigned emoji token (e.g. 🍕 vs 🌮).

---

## Winning Lines

A winning line is any straight line of 4 cells. These include:

- **Rows** — 4 in a horizontal line within a single layer
- **Columns** — 4 in a vertical line within a single layer
- **Layer diagonals** — 4 diagonally within a single layer
- **Vertical stacks** — 4 in the same position across all 4 layers (straight up through the cube)
- **Cross-layer diagonals** — 4 diagonally through multiple layers
- **Space diagonals** — 4 running from one corner of the cube to the opposite corner

When someone wins, the winning cells pulse with a highlight in both views.

---

## Game Modes

### Local 2-Player
Two players share the same device, taking turns clicking cells. Great for playing face-to-face.

### Easy AI
Play against the computer at a relaxed difficulty. The AI will try to win and occasionally block your moves, but it makes plenty of mistakes. A good starting point for learning the 3D board.

### Hard AI
A tougher computer opponent. It will always block your winning moves, actively build its own threats, and score positions strategically. A real challenge!

### Online Multiplayer
Play against someone on a different device in real time.

**To host a game:**
1. Open the **Game Mode** menu and select **Host Online Game**
2. Share the 4-letter code with your opponent
3. Wait for them to join — the mode button will turn green when they connect

**To join a game:**
1. Open the **Game Mode** menu and select **Join Online Game**
2. Enter the 4-letter code from the host
3. The game begins as soon as you connect

The mode button shows the connection status:
- 🟢 **Online** — Your opponent is connected
- 🟡 **Waiting** — Waiting for an opponent to join
- 🔴 **Disconnected** — Your opponent lost connection (they may reconnect)

---

## Tips for Beginners

1. **Think in 3D** — Lines can go in directions you might not expect. The 3D view helps you spot threats across layers.
2. **Control the centre** — Centre cells belong to more winning lines than corner or edge cells. Placing tokens in the middle of each layer gives you more options.
3. **Watch all 4 layers** — It's easy to focus on one layer and miss a threat building vertically through the stack.
4. **Use the slice slider** — In 3D view, pull the layers apart to see each one clearly. This makes it much easier to spot cross-layer diagonals.
5. **Try Easy AI first** — If the 3D board feels overwhelming, practise against the Easy AI to build your spatial awareness before taking on the Hard AI or a human opponent.
6. **Look for forks** — Try to create two threats at once. If your opponent can only block one, you win with the other!

---

## Draws

If every cell on the board is filled and no one has four in a line, the game is a draw. The game also detects **inevitable draws** — situations where neither player can possibly complete a winning line with the remaining empty cells — and will call the game early.

---

## Controls Reference

| Action | How |
|---|---|
| Place a token | Click an empty cell |
| Switch 2D/3D view | Click the view toggle buttons (top left) |
| Rotate 3D cube | Click and drag on the cube |
| Zoom in/out | Scroll wheel (desktop) or pinch (mobile) |
| Separate layers | Use the slice slider at the bottom of the 3D view |
| Select a layer (3D) | Click L1–L4 buttons or press 1–4 on keyboard |
| Toggle 2D layout | Press M |
| Edit player name | Click on a player name in the score panel |
| Start new game | Click the **New Game** button |
| Change game mode | Click the **Game Mode** button (shows current mode) |
