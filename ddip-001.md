Deadly Dumb Data Derivation Protocol (D4 protocol) - DDIP-001
====================

Basic protocol flow description
--------------------

`mandatory` `author:akhavr`

This DDIP defines the basic protocol that should be implemented by
every client and worker.

## Users

There are two types of users in D4 protocol

- client: invokes the remote procedure call (RPC)
- worker: fulfills the RPC and returns the result

Both of them are nostr users.

They MAY perform computations on free or paid basis.

Computation is defined by a function ("contract") event, published
("deployed") on any number of nost relays.  Computation code MAY be
open (available for any worker) or private (available off-band for a
selected set of workers).

Worker, willing to perform certain kind of function, SHALL subscribe
for nostr notifications for the function event.

Client, willing to call a function, SHALL reference function event in
`tags` field of the request.

RPC is specifically suited for iterative process (like autoregressive
LLM) and priced per-iteration (token for LLM) and has a price cap.

Worker SHALL NOT expect to be paid more than a client's price cap.
Worker SHALL NOT expect to be paid more than a client's price per
iteration.

RPC MAY be long running process, yielding results as computation
continues.  Each intermediate result is counted as an "iteration".

Client MAY expect a result from several workers over the same function
and data for more reliability.  Client indicates expected amount of
results in its call request.

## Call request

Call request is a nostr kind-1 event, sent by the client, which has
the following format:

```json
{
 "id": <event nostr id>,
 "pubkey": <client's nostr pubkey>,
 "created_at": <nostr timestamp in seconds>,
 "kind": 1,
 "tags": [
   ["e", <function event>"]
   ],
 "content": <serialized json of the function parameters>,
 "sig": <nostr sig>,
}
```

Function parameters MUST have following structure:

```json
{
 "max_iteration_cost": <positive integer, max price per iteration in msats>,
 "max_total_cost": <positive integer, max price per call>,
 "args": <dictionary with function call arguments,
 "results": 1,
 "version": "0.1.0"
}
```

Function parameters MAY have other fields, defined by other DDIPs.

After posting a function call event, client SHALL subscribe to
mentions of the event's id to get notified about the results.

If worker considers the computation not completed, it indicates it in
a result event (see below).

Client MAY wait for more results, than he stated in `results` field,
if it believes that workers, that provided answers, colluded.

Sample client request to perform LLM inference using quantized
mistral-7b-instruct via llama-cpp-python.  Note that `seed` is set by
the client for reproduceability.

```json
{
 "pubkey": "6e468422dfb74a5738702a8823b9b28168abab8655faacb6853cd0ee15deee93",
 "created_at": 1722611197,
 "kind": 1,
 "tags": [
   ["e", "bf2376e17ba4ec269d10fcc996a4746b451152be9031fa48e74553dde5526bce"]
	# at this address the "run llama-cpp-python" code is deployed
	],
 "content": """{
    "max_iteration_cost": 1000, # in milisatoshi
	"max_total_cost": 1000000, # in milisatoshi
	"results": 1,
	"version": "0.1.0",
	"args": {
	  "prompt": "<s>[INST] What is the capital of France? [/INST]",
	  "model": "mistral-7b-instruct-v0.1.Q4_K_M.gguf",
	  "temperature": 0.0,
	  "top_p": 0.95,
	  "top_k": 40,
	  "min_p": 0.05,
	  "typical_p": 1.0,
	  "stop": [],
	  "seed": 3283645831608526411,
	  "response_format": None,
      "max_tokens": None,
	  "presence_penalty": 0,
	  "frequency_penalty": 0,
	  "repeat_penalty": 1.0,
	  "tfs_z": 1.0,
	}}"""
}
```

Actual json on wire would be like, to compress the `content` field

```json
{
 "pubkey": "6e468422dfb74a5738702a8823b9b28168abab8655faacb6853cd0ee15deee93",
 "created_at": 1722611197,
 "kind": 1,
 "tags": [
   ["e", "bf2376e17ba4ec269d10fcc996a4746b451152be9031fa48e74553dde5526bce"]
	# at this address the "run llama-cpp-python" code is deployed
	],
 "content": """{"max_iteration_cost": 1000, "max_total_cost": 1000000, "results": 1, "version": "0.1.0", "args": {"prompt": "<s>[INST] What is the capital of France? [/INST]", "model": "mistral-7b-instruct-v0.1.Q4_K_M.gguf", "temperature": 0.0, "top_p": 0.95, "top_k": 40, "min_p": 0.05, "typical_p": 1.0, "stop": [], "seed": 3283645831608526411, "response_format": null, "max_tokens": null, "presence_penalty": 0, "frequency_penalty": 0, "repeat_penalty": 1.0, "tfs_z": 1.0}}"""
}
```

Event id for such json is `5c8fad559a474284f830751265d415e4872bd4ce51db4d56af00caa366961cc1'`

If function expects more parameters, or some parameters don't fulfull
preconditions, worker MAY fail silently or MAY return an error.

## Result response

Result response is a nostr kind-1 event, sent by worker, with the
following structure:

```json
{
 "id": <event nostr id>,
 "pubkey": <worker's nostr pubkey>,
 "created_at": <nostr timestamp in seconds>,
 "kind": 1,
 "tags"; [
   ["e", <function call event>"],
   ["e", <function definition event>],
 ],
 "content": <serialized json of the result content>,
 "sig": <nostr sig>,
}
```

Result content MUST have the following structure:

```json
{
 "result": <serialized json with the result>,
 "iteration_cost": 1000, # in milisatoshi
 "total_cost": 35000, # in milisatoshi
 "err": <optional, serialized json with the error>,
 "more_results": <true|false>,
 "version": "0.1.0"
}
```

There MAY be other fields, defined by respective DDIPs.

Sample worker response to the request above:

```json
{
 "pubkey": "7fffe526a93c7f613cb0c8bee456e961660bada21fa55aa7eeea16a92a59823b",
 "created_at": 1722612197,
 "kind": 1,
 "tags": [
   ["e", "bf2376e17ba4ec269d10fcc996a4746b451152be9031fa48e74553dde5526bce"],
	# at this address the "run llama-cpp-python" code is deployed
   ["e", "7a352156df9e6203da4df4264c87455e6c02abbc80c522701c31be19a9f28a2c"]
   # see the request eventid above
 ],
 "content": """{
   "iteration_cost": 1000, # in milisatoshi
   "total_cost": 35000, # in milisatoshi
   "result": """{
   	 "id": "chatcmpl-13e3781a-2728-4006-93d6-0cdec3ef6f69",
   	 "object": "chat.completion",
   	 "model": "mistral-7b-instruct-v0.1.Q4_K_M.gguf",
   	 "choices": [{"index": 0,
   	   "message": {"role": "assistant",
   	    "content": " The capital city of France is Paris."},
   	   "finish_reason": "stop"}],
   	 "usage": {"prompt_tokens": 27, "completion_tokens": 8, "total_tokens": 35},
   	 }""",
   "more_results": false,
   "version": "0.1.0"
}
```

## Result, indicating there are more to follow

It's hard to predict how long will take an iterative process, so
there's an additional flag `more_results` in the response.

Here's sample call that requests will result in a followup response:

```json
{
 "pubkey": "6e468422dfb74a5738702a8823b9b28168abab8655faacb6853cd0ee15deee93",
 "created_at": 1722611197,
 "kind": 1,
 "tags": [
   ["e", "bf2376e17ba4ec269d10fcc996a4746b451152be9031fa48e74553dde5526bce"]
	# at this address the "run llama-cpp-python" code is deployed
	],
 "content": """{
    "max_iteration_cost": 1000, # in milisatoshi
	"max_total_cost": 720000, # in milisatoshi
	"version": "0.1.0",
	"args": {
	  "prompt": "<s>[INST] Write a verse describing Putin as clown.\n [/INST]",
	  "model": "mistral-7b-instruct-v0.1.Q4_K_M.gguf",
	  "temperature": 0.0,
	  "top_p": 0.95,
	  "top_k": 40,
	  "min_p": 0.05,
	  "typical_p": 1.0,
	  "stop": [],
	  "seed": 3283645831608526411,
	  "response_format": None,
      "max_tokens": 41,
	  "presence_penalty": 0,
	  "frequency_penalty": 0,
	  "repeat_penalty": 1.0,
	  "tfs_z": 1.0,
	}}"""
}
```

And the response

