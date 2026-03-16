# 3D Four-In-a-Row — Design Document

## 1. Overview

3D Four-In-a-Row is a browser-based strategy game played on a 4x4x4 cubic grid. Two players take turns placing tokens, aiming to align four in a straight line across any axis or diagonal. The game supports local two-player, AI opponents (easy/hard), and real-time online multiplayer.

The entire application is delivered as a single self-contained HTML file with no build process or external dependencies beyond CDN-hosted libraries.

## 2. Tech Stack

| Layer              | Technology                                  |
|--------------------|---------------------------------------------|
| Game Logic & UI    | Vanilla JavaScript (ES2020)                 |
| Styling            | CSS3 with custom properties, Flexbox        |
| 3D Rendering       | Three.js r128 (CDN)                         |
| Online Multiplayer | Firebase Realtime Database v9.23.0 (compat) |
| Typography         | Nunito (Google Fonts)                       |
| PWA                | Service Worker + Web App Manifest           |

## 3. Architecture

### 3.1 High-Level Component Diagram

```
┌─────────────────────────────────────────────────────┐
│                   User Interface                     │
│  ┌────────────┐  ┌────────────┐  ┌───────────────┐  │
│  │ Scoreboard │  │  2D Board  │  │   3D Viewer   │  │
│  │  & Controls│  │  (4 cards) │  │  (Three.js)   │  │
│  └─────┬──────┘  └─────┬──────┘  └───────┬───────┘  │
│        │               │                 │           │
│  ┌─────▼───────────────▼─────────────────▼────────┐  │
│  │              Game State (gs)                    │  │
│  │   board[4][4][4] · turn · scores · names       │  │
│  └────────┬───────────────────────┬───────────────┘  │
│           │                       │                  │
│  ┌────────▼────────┐    ┌────────▼────────┐         │
│  │   AI Engine     │    │  Online Sync    │         │
│  │ (easy / hard)   │    │  (Firebase)     │         │
│  └─────────────────┘    └─────────────────┘         │
│                                                      │
│  ┌──────────────────────────────────────────┐        │
│  │         Persistence (localStorage)        │        │
│  └──────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────┘
```

### 3.2 State Model

All game state lives in a single plain object (`gs`):

```
gs = {
  board:       [4][4][4] array   // 'O', 'X', or null
  turn:        'O' | 'X'
  gameOver:    boolean
  scores:      { O: number, X: number }
  names:       { O: string, X: string }
  emojiPair:   [string, string]  // token emojis
  playerXType: 'human' | 'ai-easy' | 'ai-hard' | 'online'
}
```

Every mutation to `gs` triggers a full re-render of the 2D board via `renderBoard()`. There is no virtual DOM or incremental diffing — the design prioritises simplicity over micro-optimisation.

### 3.3 Data Flow

```
User Click / AI Move / Firebase Event
          │
          ▼
    placeToken(layer, row, col)
          │
          ▼
    Update gs.board → checkWin() → checkDraw()
          │
          ▼
    renderBoard() ─────► DOM rebuild (2D cards)
          │              sync3DView() (if open)
          ▼
    saveState() ────────► localStorage
```

In online mode, local moves do **not** update `gs` directly. Instead:
1. The move is written to Firebase via `onlineWriteMove()`
2. The Firebase listener fires on both clients
3. The listener detects the board diff and updates `gs`
4. This ensures both players always share the same authoritative state

## 4. Game Mechanics

### 4.1 Board & Coordinate System

The board is a 4x4x4 cube addressed as `board[layer][row][col]`:
- **Layer** (0–3): vertical planes displayed as separate cards in 2D
- **Row** (0–3): vertical axis within a layer
- **Column** (0–3): horizontal axis within a layer

### 4.2 Win Detection

The game enumerates **76 unique win lines** at startup via `buildWinLines()`:

| Category                    | Count |
|-----------------------------|-------|
| Rows within each layer      | 16    |
| Columns within each layer   | 16    |
| Diagonals within each layer | 8     |
| Vertical through layers     | 16    |
| Diagonals through layers    | 8     |
| Space diagonals             | 4     |
| **Total**                   | **76**|

After every move, `checkWin(player)` iterates all 76 lines to find four aligned tokens.

### 4.3 Draw Detection

Two draw conditions exist:
1. **Complete draw** — all 64 cells filled, no winner
2. **Inevitable draw** — no unblocked win line remains for either player (both players have 0 open lines)

## 5. Game Modes

### 5.1 Local Two-Player

Both players share the same device. Turns alternate on click.

### 5.2 AI — Easy

