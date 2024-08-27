Encryption and anonymity
--------------------

This is not an extensive guide, but only an overview of options.
Don't consider this as an up-to-date reference, but just as a start
for your own implementation.

## Private function

Simplest implementation is the private function, when the private data
is shared with worker off-band and only the metadata about call is
leaked.

## Trusted relay

If your client is ok with trusted relay and trusted workers, this is
far the simplest setup.

## Trusted workers

Clients that want to sent function call requests using public
untrusted relays to a trusted set of workers are advised to use
[NIP-17 Private Direct
Messages](https://github.com/nostr-protocol/nips/blob/master/17.md) to
establish a private DM chat with this group of workers.  This provides
deniability about call metadata.

## Single or a group of workers with ephemeral key

If client is ok with leaking metadata of it's call but not it's
content, it MAY add function parameter `encryption` with the value
`ephemeral`.  In this case worker SHOULD contact the client with a
[NIP-17 Private
DM](https://github.com/nostr-protocol/nips/blob/master/17.md) with the
following json as the message tags:

```json
{
 [...],
 "tags": [
   ["e": <function call request eventid>]
 ],
 [...]
}
```

```json
{
 "encryption": "get-clear",
}
```

The client is MAY, depending on its security policy, answer using same
[NIP-17 Private
DM](https://github.com/nostr-protocol/nips/blob/master/17.md) protocol
with the cleartext function args, wrapped in Private DMs.

The worker SHOULD NOT expect a DM with function args within any
reasonable timeframe.

## Computation over encrypted data

If function supports [secure multiparty
computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation)
then client MAY use `encryption` field set to `mpc` to hint that the
computation will be done over the encrypted data.

## General guidelines

Each implementation is welcome to use any combination of
aforementioned methods to efficiently cover it's attach surface.  New
methods are welcome via separate DDIPs.

## Reading

- [NIP-17 Private Direct Messages](https://github.com/nostr-protocol/nips/blob/master/17.md)
- [NIP-44 Encrypted Payloads (Versioned)](https://github.com/nostr-protocol/nips/blob/master/44.md)
- [NIP-59 Gift Wrap](https://github.com/nostr-protocol/nips/blob/master/59.md)
- [NIP-29 Relay-based Groups](https://github.com/nostr-protocol/nips/blob/master/29.md)
- [Secure Multiparty
Computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation)
