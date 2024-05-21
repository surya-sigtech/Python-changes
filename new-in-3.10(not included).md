# (Not covered) New in Python 3.10

**This doc contains all the changes that are not included in summary doc of Python 3.10, & contains:**
1. Libraries not directly related to framework
2. Updates in Internal libraries
3. Miscellaneous changes

## New Features

1. PEP 612: [Parameter Specification Variables](https://docs.python.org/3/whatsnew/3.10.html#pep-612-parameter-specification-variables "Link to this heading")
2. [Optional `EncodingWarning` and `encoding="locale"` option](https://docs.python.org/3/whatsnew/3.10.html#optional-encodingwarning-and-encoding-locale-option "Link to this heading")

## Other Language Changes ([link](https://docs.python.org/3/whatsnew/3.10.html#other-language-changes))

1. If object.__ipow__() returns NotImplemented, the operator will correctly fall back to object.__pow__() and object.__rpow__() as expected.
   
2. The views returned by dict.keys(), dict.values() and dict.items() now all have a mapping attribute that gives a types.MappingProxyType object wrapping the original dictionary. 
   
3. Builtin and extension functions that take integer arguments no longer accept Decimals, Fractions and other objects that can be converted to integers only with a loss (e.g. that have the __int__() method but do not have the __index__() method).
   
4. Assignment expressions can now be used unparenthesized within set literals and set comprehensions, as well as in sequence indexes (but not slices). (more details [here](https://bugs.python.org/issue40631))
   
5. Functions have a new `__builtins__` attribute which is used to look for builtin symbols when a function is executed, instead of looking into `__globals__['__builtins__']`. The attribute is initialized from `__globals__["__builtins__"]` if it exists, else from the current builtins.

[Annotations]

6. Annotations for complex targets (everything beside `simple name` targets defined by [**PEP 526**](https://peps.python.org/pep-0526/)) no longer cause any runtime effects with `from __future__ import annotations`.

7.  Annotations consist of `yield`, `yield from`, `await` or named expressions are now forbidden under `from __future__ import annotations` due to their side effects.

8. Usage of unbound variables, super() and other expressions that might alter the processing of symbol table as annotations are now rendered effectless under from __future__ import annotations.

[Error Related]

9. A [`SyntaxError`](https://docs.python.org/3/library/exceptions.html#SyntaxError "SyntaxError") (instead of a [`NameError`](https://docs.python.org/3/library/exceptions.html#NameError "NameError")) will be raised when deleting the [`__debug__`](https://docs.python.org/3/library/constants.html#debug__ "__debug__") constant.

10.  [`SyntaxError`](https://docs.python.org/3/library/exceptions.html#SyntaxError "SyntaxError") exceptions now have `end_lineno` and `end_offset` attributes. They will be `None` if not determined.

___
## Optimizations

1. The `LOAD_ATTR` instruction now uses new “per opcode cache” mechanism. It is about 36% faster now for regular attributes and 44% faster for slots.
   
2. When building Python with [`--enable-optimizations`](https://docs.python.org/3/using/configure.html#cmdoption-enable-optimizations) now `-fno-semantic-interposition` is added to both the compile and link line. 
   - This speeds builds of the Python interpreter created with [`--enable-shared`](https://docs.python.org/3/using/configure.html#cmdoption-enable-shared) with `gcc` by up to 30%. 
   - See [this article](https://developers.redhat.com/blog/2020/06/25/red-hat-enterprise-linux-8-2-brings-faster-python-3-8-run-speeds/) for more details.

3. Use a new output buffer management code for [`bz2`](https://docs.python.org/3/library/bz2.html#module-bz2 "bz2: Interfaces for bzip2 compression and decompression.") / [`lzma`](https://docs.python.org/3/library/lzma.html#module-lzma "lzma: A Python wrapper for the liblzma compression library.") / [`zlib`](https://docs.python.org/3/library/zlib.html#module-zlib "zlib: Low-level interface to compression and decompression routines compatible with gzip.") modules, and add `.readall()` function to `_compression.DecompressReader` class. bz2 decompression is now 1.09x ~ 1.17x faster, lzma decompression 1.20x ~ 1.32x faster, `GzipFile.read(-1)` 1.11x ~ 1.18x faster.
   - **In framework, `zlib` is used in `serde.py` & `ray.py`**

4.  Micro-optimizations to `_PyType_Lookup()` to improve type attribute cache lookup performance in the common case of cache hits. This makes the interpreter 1.04 times faster on average.

5. The following built-in functions now support the faster [**PEP 590**](https://peps.python.org/pep-0590/) vectorcall calling convention: [`map()`](https://docs.python.org/3/library/functions.html#map "map"), [`filter()`](https://docs.python.org/3/library/functions.html#filter "filter"), [`reversed()`](https://docs.python.org/3/library/functions.html#reversed "reversed"), [`bool()`](https://docs.python.org/3/library/functions.html#bool "bool") and [`float()`](https://docs.python.org/3/library/functions.html#float "float")

6. [`BZ2File`](https://docs.python.org/3/library/bz2.html#bz2.BZ2File "bz2.BZ2File") performance is improved by removing internal `RLock`. This makes `BZ2File` thread unsafe in the face of multiple simultaneous readers or writers, just like its equivalent classes in [`gzip`](https://docs.python.org/3/library/gzip.html#module-gzip "gzip: Interfaces for gzip compression and decompression using file objects.") and [`lzma`](https://docs.python.org/3/library/lzma.html#module-lzma "lzma: A Python wrapper for the liblzma compression library.") have always been.

___
## Deprecated

1. `zimport.zipimporter.load_module()` has been deprecated in preference for [`exec_module()`](https://docs.python.org/3/library/zipimport.html#zipimport.zipimporter.exec_module "zipimport.zipimporter.exec_module").

[ImportLib]

2. The various implementations of `importlib.abc.MetaPathFinder.find_module()` ( `importlib.machinery.BuiltinImporter.find_module()`, `importlib.machinery.FrozenImporter.find_module()`, `importlib.machinery.WindowsRegistryFinder.find_module()`, `importlib.machinery.PathFinder.find_module()`, `importlib.abc.MetaPathFinder.find_module()` ), `importlib.abc.PathEntryFinder.find_module()` ( `importlib.machinery.FileFinder.find_module()` ), and `importlib.abc.PathEntryFinder.find_loader()` ( `importlib.machinery.FileFinder.find_loader()` ) now raise [`DeprecationWarning`](https://docs.python.org/3/library/exceptions.html#DeprecationWarning "DeprecationWarning") and are slated for removal in Python 3.12

3. The deprecations of imp, `importlib.find_loader()`, `importlib.util.set_package_wrapper()`, `importlib.util.set_loader_wrapper()`, `importlib.util.module_for_loader()`, `pkgutil.ImpImporter`, and `pkgutil.ImpLoader` have all been updated to list Python 3.12 as the slated version of removal (they began raising DeprecationWarning in previous versions of Python).

4. The import system now uses the `__spec__` attribute on modules before falling back on `module_repr()` for a module’s `__repr__()` method.

[SQLite]

5. `sqlite3.OptimizedUnicode` has been deprecated, scheduled for removal in Python 3.12. As it has been undocumented and obsolete since Python 3.3

6. `sqlite3.enable_shared_cache` is now deprecated, scheduled for removal in Python 3.12. 
   - See [the SQLite3 docs](https://sqlite.org/c3ref/enable_shared_cache.html) for more details. 
   - If a shared cache must be used, open the database in URI mode using the `cache=shared` query parameter.

[Other]

7. [These](https://docs.python.org/3/whatsnew/3.10.html#optimizations:~:text=features%20have%20been%20deprecated) `ssl` features have been deprecated since Python 3.6, Python 3.7, or OpenSSL 1.1.0 and will be removed in 3.11.

8. The threading debug (`PYTHONTHREADDEBUG` environment variable) is deprecated in Python 3.10 and will be removed in Python 3.12.

___
## Removed

1. Removed special methods `__int__`, `__float__`, `__floordiv__`, `__mod__`, `__divmod__`, `__rfloordiv__`, `__rmod__` and `__rdivmod__` of the [`complex`](https://docs.python.org/3/library/functions.html#complex "complex") class. They always raised a [`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError "TypeError").
   
2. The `ParserBase.error()` method from the private and undocumented `_markupbase` module has been removed. [`html.parser.HTMLParser`](https://docs.python.org/3/library/html.parser.html#html.parser.HTMLParser "html.parser.HTMLParser") is the only subclass of `ParserBase` and its `error()` implementation was already removed in Python 3.5.
   
3. Removed the `unicodedata.ucnhash_CAPI` attribute which was an internal PyCapsule object. The related private `_PyUnicode_Name_CAPI` structure was moved to the internal C API.

4. Removed the `parser` module, which was deprecated in 3.9 due to the switch to the new PEG parser, as well as all the C source and header files that were only being used by the old parser, including `node.h`, `parser.h`, `graminit.h` and `grammar.h`.

5. Removed the Public C API functions `PyParser_SimpleParseStringFlags`, `PyParser_SimpleParseStringFlagsFilename`, `PyParser_SimpleParseFileFlags` and `PyNode_Compile` that were deprecated in 3.9 due to the switch to the new PEG parser.

6. Removed the `formatter` module, which was deprecated in Python 3.4. It is somewhat obsolete, little used, and not tested.

7. Removed the `PyModule_GetWarningsModule()` function that was useless now due to the `_warnings` module was converted to a builtin module in 2.6.
