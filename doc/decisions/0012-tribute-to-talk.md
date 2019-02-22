# 0012. Tribute-to-Talk

| Date | Tags |
|---|---|
| 2019-04-02 | spam filtering, blocking, chat |


## Status

proposed

## Context
Origin TtT design did not take into account future migration problems. E.g., how would users with a TtT v1 compliant client communicated with users having a TtT v2 setting?

## Decision

Instead of storing tribute information on-chain, opt for storing all TtT-related data on IPFS, and store content hashes on-chain instead. This will allow for simpler versioning in the future.

### Implementation notes
It is also important to check the presence of `:contact/added` tag.

Consider the scenario where contact A wants to send a message to contact B. 

Each contact has a list of `:system-tags` attached. TtT-specific ones are:
  - `:tribute-to-talk/paid`
  - `:tribute-to-talk/received`
These are symmetrical, so contact A might have a `paid` tag, and contact B might have a `received` tag.
In this case system needs to check where B has tribute-to-talk set, and if so, block sending messages until tribute has been paid. 

1. Check the TtT setting.
We check the TtT setting in 2 places - when preloading chat data, and during navigation to the profile page. First the IPFS manifest will be fetched from the contract, and second, a map containing `:tribute` and `:message` keys will be fetched from IPFS. The value of `:tribute` will be assoc'ed to the contact map. If there is no TtT manifest, the value of `:tribute` will be set to 0. 
All relevant code is located in `status-im.tribute-to-talk.core` ns.

2. Paying the tribute.
User can pay the tribute from the chat screen. After the transaction is sent, we dispatch a `:tribute-to-talk.ui/tribute-transaction-sent` that will fetch the `tribute-tx-id` and assoc it to the contact B's map in app-db. 

3. Ensuring that messages are not delivered.
There are two aspects to this: not sending outgoing messages from contact A's device, and dropping incoming messages on contact B's device if they do not satisfy whitelisting conditions.
  - outgoing messages: block sending messages from chat view until contact's tribute transaction fetched from wallet has 12 confirmations
  - incoming messages: in Message/receive protocol method, check if tribute-tx-id delivered in message payload refers to a valid tx that has at least 12 confirmations
 
## Consequences

Expand on tradeoffs and other side-effects of this decision.
