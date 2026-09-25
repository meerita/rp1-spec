---
title: Handshake
description: The connection states of protocol version 0, which frames are legal before the handshake completes, the failure a frame outside them produces, the handshake exchange in both directions, and version negotiation.
protocol_version: 0
revision: v0.5.1
status: draft
order: 5
---

# Handshake

## Scope

This document defines the state model of a protocol version 0 connection:
the states a connection occupies, the frame that moves it from one to the
next, which frames are legal in each, and the failure a frame outside them
produces. It defines the handshake exchange in both directions and the
version negotiation it carries.

It does not define the capability identifier domain or the offer and
acceptance rules for a capability, which Capabilities owns, although it
defines the layout of the capability entries the handshake payload
carries. It does not define the negotiated bounds, which Limits owns. It
defines no operation other than the handshake, which Operations owns. It
defines no conformance layer, no role obligation, and no required minimum.

An engineer implementing this document alone can decide, for every frame
on a connection, whether the connection admits it at that point in the
connection's life, what to do when it does not, and which protocol version
the connection carries after a successful exchange.

## The Connection States

A connection occupies exactly one of three states at a time. The state is
per connection and never shared between connections.

| State | Entered when | Left when |
|---|---|---|
| pre-negotiation | the transport connects | the offerer receives a handshake response, or the connection fails |
| negotiated | the offerer receives a handshake response | the transport closes, or a connection-fatal failure occurs |
| terminal | the transport closes or a peer closes it after a connection-fatal failure | never; the connection is unusable |

A connection enters the pre-negotiation state when the transport connects,
not when a frame arrives. A transport that is connected says nothing about
whether a peer has negotiated, and no rule in this document set is
satisfied by the transport's state alone.

The state a connection occupies decides which frames it admits. A frame
that is legal in one state and not in the state the connection occupies is
a protocol violation, and a receiver that meets one MUST close the
connection. The sections below state, for each state, which frames are
legal and what a frame outside them produces.

## Before the Handshake

The first frame of a connection MUST be a REQUEST frame carrying the
handshake opcode. A receiver that meets any other frame in the
pre-negotiation state — a different frame kind, or a REQUEST frame
carrying a different opcode — MUST treat the frame as a protocol violation
and close the connection.

An offerer MUST NOT send any frame other than the handshake request before
it has received the handshake response. A responder that meets one MUST
treat it as a protocol violation and close the connection.

The bounds in force in the pre-negotiation state are the maximum frame
size and the maximum metadata size Limits states. They are constants of
protocol version 0, not negotiated values, so a decoder is bounded before
any exchange completes.

A connection in the pre-negotiation state admits exactly one request from
the offerer and exactly one terminal frame from the responder: a RESPONSE
frame carrying the success result code when the handshake succeeds, and an
ERROR frame when it fails. A responder that will not admit the connection
closes the transport without sending a frame. A peer that detects a
connection-fatal failure in this state reports it the way Framing states:
a server sends an ERROR frame and then closes, and a client closes without
sending a frame.

## The Handshake Exchange

The handshake is a REQUEST frame carrying the opcode Operations assigns to
the handshake, answered by a RESPONSE frame carrying the success result
code. The opcode is `0x0001` and the result code is `0x0000`. The
`version` field of the header of the request carries the protocol version
the offerer proposes.

The request carries the offerer's proposed version range, its desired
frame size, and its capability offers. The response carries the negotiated
version, the negotiated bounds, and the accepted capability set.

### The handshake request payload

```text
offset  size  field                              type
     0     2  client maximum protocol version    u16
     2     2  client minimum protocol version    u16
     4     4  client desired maximum frame size  u32
     8     2  capability count                   u16
    10    ..  capability entries
```

The fixed head is ten bytes. The tail is `capability count` entries, each
in the layout under Capability Entries below.

The offerer proposes a version range as an inclusive minimum and maximum.
It MUST propose a range it supports in full: every value from the minimum
to the maximum, inclusive, is a version the offerer accepts. A range whose
maximum is below its minimum is a malformed request, and a receiver that
meets one MUST close the connection.

An offerer that proposes a version it does not support accepts a connection
that may become unusable at its first frame, and no peer can detect from
the wire that the proposal was false.

