# New in Python 3.10

## New Features

##### 1. Parenthesized context manager
- It is now supported to use enclosing parentheses for continuing lines across multiple lines in context managers.
- This change stems from a much larger change that appeared with Python 3.9 — the new PEG-based parser.

```python
# Python 3.9
with open('output.log', 'rw') as fout, open('input.csv') as fin:
	fout.write(fin.read())

# Python 3.10
with (open('output.log', 'w') as fout, open('input.csv') as fin): 
	fout.write(fin.read())
```

##### 2. Better Error messages
- In Python 3.10 a more informative error is emitted.
- SyntaxError exceptions raised by the interpreter will now highlight the full error range of the expression that constitutes the syntax error itself, instead of just where the problem is detected.

##### 3. _Switch Case_ / Structural pattern matching

- Inclusion of C like switch case statements.
- New keywords: `match` and `case` as soft keywords.
- Structural pattern matching allows us to perform the same match-case logic, but based on whether the structure of our comparison object matches a given pattern.

```python
match subject:
    case "pattern-1":
         action-1
    case "pattern-2":
         action-2
    case "pattern-3":
         action-3
    case _:              # Default case
		 action-default
```

- If an exact match is not confirmed, the last case, a wildcard \_, if provided, will be used as the matching case. If an exact match is not confirmed and a wildcard case does not exist, the entire match block is a no-op.
  
- You can combine several literals in a single pattern using `|` (“or”):
```python
case "pattern-1" | "pattern-2":
    action-1&2
```

