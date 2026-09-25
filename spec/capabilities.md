---
title: Capabilities
description: The capability identifier domain of protocol version 0, the entry layout the handshake carries, the receiver rule for an unassigned identifier or a value of the wrong length, the offer and acceptance rules, and the dependencies a capability may declare.
protocol_version: 0
revision: v0.6.0
status: draft
order: 6
---

# Capabilities

## Scope

This document defines the capability identifier domain, the capability
entry layout the handshake payload carries, the rules by which a capability
is offered and accepted, and the dependencies a capability may declare.

It assigns no capability identifier, so nothing is gated at this revision
and every offer is refused by acceptance of nothing. It does not define the
handshake exchange or its payloads, which Handshake owns, and it defines no
frame kind, operation, metadata identifier, result code or error class a
capability would gate.

An engineer implementing this document alone can offer and accept a
capability set, ignore an identifier it does not assign, and reject the two
handshake payloads that violate the entry rules.

## The Capability Identifier Domain

A capability identifier is a `u16`. The whole domain is covered here:

| Value | Meaning |
|---|---|
| `0x0000..0xFFFF` | unassigned |

Protocol version 0 assigns no capability identifier at revision v0.6.0. No
value is reserved as never valid and none is reserved for a later
assignment in particular.

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

From the revision that assigns an identifier, an entry whose value length
is wrong for that identifier MUST be treated the same way: not accepted,
and no error. This revision assigns no identifier, so that case cannot
arise here and no fixture of this revision reaches it.

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
protocol violation and close the connection. This revision assigns no
capability, so no frame is gated and the rule binds the revisions that
assign one.

## Dependencies

A capability may declare that it depends on another. A responder MUST
accept a capability that declares a dependency only when it also accepts
the capability the dependency names. When it does not, the accepted set
omits both, and the offer of the dependent capability alone resolves to an
accepted set that contains neither.

This revision assigns no capability identifier, so it declares no
dependency.

## Growth

A later revision assigns a capability identifier together with the surface
the identifier gates. The assignment states the identifier, what it means,
which peer offers it, which peer accepts it, its value in each direction or
that it carries none, every capability it depends on, every frame kind,
operation, metadata identifier, result code and error class it gates, and
the behavior a peer without it receives instead.

Assigning an identifier is an addition. A peer built on this revision never
offers an identifier it does not assign and never meets one, because the
accepted set is always a subset of the offered set and the peer offers
none.

## The Registry

| Id | Capability | What it gates |
|---|---|---|
| `0x0000..0xFFFF` | unassigned | nothing |

Every identifier is unassigned, no identifier is accepted unless a later
revision assigns it, and every entry is ignored by An Unassigned
Identifier. The registry covers the whole domain and states no assigned
value, so no clause of this revision depends on a capability.
