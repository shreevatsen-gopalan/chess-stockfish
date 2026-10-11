# Chess vs Stockfish

A playable chess game that runs entirely in your browser — no backend, no build step, no dependencies to install. Pick your side, pick an opponent strength, and play against **Stockfish** (compiled to WebAssembly), from beginner-friendly up to full engine strength.

## Features

- **Play as White or Black** — choose your side before the game starts
- **Three opponent strengths** — Easy (~1350 Elo), Intermediate (~1900 Elo), Pro (unrestricted Stockfish)
- **Full chess rules** — castling, en passant, pawn promotion (with a piece-choice dialog), check, checkmate, stalemate, 50-move rule, insufficient-material draws
- **Click-to-move** — click a piece to see its legal moves highlighted, then click a destination
- **Undo** — takes back your last move *and* the computer's reply
- **Restart** — return to the start screen at any time
- **Offline-capable** — the engine ships with the repo and runs locally in a Web Worker
- **Graceful fallback** — if Stockfish can't load, the game falls back to a small built-in JavaScript engine and tells you

## Play it

### Option 1: GitHub Pages (share a link)

After pushing this repo, enable Pages so anyone can play with just a URL:

1. Go to **Settings → Pages** in your GitHub repo
2. Under **Build and deployment**, set **Source** to *Deploy from a branch*
3. Choose branch `main` and folder `/ (root)`, then **Save**
4. After a minute or so, the game is live at `https://shreevatsen-gopalan.github.io/chess-stockfish/`

### Option 2: Run locally

You need to serve the folder over HTTP (see [Why not just open `index.html`?](#why-not-just-open-indexhtml) below). Any static server works. With Python, which is preinstalled on macOS and most Linux systems:

```bash
git clone https://github.com/shreevatsen-gopalan/chess-stockfish.git
cd chess-stockfish
python3 -m http.server 8000
```

Then open <http://localhost:8000> in your browser.

No Python? Any of these also work:

```bash
npx serve .
# or
php -S localhost:8000
```

## Why not just open `index.html`?

Most browsers block Web Workers on pages opened via `file://`, and Stockfish runs inside a Web Worker. If you double-click `index.html`, the game should still work, but it will use the much weaker built-in fallback engine and show an on-screen notice saying so. Serving over HTTP (either option above) gives you the real Stockfish.

## How it works

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: UI, rules engine, move generation, Stockfish integration, fallback AI |
| `stockfish.js` | Emscripten loader that runs Stockfish as a Web Worker |
| `stockfish.wasm` | The Stockfish engine binary (WebAssembly, ~1.8 MB) |
| `LICENSE` | GPL-3.0 |

- The game keeps its own board state and a full legal-move generator. For the computer's turn, it serializes the position to a FEN string and sends `position fen …` and `go movetime …` to Stockfish over the UCI protocol.
- Stockfish's reply (e.g. `e2e4`) is matched against the game's own list of legal moves, so move validation, undo, and end-of-game detection never depend on the engine.
- Difficulty is applied through Stockfish's `UCI_LimitStrength` / `UCI_Elo` options. Pro leaves strength unrestricted and thinks for about 1.8 seconds per move.
- The bundled engine is the **lite, single-threaded** build of Stockfish 19, chosen because it needs no special HTTP headers (no cross-origin isolation) and loads quickly.

## Customizing

Difficulty presets live in `DIFFICULTY_SETTINGS` near the top of the Stockfish section in `index.html`:

```js
const DIFFICULTY_SETTINGS = {
  easy:   { limitStrength: true,  elo: 1350, movetime: 200 },
  medium: { limitStrength: true,  elo: 1900, movetime: 500 },
  pro:    { limitStrength: false, elo: null, movetime: 1800 }
};
```

Raise `movetime` (milliseconds) to give Stockfish more thinking time per move.

## Credits and license

- [Stockfish](https://github.com/official-stockfish/Stockfish) — the chess engine
- [stockfish.js](https://github.com/nmrugg/stockfish.js) by Nathan Rugg — the WebAssembly build used here (v19.0.0, lite single-threaded)

Stockfish and stockfish.js are licensed under the **GNU General Public License v3.0**. Because this repository distributes them, the project as a whole is released under GPL-3.0 as well — see [LICENSE](LICENSE).
