---
title: Operations
description: Where an opcode appears in a frame, the whole domain of the opcode space, the opcodes protocol version 0 assigns at this revision, the request and response payload of each operation, the argument encoding they share, the validation each runs, what a receiver does with an opcode this revision does not assign, and what a later revision does when it assigns one.
protocol_version: 0
revision: v0.5.0
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

## The Liveness Operation

`PING` is opcode `0x0002`. It asks the responder to answer. It names no
key, it carries no argument, and it produces no answer beyond the fact that
the responder served it.

A `PING` request payload is empty. A sender MUST NOT send a `PING` request
whose payload is not empty. A receiver that meets one MUST treat the frame
as a malformed request and close the connection, because a payload the
operation does not define is refused rather than ignored.

A responder answers a `PING` request with a RESPONSE frame carrying the
success result code and an empty payload.

A `PING` request reaches the success result code and no other result code.
It never answers the absent result code, because it names no key, and it
never answers a code a later revision assigns.

A caller concludes from a completed `PING` that the responder served a
request on the connection and answered it. It concludes nothing about any
key, because `PING` names none, and nothing about whether any other
operation is reachable.

A client MAY send `PING` to keep a connection active or to measure a round
trip. A server MUST serve a `PING` request on any connection in the
negotiated state, without a capability and without a metadata entry.

## The Read Operations

`GET` and `EXISTS` read one key. `GET` asks for the key's value, and
`EXISTS` asks only whether the key is present. Both carry the key in the
encoding The REQUEST frame payload states.

### GET

`GET` is opcode `0x0003`.

A responder answers a `GET` request with one of three result codes:

| Result code | The answer |
|---|---|
| success | the value bytes, which may be zero bytes |
| absent | zero bytes; the key does not exist |
| value held outside memory | eight bytes: the logical length of the value the key holds |

A success whose payload is zero bytes states that the key exists and holds
an empty value. The absent result code states that the key does not exist.
The two are different frames, and a caller separates them by the result
code and parses no length to do it. A caller MUST NOT treat a success with a
zero-byte payload as an absent key.

The value-held-outside-memory result code is not a failure and not an
answer that carries the value: the key exists and the responder states the
logical length of its value without carrying a byte of it. A `GET` request
reaches this code because this revision assigns no capability, and a
connection that negotiated nothing is a connection the responder can send
no value fragment to.

A `GET` request reaches the success, absent and value-held-outside-memory
result codes and no other result code.

A key that does not exist is a result and not a failure. A responder MUST
answer a `GET` of a key that does not exist with the absent result code and
MUST NOT answer an error class.

### EXISTS

`EXISTS` is opcode `0x0006`.

A responder answers an `EXISTS` request with the success result code and an
empty payload when the key is present, and with the absent result code and
an empty payload when it is not. The result code carries presence; the
payload is empty in both cases.

An `EXISTS` request asks about the key and never about the representation
it holds. A responder MUST answer presence for a key whatever
representation it holds and MUST NOT refuse the request on the ground of
that representation.

An `EXISTS` request reaches the success and absent result codes and no
other result code.

## The Write and Delete Operations

`SET` writes one key and `DEL` removes one key. Both are mutations, and a
caller of either learns from the answer whether the mutation took effect.

### SET

`SET` is opcode `0x0004`. Its request payload is the `u32` key length, the
key, and the value, in the encoding The REQUEST frame payload states. A
`SET` whose key length is zero writes the empty key, and a `SET` whose key
length leaves no byte reads as a value of zero bytes.

A responder answers a `SET` request with the success result code and an
empty payload. A `SET` request reaches the success result code and no other
result code.

A `SET` request can answer a failure class instead. The classes a `SET`
request can answer are malformed request, unsupported operation, resource
limit, protocol violation, invalid argument, overloaded, wrong type and
internal error. The class states whether the mutation may have taken
effect, and Failures states that fact for every assigned class: every class
above states that nothing was written, except `internal error`, which
states that the mutation may have taken effect. A caller that must know
whether a `SET` that answered an `internal error` took effect MUST re-read
the key.

A `SET` against a key that holds a representation other than a byte value
answers the wrong type class and stores nothing. A `SET` that answers any
other failure class stores nothing.

### DEL

`DEL` is opcode `0x0005`. Its request payload is the key, in the same
encoding as `GET` and `EXISTS`.

A responder answers a `DEL` request with the success result code and an
empty payload when it removed a key, and with the absent result code and an
empty payload when the key did not exist. Removing a key that does not
exist is a result and not a failure, and a responder MUST NOT answer an
error class for it.

A `DEL` removes whichever representation the key holds, so a `DEL` request
never answers the wrong type class. The classes a `DEL` request can answer
are malformed request, unsupported operation, resource limit, protocol
violation, invalid argument, overloaded and internal error. Every one
states that nothing was written, except `internal error`, which states that
the mutation may have taken effect; a caller that must know MUST re-read
the key.

A `DEL` request reaches the success and absent result codes and no other
result code.

## Argument Validation

A receiver validates the payload of each admitted request before the
operation runs. The frame passed every check of Frame Admission Order, so
its boundary is known; the checks here are over the argument the operation
carries, and they run in the order Checks that follow the order states.

```text
operation        the check                              the class
PING             the payload is empty                   malformed request
GET, DEL, EXISTS no check: every payload is a key      none
SET              the payload is at least four bytes    malformed request
SET              4 + key length does not exceed the
                 payload length                        malformed request
every operation  a key or a value above the
                 responder's limit                     invalid argument
```

A `PING` payload of one byte or more is a malformed request: the operation
defines no field for those bytes, so they are refused rather than ignored.
A `GET`, `DEL` or `EXISTS` payload is the whole key, of zero or more bytes,
so no payload of those three fails a structural check.

The two classes differ because the failures differ. A malformed request
leaves the extent of the payload unknown, so the receiver cannot trust the
next frame and the failure is connection-fatal. An invalid argument was
read whole and refused by the responder, so one request fails and the
connection keeps serving.

A responder MUST NOT answer a connection-fatal class for a request payload
whose boundaries it read. A `SET` whose key length exceeds the bytes that
follow it is a malformed request because the value's extent is unknown; a
key the responder refuses by size after it read it is an invalid argument,
request-scoped, which Limits states.

No rejection above uses a capability, and no operation is refused on the
ground that a capability was not negotiated, because this revision assigns
no capability.

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
