---
title: Operations
description: Where an opcode appears in a frame, the whole domain of the opcode space, what a receiver does with an opcode protocol version 0 does not assign, and what a later revision does when it assigns one.
protocol_version: 0
revision: v0.1.1
status: draft
order: 5
---

# Operations

## Scope

This document defines the opcode space: where an opcode appears in a
frame, the whole domain of the field that carries it, what a receiver does
with an opcode this revision does not assign, and what a later revision
does when it assigns one.

It assigns no opcode. Protocol version 0 assigns none at revision v0.1.1
and reserves the whole space. What this revision publishes is the space
and the rule it carries, so that a later revision assigns an opcode
without invalidating a peer built on this one.

It defines no operation, no operation payload, and no work a server
performs. Known Limitations states what that costs an implementer.

## Opcodes

An opcode appears in the `code` field of a REQUEST frame. Frame Header
states that `code` is a `u16` and that its meaning follows the frame's
kind. No other frame kind this revision assigns carries an opcode.

The whole domain is covered here:

| Value | Meaning |
|---|---|
| `0x0000..0xFFFF` | reserved |

Protocol version 0 assigns no opcode at revision v0.1.1. Every value of
the field is reserved, `0x0000` on the same terms as every other, and no
value is reserved as never valid.

A peer MUST NOT send a REQUEST frame whose `code` this revision does not
assign. A receiver that meets one MUST answer the unsupported operation
error class for that request and MUST keep the connection open. Reporting
a request-scoped failure states what each peer does to answer.

Frame Admission Order places this check at step 13.

Every REQUEST frame that reaches step 13 at this revision is refused
there, because no value of the field is assigned. A client that sends one
receives an ERROR frame carrying the unsupported operation class and
naming its request, and the connection keeps serving. No other outcome of
a REQUEST frame is reachable at this revision.

A peer implementing this revision alone therefore sends no REQUEST frame,
because every value it could put in the field is one this revision does
not assign. The rule above is what a receiver does when a peer sends one
regardless, and it is the rule a peer built on this revision applies to
the first opcode a later revision assigns.

### The REQUEST frame payload

This revision assigns no opcode, so it defines no payload for a REQUEST
frame. An opcode states what its request's payload carries, and none is
assigned.

Every value of `payload length` is a legal encoding of a REQUEST frame at
this revision, within the bound Frame Size Limits states. A receiver
refuses the frame at step 13 whatever its payload carries, and interprets
no byte of that payload to decide.

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
