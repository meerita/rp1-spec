---
title: Limits
description: The pre-negotiation and negotiated size bounds of protocol version 0, the bytes each counts, the formula that derives each negotiated bound, where a receiver enforces them, and the failure a frame that exceeds one produces.
protocol_version: 0
revision: v0.5.0
status: draft
order: 10
---

# Limits

## Scope

This document defines the size bounds a peer decodes under in protocol
version 0: the two constants that apply before the handshake completes,
the two bounds the handshake derives, the bytes each bound counts, the
formula that derives each negotiated bound with its floor and its ceiling,
the point a receiver enforces each, and the failure a frame that exceeds
one produces.

It does not bound the number of frames a connection carries or the number
of requests in flight, and this revision defines no other limit.

## Pre-Negotiation Bounds

Protocol version 0 fixes two constants that apply to a frame before the
handshake completes:

```text
maximum frame size      65536 bytes
maximum metadata size    4096 bytes
```

Both bind the client and the server identically, on every frame, from the
first byte of a connection until the handshake response is read. A
receiver therefore bounds its first allocation with no prior exchange.
Neither is negotiated and neither is carried on the wire.

### The pre-negotiation maximum frame size

The maximum frame size bounds a frame's total length: the header, the
metadata region and the payload together. Every byte of the frame counts
toward it, including the 20 bytes of the header. Frame Length Arithmetic
states how a receiver computes the total.

A peer MUST NOT send a frame whose total length exceeds the bound in force.
A receiver that meets one MUST treat the frame as a resource limit failure
and close the connection.

A frame at the pre-negotiation bound carries 65516 bytes across its
metadata region and its payload together, which is the bound less the
header.

Frame Length Arithmetic states the check a receiver runs against this
bound, and requires it before the receiver reserves memory proportional to
any declared length. Frame Admission Order places that check at step 4 and
places the point at which a receiver waits for the rest of the frame at
step 7, so a frame this bound refuses is refused from its header, with no
byte of its body in hand.

### The pre-negotiation maximum metadata size

The maximum metadata size bounds the metadata region alone. It counts the
bytes `metadata length` declares, and no byte of the header and no byte of
the payload.

A peer MUST NOT send a frame whose `metadata length` exceeds the bound in
force. A receiver that meets one MUST treat the frame as a malformed
request and close the connection.

A frame can exceed both bounds, and the two produce different classes.
Frame Admission Order places the maximum frame size check at step 4 and
this one at step 6, so a frame that exceeds both is a resource limit
failure and never a malformed request.

## Negotiated Bounds

A connection in the negotiated state decodes under the values the
handshake response states. The response carries the negotiated maximum
frame size and the negotiated maximum metadata size, and each replaces the
pre-negotiation constant of the same subject. Before the handshake
completes, the constants above are in force.

The responder computes both values and states the result. The offerer
proposes the frame size and proposes nothing for the metadata bound. The
responder's ceiling is a value of its own configuration, is not carried on
the wire, and no offerer needs it: the offerer reads the negotiated value
from the response and bounds its allocations by it.

### Negotiated maximum frame size

```text
negotiated maximum frame size = min(max(proposed, 65536), ceiling)

  proposed   the client desired maximum frame size of the handshake request
  ceiling    the responder's configured maximum frame size, >= 65536
```

The value covers the header, the metadata region and the payload together.
A responder's ceiling MUST be at least 65536, so the negotiated value is at
least 65536 whatever the offerer proposes, including zero. The floor is
inside the formula, so an offerer that proposes a value below the floor is
answered a value above what it proposed, and it MUST size its receive
buffer by the value it reads back and not by the value it proposed.

A peer MUST NOT send a frame whose total length exceeds the negotiated
maximum frame size. A receiver that meets one MUST treat the frame as a
resource limit failure and close the connection.

A value below 65536 is outside what a responder may state. An offerer that
reads a `negotiated maximum frame size` below 65536 MUST treat the response
as a malformed handshake and close the connection.

### Negotiated maximum metadata size

```text
4096 <= negotiated maximum metadata size <= 65535

  floor     4096, the pre-negotiation maximum metadata size
  ceiling   65535, enforced by the u16 width of the field that carries it
```

The responder states this value in the handshake response and the offerer
proposes nothing for it. The responder MUST state a value of at least 4096,
so a responder that admitted the handshake request under the
pre-negotiation bound cannot then require the connection to use less.

A peer MUST NOT send a frame whose `metadata length` exceeds the negotiated
value. A receiver that meets one MUST treat the frame as a malformed
request and close the connection.

An offerer that reads a `negotiated maximum metadata size` below 4096 MUST
treat the response as a malformed handshake and close the connection.

### Where the negotiated checks run

Frame Admission Order places the frame size check at step 4 and the
metadata size check at step 6 for every frame, in every connection state.
The value each check compares against is the bound in force: the
pre-negotiation constant before the handshake response is read, and the
negotiated value after. Both checks run before the receiver reserves memory
proportional to any declared length.

### Scope

Negotiated bounds are connection-scoped. A new connection renegotiates both
and inherits neither.

## The Bounds the Field Widths Enforce

`metadata length` is a `u16` and `payload length` is a `u32`, so the width
of each field already bounds the region it declares:

```text
metadata region    at most 65535 bytes, from the width of the field
payload            at most 4294967295 bytes, from the width of the field
```

Each bound this document fixes lies below the width bound of the field it
applies to, so the value this document fixes is the one that decides in
every case. A receiver that enforces only a field width admits frames this
contract requires it to refuse.
