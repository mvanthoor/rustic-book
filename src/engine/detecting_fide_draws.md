# Detecting draws by FIDE rules

To be able to play chess properly, the engine has to be able to detect
draws. In addition, to fully support the XBoard protocol [(see the chapter
about communication)](../communication/communication.html), it has to be
able to claim those draws. This chapter describes how to use the board
representation to detect these draws.