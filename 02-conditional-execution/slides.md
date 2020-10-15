```rawhtml
<img src="build/python-logo.png" style="float: left">
```
## python3
### conditional execution

***

### what?

- 2+3 codebase
- sometimes you need to run interpreter-version-specific code

***

### goals

- little or no runtime overhead
- maintainable

***

### how?

- **before you do anything** check to make sure `six` doesn't already shim
- make a `_compat` module for isolation

***

### do not: `import` detection


```python
try:
    import configparser
    PY2 = False
except ImportError:
    import ConfigParser as configparser
    PY2 = True
```

- backported modules (`python-future`, `configparser`, `enum34`, etc.)
- hides failure mode

***

### _properly_ detecting python 2

```python
# Suggested
from six import PY2

# Also ok
PY2 = sys.version_info[0] == 2

# Also works (unless you are using `futurize` stage 2)
PY2 = str is bytes
```

***

### do not: switch on `PY3`

```python
# or `from six import PY3`
PY3 = sys.version_info[0] == 3

if PY3:
    # ...
else:
    #  ...
```
- python 4 _is coming_!
- treat python 2 as "old" and everything else as "new"


***

### do not: runtime version checks

- As much as possible isolate version differences to import time
- Make _functions_ and swap implementations based on version

```python
# do NOT
def foo():
    if PY2:
        ...
    else:
        ...
# do
if PY2:
    def foo():
        # python 2,3 implementation
else:
    def foo():
        # python 3+ implementation
```

***
### full example

```python
from six import PY2

def to_text(s):
    return s if isinstance(s, six.text_type) else s.decode('UTF-8')

def to_bytes(s):
    return s if isinstance(s, bytes) else s.encode('UTF-8')

if PY2:  # pragma: no cover (PY2)
    to_native = to_bytes
else:  # pragma: no cover (PY3+)
    to_native = to_text
```
