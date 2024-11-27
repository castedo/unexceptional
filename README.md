<!-- copybreak off -->
Unexceptional Exceptions
========================

Docstring Introduction
----------------------

Unexceptional exceptions are Exception objects that are returned as "normal" error
objects, with return type hints, for "normal" non-try-except execution flow in some,
but not necessarily all, layers of a Python application.

With return type hints, type checkers, such as Mypy, can help spot when unexceptional
exceptions are not handled properly.


Discussion
----------

For a nice discussion on returning error objects versus raising exceptions, read
[Raising exceptions or returning error objects in Python by Luke Plant](
https://lukeplant.me.uk/blog/posts/raising-exceptions-or-returning-error-objects-in-python/).


FAQ
---

### Will Mypy do type checking for raised exceptions?

The short answer is no.
A Mypy maintainer has stated, "mypy is extremely unlikely to do this".
For more details, see:

* <https://github.com/python/mypy/issues/1773>
* <https://github.com/python/mypy/issues/7326>
* <https://github.com/python/mypy/issues/14900>

Nevertheless, by returning unexceptional Exception types, you **can** leverage
Mypy to help spot when unexceptional Exception types are not handled properly.


<!-- copybreak on -->

A `do_` Function Pattern
------------------------

A `do_` function pattern can ease the introduction of unexceptional exceptions.

When first writing a function, the natural and easy approach is to be Pythonic and not
return exceptions.
It can be useful to initially not worry about what errors or Exception types should be
part of a function's typed interface.
This often does not become clear until after implementation and some usage of the function.

If later it makes sense to have a typed function interface with returned error objects,
then a new `do_` function can complement the initial Pythonic exception-raising function.

### A Toy Example

Consider an initial Pythonic function with the declaration:

```python
def fancy_math(x: float) -> float:
```

Later, it becomes clear that `ValueError` is an unexceptional error case.
A `do_` function has the same parameters, the same name (but with `do_` prefixed),
and the same return type, but with all unexceptional exceptions appended to the
return type:

```python
def do_fancy_math(x: float) -> float | ValueError:
```

The implementation of `fancy_math` is ported to `do_fancy_math`
to return, rather than raise, unexceptional exceptions.
By using the `unexceptional` helper function, a stack trace is included,
even though the Exception object is being returned rather than raised.

The new implementation of `fancy_math` is a simple one-liner using the
`cast_or_raise` helper function.

```python
def fancy_math(x: float) -> float:
    return cast_or_raise(do_fancy_math(x))
```

Callers have a choice. They can be Pythonic and continue to call `fancy_math` with a
possible `ValueError` being raised, and may `try:`.

Or the caller can choose to *not* `try:` and instead call the `do_` function:

```python
y = do_fancy_math(x)
if isinstance(y, ValueError):
    ...
```

... so as to use the Force:

>     "Do. Or do not. There is no try."
>                               -- Yoda


More Info and Acknowledgements
------------------------------

* <https://stackoverflow.com/a/58821552/2420027>
