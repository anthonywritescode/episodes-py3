```rawhtml
<img src="build/python-logo.png" style="float: left">
```
## python3
### file io

***

## outline

- what is meant by `text` / `binary` io
- python 2 behavior
- python 3 behavior
- code samples for 2+3 compatibility

***

## what is io?

- files on disk are a series of 1s and 0s
- for simplicity, we'll think of them as a series of bytes
- we'll also consider stdin and stdout to be special "files"

***

## what is io?

- for textual data, these bytes have a specific encoding
- a character (codepoint) is represented as a specific series of byte(s)

![encoding graphic](https://github.com/anthonywritescode/episodes-py3/raw/main/assets/encoding.png)

***

## what is io?

- a **binary** io procedure takes in binary data (`bytes` / `bytearray`)
- this data is written to the file _unmodified_
- the bytes in your program are represented identically on disk

***

## what is io?

- a **text** io procedure takes in text data (py2 `unicode`, py3 `str`)
- this data is first _encoded_ through an encoding (such as UTF-8)
- then this encoded data is written to the file
- the text in your program is represented by encoded bytes on disk

***

## python 2 - `open`

- files opened with `open` will always be in binary mode
- using `'wb'` or `'rb'` as the mode is superfluous (but good for documenting
  intent!)
- as with most py2 apis, `unicode` may be implicitly converted by the
  `ASCII` codec

***

## python 2 - `open`

```pycon
>>> with open('f.txt', 'w') as f:
...     f.write(b'hello world!')
...     f.write(u'unicode ascii text')
...
>>> with open('f.txt') as f:
...     print(type(f.read()) is bytes)
...
True
>>> with open('f.txt', 'w') as f:
...     f.write(u'☃')
...
UnicodeEncodeError: 'ascii' codec can't encode character u'\u2603' in position 0: ordinal not in range(128)
```

***

## python 2 - `open`

- you _can_ convert these into pseudo-text io objects with the C api
- `PyFile_SetEncoding` / `PyFile_SetEncodingAndErrors`
- let's look at some special streams which do just that!

***

## python 2 - `stdout` / `stderr` / `print`

- `stdout` / `stderr` are just `file` objects too
- on interpreter startup, `PyFile_SetEncodingAndErrors` is called
- **but** it is only called when the streams are `tty`s

***

## python 2 - `stdout` / `stderr` / `print`

```pycon
>>> import sys
>>> sys.stdout.write(u'☃\n')
☃
>>> sys.stderr.write(u'☃\n')
☃
>>> print(u'☃')
☃
```

***

## python 2 - `stdout` / `stderr` / `print`

- writing text is subject to environment variables

```console
$ LANG=C python -c 'print(u"\u2603")'
Traceback (most recent call last):
  File "<string>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode character u'\u2603' in position 0: ordinal not in range(128)
```

***

## python 2 - `stdout` / `stderr` / `print`

- writing text fails if piped (non-tty)

```console
$ LANG=en_US.UTF-8 python -c 'print(u"\u2603")'
☃
$ LANG=en_US.UTF-8 python -c 'print(u"\u2603")' | cat
Traceback (most recent call last):
  File "<string>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode character u'\u2603' in position 0: ordinal not in range(128)
```

***

## python 2 - `stdout` / `stderr` / `print`

- if environment may vary *or*
- if you might not have a tty
    - the only reliable way to use `stdout` / `stderr` / `print` is with bytes

***

## python 2 - `cStringIO` / `StringIO`

- `cStringIO.StringIO` - a binary stream (similar to python2 open)
- `StringIO.StringIO`
    - allows mixed mode if the `ASCII` encoding can convert
    - produces text if any text inputs
    - produces bytes otherwise
    - (much slower than `cStringIO` as it is implemented in pure python)

***

## python 2 - `cStringIO` / `StringIO`

```pycon
>>> x = StringIO.StringIO()
>>> x.write(b'\xe2\x98\x83')
>>> x.write(u'hi')
>>> x.getvalue()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python2.7/StringIO.py", line 271, in getvalue
    self.buf += ''.join(self.buflist)
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 0: ordinal not in range(128)
```

***

## python 3 - `open`

- just going to look at the `io` module
- `open` in python3 is just `io.open`
- `io.open(..., 'rb')` or `io.open(..., 'wb')` produce a binary io object
    - this object *requires* bytes
- `io.open` otherwise returns an `io.TextIOWrapper`
    - this object *requires* text

***

## python 3 - `open`

```pycon
>>> with open('f', 'wb') as f:
...     f.write('hi')
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
TypeError: a bytes-like object is required, not 'str'
```

***

## python 3 - `open`

```pycon
>>> with open('f', 'w') as f:
...     f.write(b'hi')
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
TypeError: write() argument must be str, not bytes
```

***

## python 3 - `open`

- The `encoding=` keyword argument may be passed to change what encoding
  the `TextIOWrapper` uses to write
- If `encoding=` is not passed, the encoding is determined using
  `locale.getpreferredencoding()`
    - This *usually* means looking at the `LANG` environment variable

***

## python 3 - `stdout` / `stderr`

- `stdout` / `stderr` are `TextIOWrapper`s
- write text using normal methods, write binary via `.buffer` binary stream

***

## python 3 - `print`

- `print` in python3 will write as if writing to a text stream
- Will convert by calling `str()` on arguments if necessary

```pycon
>>> print(b'foo')
b'foo'
>>> print('hi')
hi
```

***

## python 3 - `stdout` / `stderr` / `print`

- writing text is subject to environment variables

```console
$ LANG=C python3 -c 'print("\u2603")'
Traceback (most recent call last):
  File "<string>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode character '\u2603' in position 0: ordinal not in range(128)
```

***

## python 3 - stdio *gotcha*

- io in python 3 is *buffered by default*
- if your output may be piped, be sure to `.flush()` or
  `print(..., flush=True)` so it shows immediately

***

## python 3 - string io

- use `io.BytesIO` for a binary in memory file-like object
    - *requires* `bytes`
- use `io.StringIO` for a text in memory file-like object
    - *requires* text

***

## py2 + py3

- \o/ the `io` module is included in python2.6+
- replace `open` calls with `io.open`
- replace `cStringIO` / `StringIO` with either `io.BytesIO` or `io.StringIO`

***

## py2 + py3 - writing binary stdio

```python
if PY2:
    stdout_binary = sys.stdout
else:
    stdout_binary = sys.stdout.buffer

stdout_binary.write(b'\xe2\x98\x83\n')
```

***

## py2 + py3 - writing text stdio

- `io.TextIOWrapper` does not work with the stdio streams
- use `codecs.getwriter` instead!

```python
if PY2:
    stdout_text = codecs.getwriter(locale.getprefferedencoding())(sys.stdout)
else:
    stdout_text = sys.stdout

stdout_text.write('☃\n')
print('☃', file=stdout_text)
```

***

## py2 + py3 - writing in constrained environment

- earlier showed that io encoding is subject to `LANG`

```python
if PY2:
    stdout_text = codecs.getwriter('UTF-8')(sys.stdout)
else:
    stdout_text = io.TextIOWrapper(sys.stdout.buffer, encoding='UTF-8')

stdout_text.write('☃\n')
print('☃', file=stdout_text)
```