The offerer proposes a desired maximum frame size. It must be prepared to
receive a frame of the negotiated size, which Limits derives.

### The handshake response payload

```text
offset  size  field                             type
     0     2  negotiated protocol version       u16
     2     4  negotiated maximum frame size     u32
     6     2  negotiated maximum metadata size  u16
     8     2  accepted capability count         u16
    10    ..  accepted capability entries, the same layout as the request
```

The fixed head is ten bytes. The response travels in a RESPONSE frame
carrying the success result code, and it is the terminal frame of the
handshake request.

The response states the negotiated version, the negotiated maximum frame
size, the negotiated maximum metadata size, and the accepted capability
set. Negotiated Frame Size and Negotiated Metadata Size, in Limits, state
the formula that derives each bound.

### Capability Entries

A capability entry is:

```text
offset  size  field          type
     0     2  capability id  u16
     2     2  value length   u16
     4     n  value bytes
```

An entry is self-delimiting: a reader advances four bytes plus the value
length to reach the next entry. Capabilities owns what an identifier means
and which entries a responder accepts.

### Validating a handshake payload

Both payloads have a fixed head of ten bytes. A payload shorter than its
head is a malformed request, and a receiver that meets one MUST close the
connection.

The capability count is a claim to be checked and never a size to trust. A
receiver MUST NOT reserve memory proportional to the declared count before
it has read the entries. The entries fill the payload exactly: a region
that ends part way through an entry, or an entry whose value length carries
it past the end of the payload, is a malformed request, and a receiver
that meets one MUST close the connection.

The entries are in strictly ascending order of capability id. A region
whose entries are not strictly ascending, including one that carries an
identifier twice, is a malformed request, and a receiver that meets one
MUST close the connection.

These checks run before the offerer reads any negotiated value. A receiver
that meets any of them closes the connection with no frame, as Before the
Handshake states.

## Version Negotiation

The offerer proposes a version range in the handshake request. The
responder MUST select a protocol version that lies within that range and
that the responder supports, and MUST state it in `negotiated protocol
version`. The responder SHOULD select the highest version it supports that
lies within the range, so that the connection runs on the newest version
both peers can use; a responder that selects a lower version it supports
still interoperates, because the offerer accepts every value in the range
it proposed.

The offerer MUST accept any value the responder states inside the range it
proposed. An offerer that receives a version outside the range it proposed
MUST treat the response as a malformed handshake and close the connection.

A responder that supports no version in the offered range answers the
unsupported protocol version error class and closes the connection. Every
handshake failure is connection-fatal.

A negotiated version above 255 is a value no frame header can carry in its
one-byte `version` field. A responder MUST NOT state one, and an offerer
that receives one MUST treat the response as a malformed handshake and
close the connection.

At revision v0.5.1 the only version this specification defines is 0, so
the highest version a conforming responder supports in any range that
contains 0 is 0. A later revision that defines another protocol version
adds it to the range a responder supports; the rule above does not change.

## After the Handshake

A connection enters the negotiated state when the offerer receives the
handshake response. Frame Kinds states which kinds are legal in this state
and the direction each travels, and the handshake adds no frame kind of
its own.

A peer MUST NOT send a handshake request on a connection in the negotiated
state. A responder that meets a second handshake request MUST treat it as a
protocol violation and close the connection.

The negotiated protocol version, the negotiated bounds, and the accepted
capability set are connection-scoped. Capabilities states what the accepted
set gates, and this revision assigns no capability identifier, so the
accepted set is empty at this revision and gates nothing. A new connection
renegotiates every value and inherits nothing from the connection that
preceded it.

## The Terminal State

A connection reaches the terminal state when the transport closes, or when
a peer closes it after a connection-fatal failure. No frame is legal in
the terminal state, and a peer sends none on it.

A transport close is the only orderly ending protocol version 0 defines.
Neither peer sends a frame to open one, because the revision defines no
frame for it.

A connection that ends retires no request. Every request in flight at a
peer when the connection ends receives no terminal frame, and an initiator
MUST NOT conclude from the connection ending that the work a request named
took effect, and MUST NOT conclude that it did not. Correlation states the
rule in full and what an initiator does to recover completion certainty.
