---
title: "A parallel version of map() function with Progress Bar"
date: 2020-07-13T21:20:57-0500
classes: wide
categories:
  - python
  - util
tags:
  - python
  - util
  - multiprocessing
  - pool
  - tqdm
---

I work on data and a lot of times I find myself wondering.. 

> How great would it be to run every data pre-processing step in parallel?

There are obvious solutions to accomplish this such as using [vectorization][vectorization_guide] instead of looping over rows in Pandas. [Numpy][np_basic] is a great library to accomplish optimizations when dealing with numeric data. Similarly using [vectorized string methods][vectorization_str] is more efficient when dealing with string data in pandas. All these solutions are great but they don't work every time.

> Solution?

Keep a utility function in your code arsenal. I created a modified version of python's [`map()`][py_map] function. Using this one can apply any given function in parallel, on every element of a given iterable. That's not it. I can even see the progress using [`tqdm`][tqdm_git].

#### Pre-requisites
I'll be using the following packages/functions.
1. [multiprocessing.pool.Pool][mp_pool] - An object of this class controls a pool of worker processes to which jobs can be submitted.
2. [functools.partial][ft_partial] - To create a callable object with the same behavior as of the supplied function but some of the arguments "frozen".

```python
from traceback import format_exc
from tqdm import tqdm
from math import sqrt
from inspect import getfullargspec
from multiprocessing import Pool, cpu_count
from functools import partial
from typing import Iterable
```

## Multiprocessing Map() function

```python
def mp_func(func, 
            data_arg_name_in_func: str, 
            data: Iterable, 
            *args, **kwargs):
    
    """
    
    Apply given function on each element of data in parallel and show progress bar..
    
    Input:
    ------
        func:                      python function to be applied
        data_arg_name_in_func:     argument name in `func` which requires data point from data iterable
        data:                      iterable with data points
        args:                      positional args which will be supplied to the given `func`
        kwargs:                    keyword args which will be supplied to the given `func`
    
    Output:
    -------
        results:                   a list of result with each value as a result of application of function `func` 
                                   on each point in the given `data` iterable
                                                               
    Example:
    -------
    To parallelize given `base_func` function: 
    
        >>> def base_func(value, sq=True):
                if sq: return value ** 2
                return value
        >>> data = [0, 1, 2, 3, 4]
        >>> results = mp_func(base_func, 'value', data, sq=True)
        >>> print(results)
        [0, 1, 4, 9, 16]
        >>> results = mp_func(base_func, 'value', data, sq=False)
        >>> print(results)
        [0, 1, 2, 3, 4]
    """
    
    func_args = getfullargspec(func).args
    total = len(data)
    
    # Sanity checks
    assert isinstance(data, Iterable), "data should be an iterable.."
    assert data_arg_name_in_func in func_args, f"{data_arg_name_in_func} is not an argument of {func.__name__} function that you provided.."
    assert total > 1, f"len(data) should be > 1, but is {len(data)}"
    assert len(args) + len(kwargs) + 1 == len(func_args), f"{len(args)} + {len(kwargs)} + 1 != {len(func_args)}\nCheck args func_args are {func_args}"
    
    # for better user experience
    cpus = cpu_count()
    print(f"CPUS: {cpus}")
    
    # Returns a new partial object which when called will behave 
    # like func called with the positional arguments args and keyword arguments kwargs.
    _new_func = partial(func, *args, **kwargs)
    results = None
    
    try:
        # sanity check
        sample = _new_func(**{data_arg_name_in_func: data[-1]})
        # actual processing
        chunksize = int(sqrt(total) * cpus / 2 )
        with Pool(cpus) as pool:
            results = list(tqdm(pool.imap(_new_func, data, chunksize=chunksize), total=total))
    except TypeError as te:
        print(f"Please make sure that args and kwargs provided are valid for {func.__name__} function..")
        print(format_exc())
    except Exception as e:
        print(format_exc())
    finally:
        print("Done..")
        
    return results
```


[vectorization_str]: https://pandas.pydata.org/pandas-docs/stable/user_guide/text.html#string-methods
[vectorization_guide]: https://engineering.upside.com/a-beginners-guide-to-optimizing-pandas-code-for-speed-c09ef2c6a4d6
[np_basic]: https://numpy.org/devdocs/user/quickstart.html
[py_map]: https://docs.python.org/3.7/library/functions.html#map
[tqdm_git]: https://github.com/tqdm/tqdm
[mp_pool]: https://docs.python.org/3.7/library/multiprocessing.html#multiprocessing.pool.Pool
[ft_partial]: https://docs.python.org/3.7/library/functools.html#functools.partial


[exit_from]: https://docs.python.org/3/library/stdtypes.html#contextmanager.__exit__
[with_statment]: https://docs.python.org/3/reference/compound_stmts.html#the-with-statement
[cm_dec]: https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager