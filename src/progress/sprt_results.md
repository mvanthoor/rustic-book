
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [SPRT Results](#sprt-results)
  - [TT-cuts](#tt-cuts)
  - [TT-move ordering](#tt-move-ordering)
  - [Killer Move Heuristic](#killer-move-heuristic)
  - [Principal Variation Search](#principal-variation-search)
  - [Killer Move heuristic + PVS](#killer-move-heuristic-pvs)

<!-- /code_chunk_output -->


# SPRT Results

Below are the SPRT results between the different Rustic versions, for
people who are interested in the specific results. The list starts from the
oldest version at the top, down to the latest version.

## TT-cuts

```
Score of Rustic Alpha 1.5 vs Rustic Alpha 1.1: 869 - 646 - 357  [0.560] 1872
...      Rustic Alpha 1.5 playing White: 431 - 321 - 184  [0.559] 936
...      Rustic Alpha 1.5 playing Black: 438 - 325 - 173  [0.560] 936
...      White vs Black: 756 - 759 - 357  [0.499] 1872
Elo difference: 41.6 +/- 14.2, LOS: 100.0 %, DrawRatio: 19.1 %
SPRT: llr 2.95 (100.3%), lbound -2.94, ubound 2.94 - H1 was accepted
```

## TT-move ordering

```
Score of Rustic Alpha 2 vs Rustic Alpha 1.5: 409 - 197 - 133  [0.643] 739
...      Rustic Alpha 2 playing White: 218 - 84 - 68  [0.681] 370
...      Rustic Alpha 2 playing Black: 191 - 113 - 65  [0.606] 369
...      White vs Black: 331 - 275 - 133  [0.538] 739
Elo difference: 102.5 +/- 23.5, LOS: 100.0 %, DrawRatio: 18.0 %
SPRT: llr 2.95 (100.1%), lbound -2.94, ubound 2.94 - H1 was accepted
```

## Killer Move Heuristic

```
Score of Rustic Alpha 2.1.100 vs Rustic Alpha 2: 628 - 416 - 277  [0.580] 1321
...      Rustic Alpha 2.1.100 playing White: 357 - 167 - 137  [0.644] 661
...      Rustic Alpha 2.1.100 playing Black: 271 - 249 - 140  [0.517] 660
...      White vs Black: 606 - 438 - 277  [0.564] 1321
Elo difference: 56.2 +/- 16.8, LOS: 100.0 %, DrawRatio: 21.0 %
SPRT: llr 2.94 (100.0%), lbound -2.94, ubound 2.94 - H1 was accepted
```

## Principal Variation Search

```
Score of Rustic Alpha 2.2.100 vs Rustic Alpha 2.1.100: 591 - 388 - 318  [0.578] 1297
...      Rustic Alpha 2.2.100 playing White: 334 - 171 - 143  [0.626] 648
...      Rustic Alpha 2.2.100 playing Black: 257 - 217 - 175  [0.531] 649
...      White vs Black: 551 - 428 - 318  [0.547] 1297
Elo difference: 54.8 +/- 16.6, LOS: 100.0 %, DrawRatio: 24.5 %
SPRT: llr 2.95 (100.3%), lbound -2.94, ubound 2.94 - H1 was accepted
```

## Killer Move heuristic + PVS
```
Score of Rustic Alpha 2.2.100 vs Rustic Alpha 2: 423 - 218 - 174  [0.626] 815
...      Rustic Alpha 2.2.100 playing White: 221 - 106 - 80  [0.641] 407
...      Rustic Alpha 2.2.100 playing Black: 202 - 112 - 94  [0.610] 408
...      White vs Black: 333 - 308 - 174  [0.515] 815
Elo difference: 89.3 +/- 21.7, LOS: 100.0 %, DrawRatio: 21.3 %
SPRT: llr 2.96 (100.4%), lbound -2.94, ubound 2.94 - H1 was accepted
```

---

> __Sidenote__: Version Alpha 1.5 is a seperate version for testing
> transposition table cuts without hash the TT-move ordering. This has not
> been released as an official version. The versions with patch numbers of
> 100 and up are for testing each features.

