# TIDALCHESS — tiny-chess-01 entry

A chess engine in a single 4 KB file (`main.py`), written from scratch for
the make.fun tiny-chess-01 competition. Python standard library only,
deterministic fixed-node search (no clock, no randomness, no model calls).

## Design

- **Board:** 0x88 array, integer piece codes, Zobrist hashing (fixed seed).
- **Search:** negamax alpha-beta with iterative deepening to a fixed budget
  of 60,000 nodes per move; transposition table, killer moves, history
  heuristic, null-move pruning (R=2, R=3 at depth >= 6), late-move
  reductions, check extensions, quiescence search (captures, promotions,
  evasions in check), repetition detection.
- **Eval (side-to-move, centipawns):** material + formula piece-square
  tables + bishop pair + pawn structure (doubled, isolated, passed) +
  rook open/semi-open files + king middlegame/endgame taper + tempo.
- **Protocol:** talks to the make.fun game environment over HTTP
  (`/game`, `/move`, `/resign`); root moves come from the env's legal
  move list; the board is rebuilt by replaying the game moves each turn.

## Files

- `main.py` — the entrypoint, packed from `engine.py` with
  `makefun compress` (lzma payload, 4,035 bytes).
- `engine.py` — the readable source (NOT committed: it exceeds the
  4,096-byte per-file cap; it is sent beside the sessions by
  `makefun bundle` via the compress record).
- `LICENSE` — MIT.

## Inspiration

The 4 KB Python engine concept is inspired by Thomas Ahle's sunfish
(github.com/thomasahle/sunfish). All code here was written from scratch
for this entry; no sunfish code was copied.

## Local results (final engine: 60k nodes/move, PVS root, R=3 null-move)

- Practice trial 0 (6 games, earlier 50k build): 5/6, Elo ~1654.
- 8-game gauntlet incl. nodes-1M (1667) and nodes-2M (1712), final
  engine: 5.5/8, Elo ~1598 — wins vs depth-2, depth-3, nodes-100k,
  nodes-250k, nodes-1M; draw vs depth-4; losses vs nodes-500k,
  nodes-2M. Trial wall time 1086 s (limit 1800 s).
- Sunfish baseline practice mean is 1325; this engine beats it by
  roughly 250 Elo locally.
- Perft verified to depth 4 on startpos, kiwipete, and the standard
  en-passant / castling / promotion suites.
