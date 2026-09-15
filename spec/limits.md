---
title: Limits
description: The maximum frame size and the maximum metadata size of protocol version 0, the bytes each one counts, where a receiver enforces them, and the failure a frame that exceeds one produces.
protocol_version: 0
revision: v0.1.2
status: draft
order: 8
---

# Limits

## Scope

This document defines the two size bounds a peer decodes under in protocol
version 0. For each one it states the bytes the bound counts, its value,
the point a receiver enforces it relative to the memory it reserves, and
the failure a frame that exceeds it produces.

It does not define a negotiated bound. This revision defines no exchange
that could carry one, so both bounds below are fixed values of this
revision.

It bounds nothing else. Neither bound below limits the number of frames a
connection carries or the number of requests in flight, and this revision
defines no other limit.

## Frame Size Limits

Protocol version 0 fixes two bounds:

```text
maximum frame size      65536 bytes
maximum metadata size    4096 bytes
```

Neither bound is carried on the wire, neither is negotiated, and neither
has a default distinct from its value. Each is a fixed value of protocol
version 0 and each is protocol-wide: it binds the client and the server
identically, on every frame of every connection, from the first byte of
that connection. A receiver therefore bounds its first allocation with no
prior exchange.

### The maximum frame size

The maximum frame size bounds a frame's total length: the header, the
metadata region and the payload together. Every byte of the frame counts
toward it, including the 20 bytes of the header. Frame Length Arithmetic
states how a receiver computes the total.

A peer MUST NOT send a frame whose total length exceeds 65536 bytes. A
receiver that meets one MUST treat the frame as a resource limit failure
and close the connection.

A frame at the bound carries 65516 bytes across its metadata region and
its payload together, which is the bound less the header.

Frame Length Arithmetic states the check a receiver runs against this
bound, and requires it before the receiver reserves memory proportional to
any declared length. Frame Admission Order places that check at step 4 and
places the point at which a receiver waits for the rest of the frame at
step 7, so a frame this bound refuses is refused from its header, with no
byte of its body in hand. A receiver that waited first would let the peer
that sent those 20 bytes choose both the buffer it reserves and how long
it holds a connection open for a frame it is required to refuse.

### The maximum metadata size

The maximum metadata size bounds the metadata region alone. It counts the
bytes `metadata length` declares, and no byte of the header and no byte of
the payload.

A peer MUST NOT send a frame whose `metadata length` exceeds 4096. A
receiver that meets one MUST treat the frame as a malformed request and
close the connection.

A receiver MUST check `metadata length` against this bound before it
reserves memory proportional to that length.

A frame can exceed both bounds, and the two produce different classes.
Frame Admission Order places the maximum frame size check at step 4 and
this one at step 6, so a frame that exceeds both is a resource limit
failure and never a malformed request.

### The bounds the field widths enforce

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