- Check more [here](https://docs.python.org/3/whatsnew/3.10.html#pep-634-structural-pattern-matching).

[Related to Type Hints:]
##### 4. New Type Union Operator

- Using pipe character to specify the type of any attribute as opposed to ``Union`` in python3.9.

```python
# Python 3.9
def func(num: Union[int, float]) -> Union[int, float]:
	return num + 10

# Python 3.10
def func(num: int | float) -> int | float:
	return num + 10
```

##### 6. TypeAlias
- Added in typing module.
- Type aliases are user-specified types which may be as complex as any type hint, and are specified with a simple variable assignment on a module top level.
- Python 3.10 formalizes a way to explicitly declare an assignment as a type alias.
``
```python
from typing import TypeAlias

StrCache: TypeAlias = 'Cache[str]'  # a type alias

def func() -> StrCache: ...
```

##### 7. User-Defined Type Guards
- The typing module now includes `TypeGuard` to annotate type guard functions and enhance the information available to static type checkers for type narrowing.
- `TypeGuard` can be used to annotate the return type of a user-defined type guard function.
- When using user-defined boolean function as a type guard. Such a function should use TypeGuard[...] as its return type to alert static type checkers to this intention.
``
```python
from typing import TypeGuard


def is_str_list(val: list[object]) -> TypeGuard[list[str]]:
    '''Determines whether all objects in the list are strings'''
    return all(isinstance(x, str) for x in val)

def func(val: list[object]):
    if is_str_list(val):
        # Type of ``val`` is narrowed to ``list[str]``.
        print(" ".join(val))
    else:
        # Type of ``val`` remains as ``list[object]``.
        print("Not a list of strings!")
```

- the form `def is_str_list(val: list[object]) -> TypeGuard[list[str]]: ...`, means that if `is_str_list(val)` returns `True`, then `val` narrows from `list[object]` to `list[str]`.


___

## Other Language Changes

1. The `int` type has a new method `int.bit_count()`, returning the number of ones in the binary expansion of a given integer.
   
2. Strict Zipping
	- The zip() function now has an optional strict flag, used to require that all the iterables have an equal length.
	- With `zip(..., strict=True)`, 
	  Unlike the default behavior, it raises a ValueError if one iterable is exhausted before the others.
```python
for item in zip(range(3), ['fee', 'fi', 'fo', 'fum'], strict=True):  
	print(item)
...
(0, 'fee')
(1, 'fi')
(2, 'fo')
Traceback (most recent call last):
  ...
ValueError: zip() argument 2 is longer than argument 1
```

3. Two new builtin functions – aiter() and anext() have been added to provide asynchronous counterparts to iter() and next(), respectively. 

4. Static methods (@staticmethod) and class methods (@classmethod) now inherit the method attributes (__module__, __name__, __qualname__, __doc__, __annotations__) and have a new __wrapped__ attribute. 
	- Moreover, static methods are now callable as regular functions. 
```python
class Trade():
    
    @staticmethod
    def calculate_fee(trade_value):
        return trade_value * 0.001

    reference = {1: calculate_fee}

Trade.reference[1](2718281)

# Python 3.9 -> 
# TypeError: 'staticmethod' object is not callable

# Python 3.10 ->
# 2718.281
```

5. Hashes of NaN values of both `float` type and `decimal.Decimal` type now depend on object identity.
	- Before python 3.10, they always hashed to 0, despite NaN values not being equal to each other. This caused potentially quadratic runtime behavior due to excessive hash collisions when creating dictionaries and sets containing multiple NaNs.
	- Output in Python 3.9
```python
f1 = float("nan")
f2 = float("nan")

print(f1, f2)                  # nan nan
print(f1 == f2)                # False
print(hash(f1))                # 0
print(hash(f2))                # 0
print(hash(f1) == hash(f2))    # True
```
- Output in python 3.10
```python
f1 = float("nan")
f2 = float("nan")

print(f1, f2)                  # nan nan
print(f1 == f2)                # False
print(hash(f1))                # 305075651
print(hash(f2))                # 276643293
print(hash(f1) == hash(f2))    # False
```
- Now, hash of nan depends on the object identity. 

6. Class and module objects now lazy-create empty annotations dicts on demand. The annotations dicts are stored in the object’s `__dict__` for backwards compatibility. This improves the best practices for working with `__annotations__`
	- Classes don't have to define type hints (using `__annotations__`) and can inherit attributes from their parents. This means that if you try to access the `__annotations__` attribute of a class, you might accidentally get the type hints dictionary from its base class instead of the ones defined in the current class itself.
```python
class Base:
    a: int = 3
    b: str = 'abc'

class Derived(Base):
    pass

print(Derived.__annotations__)
# Python 3.9 -> 
# {'a': 'int', 'b': 'str'}

# Python 3.10
# {}
```
- In python 3.9, This will print the annotations dict from `Base`, not `Derived`.
- In python 3.10, Class and module objects now lazy-create empty annotations dicts on demand.

___
## Optimizations

1. Constructors [`str()`](https://docs.python.org/3/library/stdtypes.html#str "str"), [`bytes()`](https://docs.python.org/3/library/stdtypes.html#bytes "bytes") and [`bytearray()`](https://docs.python.org/3/library/stdtypes.html#bytearray "bytearray") are now faster (around 30–40% for small objects)

2. When building Python with [`--enable-optimizations`](https://docs.python.org/3/using/configure.html#cmdoption-enable-optimizations) now `-fno-semantic-interposition` is added to both the compile and link line. This speeds builds of the Python interpreter created with [`--enable-shared`](https://docs.python.org/3/using/configure.html#cmdoption-enable-shared) with `gcc` by up to 30%. See [this article](https://developers.redhat.com/blog/2020/06/25/red-hat-enterprise-linux-8-2-brings-faster-python-3-8-run-speeds/) for more details.

3. When using stringized annotations, annotations dicts for functions are no longer created when the function is created. 
	   - Instead, they are stored in a compact form (tuple of strings), and the function object lazily converts this into the annotations dict on demand (when  `func.__annotation__` is accessed). 
	   - This optimization cuts the CPU time needed to define an annotated function by half.

4. The [`runpy`](https://docs.python.org/3/library/runpy.html#module-runpy "runpy: Locate and run Python modules without importing them first.") module (used to implement the [`-m`](https://docs.python.org/3/using/cmdline.html#cmdoption-m) command line switch) now imports fewer modules. The `python3 -m module-name` command startup time is 1.4x faster in average.

5. Substring search functions such as `str1 in str2` and `str2.find(str1)` now sometimes use Crochemore & Perrin’s “Two-Way” string searching algorithm to avoid quadratic behavior on long strings. (more details [here](https://github.com/python/cpython/issues/86138))

___
## Deprecated

1. Deprecation warning is raised if the numeric literal is immediately followed by one of keywords [`and`](https://docs.python.org/3/reference/expressions.html#and), [`else`](https://docs.python.org/3/reference/compound_stmts.html#else), [`for`](https://docs.python.org/3/reference/compound_stmts.html#for), [`if`](https://docs.python.org/3/reference/compound_stmts.html#if), [`in`](https://docs.python.org/3/reference/expressions.html#in), [`is`](https://docs.python.org/3/reference/expressions.html#is) and [`or`](https://docs.python.org/3/reference/expressions.html#or).
   - Example, the below code is valid syntax prior python 3.10:
```python
0in x
1or x
0if 1else 2
[0x1for x in y]
```
   - In future releases it will be changed to syntax warning, and finally to syntax error.

2. Starting with 3.10, efforts will be made to clean up old import semantics kept for Python 2.7 compatibility. Functions and attributes like `find_loader()`, `find_module()`, `load_module()`, `module_repr()`, `__package__`, `__loader__`, and `__cached__` will slowly be removed
   - [`ImportWarning`](https://docs.python.org/3/library/exceptions.html#ImportWarning "ImportWarning") and/or [`DeprecationWarning`](https://docs.python.org/3/library/exceptions.html#DeprecationWarning "DeprecationWarning") will be raised as appropriate to help identify code which needs updating during this transition.

3. The entire `distutils` namespace is deprecated, to be removed in Python 3.12.

4. Non-integer arguments to [`random.randrange()`](https://docs.python.org/3/library/random.html#random.randrange "random.randrange") are deprecated. The [`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError "ValueError") is deprecated in favor of a [`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError "TypeError")

5. The following `threading` methods are now deprecated:
	- `threading.currentThread` => [`threading.current_thread()`](https://docs.python.org/3/library/threading.html#threading.current_thread "threading.current_thread")
	- `threading.activeCount` => [`threading.active_count()`](https://docs.python.org/3/library/threading.html#threading.active_count "threading.active_count")
	- `threading.Condition.notifyAll` => [`threading.Condition.notify_all()`](https://docs.python.org/3/library/threading.html#threading.Condition.notify_all "threading.Condition.notify_all")
	- `threading.Event.isSet` => [`threading.Event.is_set()`](https://docs.python.org/3/library/threading.html#threading.Event.is_set "threading.Event.is_set")
	- `threading.Thread.setName` => [`threading.Thread.name`](https://docs.python.org/3/library/threading.html#threading.Thread.name "threading.Thread.name")
	- `threading.thread.getName` => [`threading.Thread.name`](https://docs.python.org/3/library/threading.html#threading.Thread.name "threading.Thread.name")
	- `threading.Thread.isDaemon` => [`threading.Thread.daemon`](https://docs.python.org/3/library/threading.html#threading.Thread.daemon "threading.Thread.daemon")
	- `threading.Thread.setDaemon` => [`threading.Thread.daemon`](https://docs.python.org/3/library/threading.html#threading.Thread.daemon "threading.Thread.daemon")

6. `pathlib.Path.link_to()` is deprecated and slated for removal in Python 3.12. Use [`pathlib.Path.hardlink_to()`](https://docs.python.org/3/library/pathlib.html#pathlib.Path.hardlink_to "pathlib.Path.hardlink_to") instead.

7. `cgi.log()` is deprecated and slated for removal in Python 3.12.

8. Importing from the `typing.io` and `typing.re` submodules will now emit `DeprecationWarning`. These submodules will be removed in a future version of Python. Anything belonging to these submodules should be imported directly from [`typing`](https://docs.python.org/3/library/typing.html#module-typing "typing: Support for type hints (see :pep:`484`).") instead.

[ImportLib]

9. The various `load_module()` methods of [`importlib`](https://docs.python.org/3/library/importlib.html#module-importlib "importlib: The implementation of the import machinery.") have been documented as deprecated since Python 3.6, but will now also trigger a [`DeprecationWarning`](https://docs.python.org/3/library/exceptions.html#DeprecationWarning "DeprecationWarning"). Use [`exec_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module") instead.

10. The use of [`load_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.load_module "importlib.abc.Loader.load_module") by the import system now triggers an [`ImportWarning`](https://docs.python.org/3/library/exceptions.html#ImportWarning "ImportWarning") as [`exec_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module") is preferred.

11.  The import system now triggers an ImportWarning when using `importlib.abc.MetaPathFinder.find_module()` and `importlib.abc.PathEntryFinder.find_module()`, as the preferred methods are respective modules `find_spec()`. To aid in porting, you can use [`importlib.util.spec_from_loader()`](https://docs.python.org/3/library/importlib.html#importlib.util.spec_from_loader).

12. The import system now triggers an ImportWarning when using `importlib.abc.PathEntryFinder.find_loader()`, as the preferred methods is `importlib.abc.PathEntryFinder.find_spec()`. To aid in porting, you can use [`importlib.util.spec_from_loader()`](https://docs.python.org/3/library/importlib.html#importlib.util.spec_from_loader).

13. `importlib.abc.Finder` is deprecated (including its sole method, `find_module()`). Both [`importlib.abc.MetaPathFinder`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder "importlib.abc.MetaPathFinder") and [`importlib.abc.PathEntryFinder`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder "importlib.abc.PathEntryFinder") no longer inherit from the class. Users should inherit from one of these two classes as appropriate instead.

14. `importlib.abc.Loader.module_repr()`, `importlib.machinery.FrozenLoader.module_repr()`, and `importlib.machinery.BuiltinLoader.module_repr()` are deprecated and slated for removal in Python 3.12.

## Removed

1. Remove deprecated aliases to [Collections Abstract Base Classes](https://docs.python.org/3/library/collections.abc.html#collections-abstract-base-classes) from the [`collections`](https://docs.python.org/3/library/collections.html#module-collections "collections: Container datatypes") module.

2. The `loop` parameter has been removed from most of [`asyncio`](https://docs.python.org/3/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O.")‘s [high-level API](https://docs.python.org/3/library/asyncio-api-index.html) following deprecation in Python 3.8. (Check [here](https://docs.python.org/3/whatsnew/3.10.html#optimizations:~:text=motivation%20behind%20this%20change) for more details)

