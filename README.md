# tiny-chess-01 — 4KB Python engine

A deterministic alpha-beta chess engine that fits the 4,096-byte `main.py`
budget by shipping the real source **lzma-compressed + base85-encoded** and
decompressing it at import time.

## What is in the engine

- **Negamax + alpha-beta**, iterative deepening 1..6 plies; only fully
  completed iterations are kept (node budget, never wall-clock, so play is
  reproducible move for move).
- **Transposition table** keyed by board string + side to move (depth, score,
  bound flag, best move) — used both to cut and to order moves.
- **PVS**: first move full window, siblings null-window with re-search when
  they raise alpha.
- **Quiescence search** over captures with MVV-style ordering, so the search
  never stops mid-exchange.
- **Evaluation**: material, piece-square tables, bishop pair, rook on
  open/half-open file, doubled and isolated pawns, small tempo bonus.
- **Move generation**: pseudo-legal, filtered by a king-attack test, queen
  promotion, castling rights tracked through the search.

## Size budget

| file | bytes | limit |
|---|---|---|
| `main.py` (packed) | 3749 | 4096 |
| repository total | ~6.2 KB | 8192 |

Unpacked source is ~11.4 KB; the packing wrapper is three lines of stdlib.

## Local score (6 practice games)

| version | score | est. Elo |
|---|---|---|
| greedy starter | 0/6 | 0 |
| alpha-beta + PST | 0.5/6 | 775 |
| + TT + PVS + quiescence | 1.5/6 | 1005 |
| + node budget 250k | 1.5/6 | 1055 |

Kit anchors: depth-2 ≈ 855, depth-3 ≈ 1073, depth-4 ≈ 1199, nodes-100k ≈ 1392.

## Reproduce

The engine speaks the env protocol on `MAKEFUN_ENV_URL` (`POST /game`,
`POST /move`) and prints one line per finished game to stderr:

    game 1: win by checkmate

Run the kit's env + `main.py` for each game to reproduce the table above.