| Behaviour               | Detail                                           |
|--------------------------|--------------------------------------------------|
| Wins if possible         | Always takes an immediate winning move            |
| Blocks opponent wins     | ~55% of the time (intentionally misses ~45%)      |
| Move selection           | Heuristic score + heavy random noise (0–12)       |
| Candidate pool           | Top 40% of scored moves, pick randomly            |
| Think time               | 500 ms artificial delay                           |

### 5.3 AI — Hard

| Behaviour               | Detail                                           |
|--------------------------|--------------------------------------------------|
| Wins if possible         | Always takes an immediate winning move            |
| Blocks opponent wins     | Always blocks                                     |
| Heuristic weights        | Own 3-in-line: +100, 2-in-line: +10, 1-in-line: +2; Opponent 3-in-line: +80, 2-in-line: +8 |
| Move selection           | Best score(s), random tiebreak                    |
| Think time               | 500 ms artificial delay                           |

### 5.4 Online Multiplayer

Real-time two-player over Firebase Realtime Database.

**Lobby Flow:**
1. Host creates a game → receives a 4-letter room code
2. Guest enters the code → joins the game
3. Firebase syncs all moves, presence, and game resets

**Firebase Data Path:** `/games/{CODE}`

**Key Design Decisions:**
- Firebase is the single source of truth (no local state mutation in online mode)
- Presence tracked via `onDisconnect()` callbacks
- Stale games pruned client-side after 24 hours of inactivity
- Only the host can initiate new games

## 6. Rendering

### 6.1 2D Board

Four `.board-card` elements arranged horizontally, each containing a 4x4 CSS grid. The entire DOM is rebuilt on every state change. Visual feedback includes:
- Pop-in scale animation on token placement
- Blue highlight on last-placed token
- Gold pulsing animation on winning cells
- Disabled state when game is over

### 6.2 3D Viewer (Three.js)

Activated via the "3D" toggle button. Features:

- **Layer groups** — each of the 4 layers is a separate `THREE.Group`
- **Wireframe cube** — dashed lines with brighter outer edges, dimmer inner edges
- **Emoji sprites** — rendered to 128x128 canvas textures, applied to `THREE.Sprite`
- **Assembly animation** — layers fly in from spread positions on open
- **Slice slider** — separates layers along the Y-axis (0–100%)
- **Auto-rotation** — continuous slow rotation; pauses on drag or win
- **Momentum drag** — velocity sampled at ~30 ms, friction coefficient 0.92
- **Floating labels** — CSS labels that track 3D layer positions via projection

### 6.3 Camera Controls

| Input          | Action             |
|----------------|--------------------|
| Mouse drag     | Rotate cube        |
| Scroll wheel   | Zoom in/out        |
| Touch drag     | Rotate cube        |
| Pinch          | Zoom in/out        |

## 7. Persistence

Game state is serialised to `localStorage` (key: `4iar3d`) after every move and restored on page load. Online sessions are not persisted — refreshing resets to local human mode.

## 8. Responsive Design

| Breakpoint         | Behaviour                                        |
|--------------------|--------------------------------------------------|
| Desktop (≥960px)   | Fixed 220px card widths, scroll-zoom             |
| Tablet (<960px)    | Fluid flex cards, reduced token sizes            |
| Small phone (<450px height) | Hidden turn indicator, minimal spacing  |

Touch events use passive listeners. Safe-area insets support notched devices.

## 9. PWA Support

The app references a `manifest.json` and `sw.js` for installability and offline caching, though these support files are not included in the repository.

## 10. Security & Privacy

- Firebase security rules restrict read/write to `/games/{$gameId}` paths only
- No authentication is required (public games with obscure codes)
- No personal data is collected beyond player-chosen display names
- All game data is ephemeral (24-hour TTL via client-side pruning)

## 11. Dev Mode

Pressing **D** on desktop toggles a debug panel showing:
- Open win lines remaining for each player
- Total empty cells
- Inevitable draw detection status

## 12. File Structure

```
3D-Four-In-a-Row/
├── four-in-a-row.html   # Complete application (2,453 lines)
├── README.md            # Project documentation
├── manifest.json        # PWA manifest (referenced)
├── sw.js                # Service worker (referenced)
└── OIP-2206181049.jpg   # Screenshot asset
```

## 13. Key Design Principles

1. **Single-file delivery** — zero build tooling, instant deployment
2. **State simplicity** — one plain object, full re-render on change
3. **Firebase as authority** — online mode never trusts local state
4. **Progressive enhancement** — 2D board is the baseline; 3D is an optional overlay
5. **Mobile-first interactions** — touch drag, pinch zoom, responsive layout
