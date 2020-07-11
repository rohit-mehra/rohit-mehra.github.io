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

A simple utility to time any python code block.

#### Pre-requisites

1. **[Context Manager][context_manager]** - an object that defines runtime context for the desired code block.
   * **[`__enter__()`][enter_into]** method handles the entry into the code block - as in execute this before executing the code block..
   * **[`__exit__(exc_type, exc_val, exc_tb)`][exit_from]** method handles the exit from the runtime context - as in execute this after executing the code block..
   * Context manager objects find usage in saving and restoring various kinds of global state, locking and unlocking resources, closing opened files, etc.
 
2. **[The `with` statement][with_statment]** - This statement encapsulates `try…except…finally` usage patterns with the help of [context manager][context_manager] objects. The desired behavior is defined in the context manager and used in the `with` statement to execute it.

## Timer context manager

Using [contextmanager decorator][cm_dec], we will define a timer context manager. Once we define this we can reuse this as required. The desired behavior is to -
1. note start time before executing the code block
2. yield to execute the code block
3. note the end time and print the execution time

We will define a generator-iterator to do this. There is no need to return a `value` but if desired in we can return the start time.

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(code_block_name: str) -> None:
  start_time = time.time()
  yield
  print(f'[{code_block_name}] completed in {time.time() - start_time:.0f} s..')
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
```Python
[Square root of abs col 'X'] completed in 1 s..
```

[context_manager]: https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers
[enter_into]: https://docs.python.org/3/library/stdtypes.html#contextmanager.__enter__
[exit_from]: https://docs.python.org/3/library/stdtypes.html#contextmanager.__exit__
[with_statment]: https://docs.python.org/3/reference/compound_stmts.html#the-with-statement
[cm_dec]: https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager