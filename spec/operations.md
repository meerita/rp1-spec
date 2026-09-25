---
title: Operations
description: Where an opcode appears in a frame, the whole domain of the opcode space, the opcode protocol version 0 assigns at this revision, what a receiver does with an opcode protocol version 0 does not assign, and what a later revision does when it assigns one.
protocol_version: 0
revision: v0.4.0
status: draft
order: 7
---

# Operations

## Scope

This document defines the opcode space: where an opcode appears in a
frame, the whole domain of the field that carries it, the opcodes this
revision assigns, what a receiver does with an opcode this revision does
not assign, and what a later revision does when it assigns one.

It assigns one opcode, the handshake, and reserves the rest of the space.
Handshake owns the handshake request payload and the rule for when the
handshake request is legal. No other operation is defined, so this
document names no other work a server performs and no other payload a
request carries. Known Limitations states what that costs an implementer.

## Opcodes

An opcode appears in the `code` field of a REQUEST frame. Frame Header
states that `code` is a `u16` and that its meaning follows the frame's
kind. No other frame kind this revision assigns carries an opcode.

The whole domain is covered here:

| Value | Meaning |
|---|---|
| `0x0001` | the handshake, defined by Handshake |
| `0x0000` | reserved |
| `0x0002..0xFFFF` | reserved |

No value of the field is reserved as never valid. `0x0000` is reserved on
the same terms as every value but `0x0001`, and a receiver answers it by
the rule below.

A peer MUST NOT send a REQUEST frame whose `code` this revision does not
assign. A receiver that meets one MUST answer the unsupported operation
error class for that request and MUST keep the connection open. Reporting
a request-scoped failure states what each peer does to answer.

Frame Admission Order places this check at step 13.

A REQUEST frame carrying the assigned opcode is the handshake request.
Before the Handshake states that it is the only frame legal in the
pre-negotiation state, and it states the consequence of sending it at any
other time.

## The REQUEST frame payload

The handshake request payload is defined by Handshake. It is the only
payload this revision defines for a REQUEST frame.

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
