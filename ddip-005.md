D4 Python function implementation
====================

Python function implementation
--------------------

`mandatory` `author:akhavr`

Python function implementation MUST `#r` tag this DDIP.

Python function definition event MUST follow the following format

```json
{
 [...].
 "entry": <mandatory, function name to be called>,
 "venv": <optional, list of required additional modules>
 "venv-ref": <optional, filename with required additional modules>
}
```

Only one of `venv` or `venv-ref` MAY be present.  If both are present,
the function definition MUST be deemed invalid.

`code-ref` is treated as a clonnable repository address

Sample python function definition that gets a single integer and
increments it by 1:

```json
{
 "entry": "main",
 "code": """\
def main(num)
  return num + 1
"""
}
```

Function that performs iterations SHOULD use `yield`:

```json
{
 "entry": "main",
 "code": """\
def main(num)
  for i in range(1):
    num += 1
    yield num
  return num
"""
}
```


Note: `venv-ref` makes sense only with `code-ref`

Q: should we return stdout?
A: No, there is no stdout

Q: how set a range of versions?

TBD: `traits`
- `traits`: json which interpretation depends on `url`.  For example,
  it may refer to a range of versions of language to be used:

```
{"traits": {"versions": "3.7+"},
}
```

Implementation TODO: try function import (eval?) in python

`docker run -i --rm --name python -v $(pwd):/app -w /app python:3 python test.py` produces stdout to stdout and stderr to stderr
