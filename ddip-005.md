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
