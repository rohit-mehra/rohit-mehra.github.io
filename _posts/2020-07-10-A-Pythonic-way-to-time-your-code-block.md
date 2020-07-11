---
title: "A Pythonic way to time your code block"
date: 2020-07-10T22:03:33-0500
categories:
  - python
  - util
tags:
  - python
  - util
  - timer
  - date
---

A simple utility to time your python code block.

#### Pre-requisites

1. **[Context Manager][context_manager]** - an object that defines runtime context for the desired code block.
   * **[`__enter__()`][enter_into]** method handles the entry into the code block - as in execute this before executing the code block..
   * **[`__exit__(exc_type, exc_val, exc_tb)`][exit_from]** method handles the exit from the runtime context - as in execute this after executing the code block..
   * Context manager objects find usage in saving and restoring various kinds of global state, locking and unlocking resources, closing opened files, etc.
 
2. **[The `with` statement][with_statment]** - This statement encapsulates `try…except…finally` usage patterns with the help of [context manager][context_manager] objects. The desired behavior is defined in the context manager and used in the `with` statement to execute it.

## Timer context manager

Using [@contextmanager][cm_dec] decorator, we will define a timer context manager. Once we define this we can reuse this as required. The desired behavior is to -
1. note start time before executing the code block
2. yield to execute the code block
3. note the end time and print the execution time

We will define a generator-iterator to do this. There is no need to return a `value` but if desired in we can return the start time.

```python
from contextlib import contextmanager
import time

# Simplest
@contextmanager
def timer(code_block_name: str) -> None:
    start_time = time.time()
    # yields the control to the with statement
    yield
    # executed after with code block execution
    execution_time = time.time() - start_time
    print(f"[{code_block_name}] completed in {execution_time:.0f} seconds..")
```

#### Usage

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({'X' : np.random.randint(low=-10, high=10, size=1000000),
                   'Y' : np.random.normal(0.0, 1.0, size=1000000)})

with timer("Square root of abs col 'X'"):
    df['X_square_root'] = df['X'].apply(lambda x: np.sqrt(abs(x)))

```
```python
[Square root of abs col 'X'] completed in 1 seconds..
```

#### Another version

Context manager yielding start time.

```python
from contextlib import contextmanager
import time
from datetime import datetime

# Return Start Date and time
# just to show the use of `as` assignment in the `with` statement
@contextmanager
def timer_v2(code_block_name: str) -> None:
    start_time = time.time()
    # yields the value to the with statement
    yield datetime.now()
    # executed after with code block execution
    execution_time = time.time() - start_time
    print(f"[{code_block_name}] completed in {execution_time:.0f} seconds..")
```

Using the v2 version - which yields the start time. The start time can be assigned to a variable in the with statement as followseconds..

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({'X' : np.random.randint(low=-10, high=10, size=1000000),
                   'Y' : np.random.normal(0.0, 1.0, size=1000000)})

with timer_v2("Square root of abs col 'X'") as current_time:
    print(f"Started at {current_time}")
    df['X_square_root'] = df['X'].apply(lambda x: np.sqrt(abs(x)))
```

```bash
Started at 2020-07-11 13:45:45.194144
[Square root of abs col 'X'] completed in 1 seconds..
```

#### Class based implementation of timer

```python
import time

class Timer:
    def __init__(self, code_block_name):
        # init the context manager with a name
        self.code_block_name = code_block_name
    def __enter__(self):
        # executes and yields the control to the code block
        self.start_time = time.time()
    def __exit__(self, type, value, traceback):
        # executed after with code block execution
        execution_time = time.time() - self.start_time
        print(f"[{self.code_block_name}] completed in {execution_time:.0f} seconds..")
```

```python
with Timer("for loop 100,000,000"):
    _ = [x for x in range(100_000_000)]
```

```python
[for loop 100,000,000] completed in 4 seconds..
```

[context_manager]: https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers
[enter_into]: https://docs.python.org/3/library/stdtypes.html#contextmanager.__enter__
[exit_from]: https://docs.python.org/3/library/stdtypes.html#contextmanager.__exit__
[with_statment]: https://docs.python.org/3/reference/compound_stmts.html#the-with-statement
[cm_dec]: https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager