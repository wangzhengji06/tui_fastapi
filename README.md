# Bank Account Simulator

Used to simulate the behavior of a bank account


### main.py

main.py is the main entry point.
Right now it creats an app object and creates a tui object.
App can render the object passed to tui given its current state.
App can dispatch its event to move to next state.
tui can read the user's input based on keyboard.

Keyboard -> TUI -> App -> Render -> TUI