```json
{
 "pubkey": "7fffe526a93c7f613cb0c8bee456e961660bada21fa55aa7eeea16a92a59823b",
 "created_at": 1722612197,
 "kind": 1,
 "tags": [
   ["e", "bf2376e17ba4ec269d10fcc996a4746b451152be9031fa48e74553dde5526bce"],
	# at this address the "run llama-cpp-python" code is deployed
   ["e", "7a352156df9e6203da4df4264c87455e6c02abbc80c522701c31be19a9f28a2c"]
   # see the request eventid above
 ],
 "content": """{
   "iteration_cost": 1000, # in milisatoshi
   "total_cost": 72000, # in milisatoshi
   "result": """{
   	 "id": "chatcmpl-13e3781a-2728-4006-93d6-0cdec3ef6f69",
   	 "object": "chat.completion",
   	 "model": "mistral-7b-instruct-v0.1.Q4_K_M.gguf",
   	 "choices": [{"index": 0,
   	   "message": {"role": "assistant",
   	   "content": " Putin, the clown of Russia's throne,\n"
                  "A jester with a heart of stone,\n"
                  "With a grin that's wide and cruel,\n"
                  "He rules with an iron rule.\n",},
   	   "finish_reason": "length"}],
   	 "usage": {"prompt_tokens": 30, "completion_tokens": 42, "total_tokens": 72},
   	 }""",
   "more_results": true,
   "version": "0.1.0",
}
```

`more_results: true` means that worker will produce more results if
given more funding with an updated function call.

## Price negotiation

If a worker is ready to perform the computation but disagrees with the
price, it may submit a responce to negotiate the price.  In this
kind-1 message it MUST have the following format:

```json
{
 "id": <event nostr id>,
 "pubkey": <worker's nostr pubkey>,
 "created_at": <nostr timestamp in seconds>,
 "kind": 1,
 "tags": [
   ["e", <function call event>],
   ["e", <id of negotiate price function>]
 ],
 "content": <serialized json with the price suggestion>
}
```

Price suggestion content MUST have the format:

```json
{
  "result": null,
  "iteration_cost": <integer iteration cost in msats>,
  "total_cost": <integer max cost in mstats>
  "more_results": false,
  "version": "0.1.0"
}
```

Sample worker answer with price alternative

```
{
 "pubkey": "7fffe526a93c7f613cb0c8bee456e961660bada21fa55aa7eeea16a92a59823b",
 "created_at": 1722612197,
 "kind": 1,
 "tags": [
   ["e", "321ddfdaba3a2637e79c9b2f7ec8ec2d9ce6cc0983c7de0c621c0c407a6daab7"],
   # sample eventid of the "negotiate price" function
   ["e", "7a352156df9e6203da4df4264c87455e6c02abbc80c522701c31be19a9f28a2c"]
   # see the request eventid above
 ],
 "content": """{
   "result": null,
   "iteration_cost": 2000, # msats
   "total_cost": 200000, # msats
   "more_results": false,
   "version": "0.1.0"
   }"""
}

```

If client accepts a higher negotiated price, it posts a new function
call event with a new id.  Client MAY refer the suggested price event
in e-tags.

## Protocol versioning

Deadly Dumb Data Derivation Protocol uses [Semantic
Versioning](https://semver.org/)

This DDIP describes version 0.1.x of Deadly Dumb Data Derivation
Protocol.

## Error handling

The worker MAY return an in `err` field of the result respose content.
Interpretation depends on the function definition (see
[DDIP-002](./ddip-002.m2)

## Result feedback

After the result is posted, the client MAY post the followup kind-1
event, refering the result response in `#e` tag.

```json
{
 "id": <event nostr id>,
 "pubkey": <worker's nostr pubkey>,
 "created_at": <nostr timestamp in seconds>,
 "kind": 1,
 "tags"; [
   ["e", <function result responce event>"],
   ["e", <function definition event>],
 ],
 "content": <serialized json of the result feedback>,
 "sig": <nostr sig>,
}
```

Result content MUST have the following structure:

```json
{
 "score": <integer from 0 to 9, 0 - completely unacceptable, 9 - perferctly useful>,
 "version": "0.1.0"
}
```

This may help collect feedback to finetune trainable functions like
language and diffusion models.

There MAY be other optional fields defined in other DDIPs.
