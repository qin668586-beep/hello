# tiny-chess-01 — a 4 KB Python chess engine

`main.py` is a deterministic alpha-beta engine that fits the 4,096-byte entry
budget by shipping its real source lzma-compressed inside a 3-line stdlib
loader and decompressing it at import time. Source is ~13 KB of plain Python
(`engine.py`); no dependencies, standard library only, Python 3.13.

## Engine

- **0x88 board** with incremental material + piece-square scores (`SC`),
  non-pawn material (`NP`), bishop counts, and per-file pawn counts, all
  maintained by `mk`/`um` so evaluation costs almost nothing per node.
- **Search**: negamax PVS, iterative deepening 1..N plies, transposition
  table with mate-distance correction, killer moves, history heuristic,
  null-move pruning, late-move reductions, check extension and repetition
  detection.
- **Quiescence**: captures only, standing pat plus delta pruning, full move
  ordering (TT move, MVV-LVA, promotions, killers, history).
- **Evaluation**: material, 7 piece-square tables tapered by phase, bishop
  pair, doubled/isolated pawns, king safety corner term.
- **Determinism**: a node budget (`BUD`) replaces wall-clock time, so the
  same position always yields the same move; no randomness, no threads.
- Validated by perft against python-chess (start position, Kiwipete and four
  position-5 style suites, all exact) plus state-by-state diffs.

## Size budget

| file | bytes | limit |
|---|---|---|
| `main.py` (packed) | 4068 | 4096 |
| `LICENSE` | 1104 | 4096 |
| `README.md` | this file | 4096 |
| repository total | ~6.6 KB | 8192 |

## Protocol

The engine speaks the kit env protocol on `MAKEFUN_ENV_URL`: `POST /game` to
start, `POST /move` with `{fen, legal_moves}` to pick a move, and prints one
line per finished game to stderr. Run the kit env with `main.py` to reproduce.

## License

MIT (see `LICENSE`).
