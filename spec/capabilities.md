---
title: Capabilities
description: The capability identifier domain of protocol version 0, the entry layout the handshake carries, the receiver rule for an unassigned identifier or a value of the wrong length, the offer and acceptance rules, and the dependencies a capability may declare.
protocol_version: 0
revision: v0.5.1
status: draft
order: 6
---

# Capabilities

## Scope

This document defines the capability identifier domain, the capability
entry layout the handshake payload carries, the rules by which a capability
is offered and accepted, and the dependencies a capability may declare.

This revision assigns identifier `0x0002`, the cancellation capability,
which gates the withdrawal frame kind, identifier `0x0004`, the deadlines
capability, which gates the optional metadata identifier `0x0001`, and
identifier `0x0005`, the request classes capability, which gates the
optional metadata identifier `0x0002`. It does not define the handshake
exchange or its payloads, which Handshake owns, and it defines no
operation, result code or error class a capability would gate.

An engineer implementing this document alone can offer and accept the
capability set this revision assigns, ignore an identifier it does not
assign, and reject the two handshake payloads that violate the entry rules.

## The Capability Identifier Domain

A capability identifier is a `u16`. The whole domain is covered here:

| Value | Meaning |
|---|---|
| `0x0002` | cancellation |
| `0x0004` | deadlines |
| `0x0005` | request classes |
| `0x0000..0x0001`, `0x0003`, `0x0006..0xFFFF` | unassigned |

Protocol version 0 assigns identifiers `0x0002`, `0x0004` and `0x0005` at
revision v0.6.0. No value is reserved as never valid and none is reserved
for a later assignment in particular.

## Capability Entries

The entry layout is Capability Entries, in Handshake. An entry is
self-delimiting and a receiver advances four bytes plus the value length to
reach the next one. The entries fill the handshake payload exactly, and
they are in strictly ascending order of identifier; Validating a handshake
payload, in Handshake, states the failure each violation produces.

## An Unassigned Identifier

A receiver that meets a capability identifier this revision does not assign
MUST ignore the entry. It MUST NOT accept the identifier, MUST NOT produce
an error, and MUST advance four bytes plus the value length stated by the
entry. The value length is not a reason to refuse the entry: an entry
carrying an unassigned identifier is ignored whatever value or value length
it states.

This revision assigns identifiers, and each one carries no value. An entry
carrying an assigned identifier whose value length is not zero MUST be
treated as not accepted, and no error, by the rule the paragraph above
states for an identifier this revision does not assign. Each capability
below states the behavior of a connection that did not accept it.

Ignoring is safe in both directions because an unassigned identifier never
enters the accepted set. A responder that did not accept an identifier
never sends what it would gate, and an offerer that did not offer one never
receives it. The rule that makes it safe in the acceptance direction is
stated below.

## Offering and Acceptance

Only the offerer offers capabilities, in the handshake request. Only the
responder accepts them, and the accepted set is the entries the handshake
response carries. The accepted set is always a subset of the offered set.

A responder MUST NOT accept a capability the offerer did not offer. An
offerer that receives an accepted entry naming a capability it did not
offer MUST treat the response as a protocol violation and close the
connection.

A capability the responder did not accept is absent from the response. The
offerer MUST treat an absent capability as not available.

A peer MUST use a capability only after the acceptance names it. A peer
having accepted a capability, or an implementation of it, is not the same
as the connection having it: only an entry in the accepted set makes a
gated behavior legal.

A receiver that meets a frame a capability gates on a connection whose
accepted set does not name that capability MUST treat the frame as a
protocol violation and close the connection. The cancellation capability
gates the withdrawal frame kind, and its section states the rule.

## Dependencies

A capability may declare that it depends on another. A responder MUST
accept a capability that declares a dependency only when it also accepts
the capability the dependency names. When it does not, the accepted set
omits both, and the offer of the dependent capability alone resolves to an
accepted set that contains neither.

This revision assigns three capability identifiers, and it declares no
dependency for any of them.

## Growth

A later revision assigns a capability identifier together with the surface
the identifier gates. The assignment states the identifier, what it means,
which peer offers it, which peer accepts it, its value in each direction or
that it carries none, every capability it depends on, every frame kind,
operation, metadata identifier, result code and error class it gates, and
the behavior a peer without it receives instead.

Assigning an identifier is an addition. A peer built on an earlier revision
never offers an identifier it does not assign and never meets one, because
the accepted set is always a subset of the offered set and the peer offers
none.

## Cancellation

Identifier `0x0002` is the cancellation capability. It gates frame kind
`0x07`, the withdrawal frame: a connection that did not accept it has no
withdrawal.

The client offers it in the handshake request and the server accepts it in
the handshake response. It carries no value in either direction: an entry
carrying this identifier has a value length of zero.

A peer MUST NOT send a withdrawal frame on a connection whose accepted set
does not name this capability. A receiver that meets one MUST treat the
frame as a protocol violation and close the connection, by the rule
Offering and Acceptance states for a frame a capability gates. Request
Lifetime states what a withdrawal does.

A receiver that meets a handshake entry carrying this identifier whose
value length is not zero MUST ignore the entry and produce no error, by
the rule An Unassigned Identifier states, and the capability is not
accepted.

## Deadlines

Identifier `0x0004` is the deadlines capability. It gates metadata
identifier `0x0001`, the deadline entry: a connection that did not accept
it has no deadlines.

The client offers it in the handshake request and the server accepts it in
the handshake response. It carries no value in either direction: an entry
carrying this identifier has a value length of zero.

A receiver that meets a deadline entry on a connection whose accepted set
does not name this capability MUST treat the entry as one it does not
read: it skips the entry by the rule The Optional Range states, produces
no failure, and serves the request without a deadline. Metadata states the
entry's encoding and Request Lifetime states what a deadline does.

A receiver that meets a handshake entry carrying this identifier whose
value length is not zero MUST ignore the entry and produce no error, by
the rule An Unassigned Identifier states, and the capability is not
accepted.

## Request Classes

Identifier `0x0005` is the request classes capability. It gates metadata
identifier `0x0002`, the request class entry: a connection that did not
accept it serves every request at the responder's default.

The client offers it in the handshake request and the server accepts it in
the handshake response. It carries no value in either direction: an entry
carrying this identifier has a value length of zero.

A receiver that meets a request class entry on a connection whose accepted
set does not name this capability MUST treat the entry as one it does not
read: it skips the entry by the rule The Optional Range states, produces
no failure, and serves the request at the responder's default. Metadata
states the entry's encoding.

A receiver that meets a handshake entry carrying this identifier whose
value length is not zero MUST ignore the entry and produce no error, by
the rule An Unassigned Identifier states, and the capability is not
accepted.

## The Registry

| Id | Capability | What it gates |
|---|---|---|
| `0x0002` | cancellation | frame kind `0x07` |
| `0x0004` | deadlines | metadata identifier `0x0001` |
| `0x0005` | request classes | metadata identifier `0x0002` |
| `0x0000..0x0001`, `0x0003`, `0x0006..0xFFFF` | unassigned | nothing |

Every unassigned identifier is ignored by An Unassigned Identifier, and an
identifier this revision assigns enters the accepted set only when the
handshake response names it and the request offered it. The registry covers
the whole domain.
