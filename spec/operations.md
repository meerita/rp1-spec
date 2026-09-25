---
title: Operations
description: Where an opcode appears in a frame, the whole domain of the opcode space, the opcodes protocol version 0 assigns at this revision, the request and response payload of each operation, the argument encoding they share, the validation each runs, what a receiver does with an opcode this revision does not assign, and what a later revision does when it assigns one.
protocol_version: 0
revision: v0.4.0
status: draft
order: 7
---

# Operations

## Scope

This document defines the opcode space: where an opcode appears in a
frame, the whole domain of the field that carries it, the opcodes this
revision assigns, the request and response payload of each operation, the
argument encoding the operations share, the validation each runs, what a
receiver does with an opcode this revision does not assign, and what a
later revision does when it assigns one.

It assigns six opcodes: the handshake, and the five ungated operations
`PING`, `GET`, `SET`, `DEL` and `EXISTS`. Each of the five is reached by a
peer that offers and negotiates no capability. Handshake owns the handshake
request payload and the rule for when the handshake request is legal.

It defines no capability-gated operation, and this revision assigns no
capability. It does not define the result code a response carries, which
Results owns, nor the failure a request produces, which Failures owns.

An engineer implementing this document alone can run the five ungated
operations, encode each request, decode each response, and attribute each
answer to the request it retires.

## Opcodes

An opcode appears in the `code` field of a REQUEST frame. Frame Header
states that `code` is a `u16` and that its meaning follows the frame's
kind. No other frame kind this revision assigns carries an opcode.

The whole domain is covered here:

| Value | Operation | Capability |
|---|---|---|
| `0x0001` | the handshake | none |
| `0x0002` | `PING` | none |
| `0x0003` | `GET` | none |
| `0x0004` | `SET` | none |
| `0x0005` | `DEL` | none |
| `0x0006` | `EXISTS` | none |
| `0x0000` | reserved | |
| `0x0007..0xFFFF` | reserved | |

The handshake is owned by Handshake. Each of the five operations below it
is ungated: a peer reaches it without offering or negotiating any
capability, and the `Capability` column states `none` for every one.

`0x0000` is reserved on the same terms as every value but the six assigned,
and a receiver answers it by the rule below. No value of the field is
reserved as never valid.

A peer MUST NOT send a REQUEST frame whose `code` this revision does not
assign. A receiver that meets one MUST answer the unsupported operation
error class for that request and MUST keep the connection open. Reporting
a request-scoped failure states what each peer does to answer.

Frame Admission Order places this check at step 13.

A REQUEST frame carrying the handshake opcode is the handshake request.
Before the Handshake states that it is the only frame legal in the
pre-negotiation state, and it states the consequence of sending it at any
other time.

The sections below define the payload each of the five operations carries
in each direction and the validation it runs on its arguments.

## The REQUEST frame payload

Handshake defines the handshake request payload. Each of the five ungated
operations defines its request payload below, and all five share one
argument encoding.

### The key and the value

A key is an opaque byte string. A value is an opaque byte string. Neither
is required to be UTF-8, and neither is interpreted by the protocol. A key
and a value may each be empty: a key of zero bytes and a value of zero
bytes are each one value of the field. An empty key is a key, and a value
of zero bytes is a value; neither is the same as an absent key, which is a
result code and not a payload.

`PING` carries no key and no value, and its request payload is empty. `GET`,
`DEL` and `EXISTS` carry one key, and their request payload is the key bytes
alone: the key runs from the first byte of the payload to its last, so its
length is `payload length` and no length field is carried. A receiver MUST
read the whole payload as the key and MUST NOT require a length field for
it.

`SET` carries a key and a value:

```text
offset  size  field        type
     0     4  key length   u32
     4     n  key          bytes
   n+4     -  value        bytes
```

The value length is derived from the frame's `payload length`:

```text
value length = payload length - 4 - key length
```

`key length` is a `u32`, so a key of any length the frame can carry is
expressible in it. The value length is the rest of the payload after the
key, so a `SET` whose key length leaves no byte carries a value of zero
bytes, and a `SET` whose key length is zero carries an empty key.

A receiver MUST check that `4 + key length` does not exceed `payload
length` before it takes either slice. A receiver that meets a `SET` payload
shorter than its four-byte head, or a `SET` payload whose `key length`
exceeds the bytes that follow it, MUST treat the frame as a malformed
request and close the connection. The check runs after the frame is
admitted, in the order Checks that follow the order states.

An encoder MUST write the key length as the length of the key it writes and
MUST write the value immediately after the key. Every `SET` payload has one
encoding.

A receiver refuses a REQUEST frame carrying an unassigned opcode at step 13
whatever its payload carries, and interprets no byte of that payload to
decide.

Every value of `payload length` is a legal encoding of a REQUEST frame at
this revision, within the frame size bound in force. A receiver
refuses a REQUEST frame carrying an unassigned opcode at step 13 whatever
its payload carries, and interprets no byte of that payload to decide.

## Assigning an Opcode Later

A later revision assigns an opcode as an addition. It needs neither a
protocol version bump nor a capability to do it.

A peer built on this revision meets the new value by the rule above. It
answers the unsupported operation class for that request and keeps the
connection open, the caller is told which request failed and why, and
every other request on the connection is unaffected.

The opcode space is the protocol's cheapest extension point, and it is
cheap because the rule for an unassigned value there is request-scoped.
Result Codes states a space whose rule is connection-fatal, and states
what that costs its own growth.
