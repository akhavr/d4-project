Payments in D4 protocol
--------------------

This DDIP defines OPTIONAL field `payment` in the worker response that
defines payment expectations.

```json
{
[...]
 "version": "0.1.0",
 "payment": <payment protocol: string>
}
```

If no `payment` field is present, no payment is expected.  Worker then
MAY ignore `max_iteration_cost` and `max_total_cost` entries in this
case.

If a `payment` field is present in the worker result response, the
client MUST use payment protocol it to pay `total_cost` amount.

This DDIP defines [lnurl]((https://github.com/lnurl/luds), [lightning address](https://github.com/andrerfneves/lightning-address/) and 
[nip-57](https://github.com/nostr-protocol/nips/blob/master/57.md) payment methods:

- `lnurl`
- `lnaddr`
- `nip57`

Other payment protocols, for example defining less custodial payment
methods, are to be defined in separate DDIPs.  They MAY or MAY NOT
use the same balance logic.

Worker keeps a balance of the client.  There might be several workers,
sharing the same balance.

Balance policy is up to worker.  For example, the worker MAY allow the
balance of a particular client go negative.  The worker MAY execute a
function call if the balance is already negative.

Since worker balance is not intended for money storage, it MAY also
charge a storage fee.

Recomended approach is to fund worker balance, perform a call or
several ones, and withdraw it.

If the client has a positive balance at the worker, or the worker
allows balance to go negative, client discovers payment protocol and
payment details from result response.

If worker needs a prepayment, then the worker MUST submit a price
negotiation request with the payment data.

Worker's payment details MAY change anytime, so client SHOULD use the
latest supplied worker payment data.

Client SHOULD verify result and price negotiation event signature if
it contains `payment` field.

## LNURL payment protocol

For lnurl purposes the worker is "the service".

If worker signaled that he's willing to get paid with `lnurl.balance`
protocol, there MUST be extra field `lnurl` which contains
`payRequest` endpoint per [LUD-06: payRequest base
spec](https://github.com/lnurl/luds/blob/luds/06.md)

Then the payment MUST proceed using LUD-06 spec.

The client SHALL identify itself using [LUD-18: Payer identity in
payRequest protocol.](https://github.com/lnurl/luds/blob/luds/18.md)

The client MUST pay at least `max_total_cost` msats.

The worker MUST support at least [LUD-03: withdrawRequest base
spec](https://github.com/lnurl/luds/blob/luds/03.md)

## Lightning address payment protocol

If worker signaled that he's willing to get paid with `lnaddr.balance`
protocol, there MUST be an extra field `lnaddr` which contains [the
lightning
address](https://github.com/andrerfneves/lightning-address/blob/master/README.md)
in the form of `user@domain.com`

From this point on, the event flow is the same as with `lnurl` payment
protocol.

## NIP57 payment protocol

Event flow happens per [Nostr Lightning
Zaps](https://github.com/nostr-protocol/nips/blob/master/57.md)
specification.  Client MAY zap either worker pubkey or result response
or price negotiation event.

Withdrawal is not possible in this payment protocol.
