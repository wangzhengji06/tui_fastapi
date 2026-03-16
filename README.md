# Bank Account Simulator

Used to simulate the behavior of a bank account

## Current Structure

```
├── app
│   ├── actions.py
│   ├── context.py
│   ├── core.py
│   ├── domain
│   │   ├── account.py
│   │   ├── **init**.py
│   │   └── setup.py
│   ├── **init**.py
│   ├── network.py
│   ├── render_spec.py
│   ├── setup.py
│   ├── states
│   │   ├── **init**.py
│   │   ├── input_state.py
│   │   ├── login_state.py
│   │   ├── menu_state.py
│   │   ├── quit_state.py
│   │   └── state.py
│   └── tui
│   ├── **init**.py
│   ├── **pycache**
│   │   ├── **init**.cpython-313.pyc
│   │   └── tui.cpython-313.pyc
│   └── tui.py
├── bank.db
├── exchange_cache.json
├── main.py
├── pyproject.toml
├── README.md
├── requirements.txt
├── test_bank.py
├── test_states.py
└── uv.lock
```

### main.py

main.py is the main entry point.
Right now it creats an app object and creates a tui object.
App can render the object passed to tui given its current state.
App can dispatch its event to move to next state.
tui can read the user's input based on keyboard.

Keyboard -> TUI -> App -> Render -> TUI

/app stores all the source code for the app object.
In hindsight, maybe I should not define TUI in the same folder, but I am too lazy to move it.

### domain/account.py domain/setup.py

/app/domain defines the data class of bank accounts. This place is like the business model.
BankAccount data class has id, pin, \_balance. It has deposit, withdraw, and getbalance as class method.
AccountStorage data class defines the context that connects to sqlite database, has create, get, update function for the database.
I think I might to rewrite this two classes using pydantic dataclass and sqlalchemy...
Also the password stores inside the database is encrypted.

App -> AccountStorage(Sqlite) -> BankAccount -> AccountStorage

Note: setup.py creates an account that has id 1234, pin 4321 and balance 1000 initially.

### states/input_state.py login_state.py, menu_state.py, quit_state.py

Defines different states of the app, input_state, login_state, menu_state, quit_state.
THe state has on_enter, on_ui, on_text, and render that can takes an appcontext, action or string.

App(Context, Current State) -> State(On enter, On ui ...) takes Action or string given the current state and context -> New State

### tui/tui.py

Defines the Tui class that composites a terminal object from blessed. The Tui has read that can read the input for the terminal object, the draw function that reads the RenderSpec and draws the actually graph on terminal.

### Actions.py

Defines the Enum class that shows the Action that can be sent to Terminal.

### RenderSpec.py

Defines the UI(?) that will be rendered by TUI class.

### Network.py

Mainly Implements the get_exchange_rates function. This will return the exchange rate from USD to other countries.

### context.py

Protocol of app.py. Just understand it as the class where methods are available to the states safely.
The methods include login, logout, balance, format_amount, deposit and withdraw.

### core.py

The main operations of app.

login -> returns a BankAccount Object that allows for further manipulation.

logout -> overwrites the BankACcount Object to None.

balance -> returns the current balance in the bank account if logged in.

format_amount -> returns the amount with a $ sign attached.

deposit -> deposits the money, updates the AccountStorage

withdraw -> withdraw the money, updates the AccountStorage

render -> Render the current state of the app

dispatch -> Given an action or string inputted, it will goes to the next state

convert_balance_to -> Call the api defined in network.py and returns the balance in another currency

## The plan to add fastapi as the entry point

My biggest problem is currently the core use self.\_account as the way to tell wether the user is logged in or not.

This is of course okay for TUI, because one user one app. For fastapi as a server, the app would recieve call from different clients.
If I create app per request, this would not presist the login state.

First the Get accounts endpoint require me to add a function that can get the BankAccount without providing the pin.

Second I am currently not closing the db connection at all and keep all the transactions in the same connection session, this is okay for the tui but not good for fastapi.

## Adding the test file for fastapi

Now my way of defining everything backfires at me. Now I understand why it is better to use depends, and it works very well with the pytest depedency_overrides.
