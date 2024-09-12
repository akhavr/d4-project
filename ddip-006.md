D4 Docker function definition
====================

`mandatory` `author:akhavr`

Docker function definition
--------------------

Docker function definition event MUST `#r` tag this DDIP.

Docker function definition event MUST follow the following format

```json
{
 [...],
 "image": <mandatory, url of docker image>,
 "type": "httppost",
 "link": <mandatory, http path to trigger the call>,
 "port": <mandatory, http docker port to trigger the call>,
 "pre": <mandatory, nostr id of d4 function to preprocess params>,
 "post": <mandatory, nostr id of d4 function to postprocess results>,
 "stream": <optional integer, num of output bytes after which an intermediate result is posted>
}
```

Other types of parameter passing MAY be defined in other ddips.
Example might be `env`, `file`, `httpget`.

Also, other ddips may introduce other ways to trigger the execution,
for example executing a command wihtin the docker.

`pre` is necessary to compute the max number of iterations in the
format, suitable for the docker image.

`post` is necessary to convert actual number of iterations to the
form, suitable for nostr worker code to compute the actual cost.

`pre` and `post` functions are to be trusted, since there's no way
within the protocol to verify or guarantie that they will supply the
intended result.
