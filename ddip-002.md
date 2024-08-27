D4 Function definition
====================

Function definition event
--------------------

`mandatory` `author:akhavr`

Function definition event is a nostr event.  It MUST have following
structure

```json
{
 "id": <event nostr id>,
 "pubkey": <nostr pubkey>,
 "created_at": <nostr timestamp in seconds>,
 "kind": 1,
 "tags": [
  ["r", <url of the webpage with programming language definition>,
   # there MAY be more than one "r" tag referencing programming language
  ]
 ],
 "content": <serialized json with the function definition>,
 "sig": <nostr sig>,
}
```

Function definition MUST have following format:

```json
{
 "traits": <optional, list of webpage-dependend traits, like language version>,
 "code": <optional, the actual code implementing the contract>,
 "code-ref": <optional, URI reference (RFC 3986) of the code, implementing the contract>,
 "version": "0.1.0",
}
```

Only one of `code` and `code-ref` MAY be present.  If both are
present, the function definition event MUST be deemed invalid.

If not `code` nor `code-ref` are present, function definition is
interpreted as "private" and workers can receive it only off-band.
Obvious usage of private function is a signing primitive: the only
worker is connected to the signing system.  Other usage is a webapi
call to an external service: the api secret is shared off-band.

Function definition MAY contain additional fields that depend on the
language definition.  Possible additioinal fields are defined in this
and other DDIPs.

`#r` tags of language definitions MUST be interpreted as first match,
so put more specific definition first.

Textual contract MUST reference url with their definition.  For
example "negotiate price" contract will refer canonical page with
[DDIP-001](./ddip-001.md)

If `code-ref` is used, it MUST be present in `#r` tags for ease of
search and subscription.

Interpretation of `traits` depends on the `#r`-tagged language.

## Function implementation

D4 worker implementation MUST deserialize `args` into a dictionary and
pass it to the code in a language-specific manner.

Exact requirements for the implementations are defined in separate DDIPs.

## Iterations

Based on supplied max iteration cost and max total cost, worker
implementation MUST count the max number of iterations and do the
counting.

Worker MUST NOT expect to get paid more than `max_total_cost`.

Meaning of iteration is function and language-dependent.  Caller
better be aware of that.

## Function definition discovery

To discover new contracts potentially runnable by the worker, the
worker MIGHT subscribe to a specific `#r` language tag.

It's up to worker to determine it's pricing.

To facilitate function discovery, function definition MAY have
`description` field.  If there's one, it SHALL contain plain english
description of the nature of the function and parameters (like python
or java docstring)

## Authentication and rate control

Workers reply to call requests at their discretion.  They MAY
implement an ACL and reply only to specific clients.

To prevent flood, workers MAY implement a rate control and drop call
requests at their discretion.
