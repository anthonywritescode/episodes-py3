```rawhtml
<img src="build/python-logo.png" style="float: left">
```
## python3
### backports

***

## outline

- what is a backport?
- using backports in code
- conditionally depending on libraries in `setup.py`

***

## what is a backport?

- new modules, functions, parameters, etc. are added to the standard library
  all the time
- a backport packages up the new features into a pypi package
- this package can be installable in older versions of python

***

## backports in code
#### whole modules

- some examples: `configparser`, `enum34` (`enum`), `argparse` (for py26)

```python
# The easiest!
# just import the new name
import configparser

parser = configparser.ConfigParser()
```

***

## backports in code
#### single functions

- some examples: `backports.shutil_get_terminal_size`,
  `backports.shutil_which`
- often single function additions to modules in stdlib

```python
if PY2:  # or sys.version_info < (3, 3) if you need to support python3.2
    from backports.shutil_get_terminal_size import get_terminal_size
else:
    from shutil import get_terminal_size
```

***

## backports in code
#### renamed modules

- some examples: `contextlib2`, `functools32`, `mock`
- often existing modules or namespaced modules which cannot be shadowed

```python
if PY2:
    import contextlib2 as contextlib
else:
    import contextlib

# or
if PY2:
    from contextlib2 import ExitStack
else:
    from contextlib import ExitStack
```

***

## writing setup.py

- it's undesirable to install unneeded packages
- as such, many of the backport packages (such as `functools32`) intentionally
  error when installed in python 3
- as such it's necessary to _conditionally_ require them in setup.py

***

## writing setup.py

- setup.py is arbitrary python, why not just check the version?
- don't do this (see next slide):

```python
import sys

from setuptools import setup

if sys.version_info < (3,):
    install_requires = ['functools32']
else:
    install_requires = []

setup(
    ...,
    install_requires=install_requires,
)
```

***

## writing setup.py

- `wheel`s are pre-built installations of packages
- essentially "run `setup.py`" and `zip` the result
- Let's see what happens when we build a wheel of the example setup.py

***

## writing setup.py

```console
# Make a wheel with a python 2 interpreter
$ ./venv27/bin/pip wheel . --no-deps

...

# Try and install with a python 3 interpreter
$ ./venv3/bin/pip install test_pkg-0.0.0-py2.py3-none-any.whl
Processing ./test_pkg-0.0.0-py2.py3-none-any.whl
Collecting functools32 (from test-pkg==0.0.0)
  Using cached functools32-3.2.3-2.zip
    Complete output from command python setup.py egg_info:
    This backport is for Python 2.7 only.

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-0bughj6v/functools32/
```

***

## writing setup.py
#### why did this happen?

- *problem*: the setuptools metadata is written **once**: at build time

    ```console
    $ unzip *.whl
    $ grep ^Requires test_pkg-0.0.0.dist-info/METADATA
    Requires-Dist: functools32
    ```

- *solution*: use _environment markers_

***

## writing setup.py

```python
setup(
    ...
    extras_require={
        ':python_version=="2.7"': ['functools32'],
    },
)
```
- _note: newer versions of pip support more operators than `==`_
- See [PEP 508](https://www.python.org/dev/peps/pep-0508/#id23) for a full
  list of supported markers.
- Other markers make other conditional dependence easy too (for example,
  `pypy` specific packages can use `:platform_python_implementation=="PyPy"`)

***

## writing setup.py

```console
$ ./venv27/bin/pip wheel . --no-deps

...

$ ./venv3/bin/pip install test_pkg-0.0.0-py2.py3-none-any.whl
Processing ./test_pkg-0.0.0-py2.py3-none-any.whl
Installing collected packages: test-pkg
Successfully installed test-pkg-0.0.0
```

**~\*~success!~\*~**
