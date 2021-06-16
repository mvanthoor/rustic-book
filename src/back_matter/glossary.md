# Glossary

- B
    - **Bitboard**: An integer of 64 bits long, which is part of the
      engine's board representation.
- C
    - **CECP**: Chess Engine Communication Protocol. A way for a chess
      engine and a (G)UI to communicate with one-another; it originated as
      part of the XBoard/WinBoard GUI. Therefore many people call this either
      "XBoard" or "WinBoard" for short. See "WinBoard" and "XBoard" for
      more information.
    - **Chess Engine**: The chess-playing entity of a chess program.
- G
    - **GUI**: Graphical User Interface, used to control the chess engine
      to view the output, use it to analyze, or to play against it.
- M
    - **Move ordering**: A technique to make the engine try promising moves
      before other moves in the move list, in an effort to reduce the
      number of moves it has to search.
- P
    - **Principal Variation (PV)**: The main line in the game.
    - **PV-Move**: A move that is part of the main line.
    - **PST** or **PSQT**: Piece-Square Tables. This is a set of tables
      which tells the engine which are good squares for each piece type.
- T
    - **Transposition Table (TT)**: A structure which stores moves and
      information about positions that were visited before, by taking
      different paths through the search tree.
    - **TT-Move**: A move returned from the transposition table.
    - **TT-Value**: A value returned from the transposition tbale.
- U
    - **UCI(-protocol)**: A way for the chess engine and GUI to communicate
      with one-another.
- W
    - **WinBoard(-protocol)**: WinBoard is the Windows-version of the
      XBoard GUI. The WinBoard and XBoard-protocols are the same thing.
- X
    - **XBoard(-protocol)**: A way for the chess engine and GUI to
      communicate with one-another. XBoard is also a graphical user
      interface for running chess engines under Unix-like operating
      systems.



