---
title: Framing
description: The byte order, the frame header, the frame kinds protocol version 0 assigns, the flags field, the length arithmetic a receiver runs before it allocates, and the order in which it admits a frame.
protocol_version: 0
revision: v0.2.0
status: draft
order: 2
---

# Framing

## Scope

This document defines the byte order of the whole protocol, the frame
header, the frame kinds protocol version 0 assigns and the direction each
one travels, the flags field, the arithmetic a receiver runs to learn a
frame's total length, and the order in which a receiver admits a frame.

It does not define what any value the `code` field carries means. Each of
the three spaces that field draws from is defined in the section that owns
it.

## Byte Order

Every multi-byte integer on the wire is little-endian. This covers every
field of the frame header, every field of the metadata region, and every
length field inside a payload.

The byte order is fixed by this specification. It is never
platform-dependent, and it is never negotiated. A protocol that negotiated
its byte order would have two wire formats.

## Frame Header

Every frame of every kind begins with the same 20-byte header.

```text
offset  size  field            type
     0     1  version          u8
     1     1  kind             u8
     2     2  flags            u16
     4     2  code             u16
     6     2  metadata length  u16
     8     4  payload length   u32
    12     8  request id       u64
                               total = 20 bytes
```

A frame is the header, then the metadata region occupying exactly
`metadata length` bytes, then the payload occupying exactly
`payload length` bytes.

A receiver that holds the 20 bytes of the header knows the kind of the
frame, its total length, and the request it belongs to, without reading a
byte of the body.

### version

The only valid value of the `version` field in this document set is 0.
Values `0x01` to `0xFF` are reserved to other protocol versions, and no
revision of protocol version 0 assigns one.

A receiver that meets any value but 0 MUST treat the frame as an
unsupported protocol version failure and close the connection.

The version travels on every frame although one connection carries one
protocol version. A frame is then self-describing: a reader decodes it
without connection state, and a stream that has lost its framing fails at
its first frame rather than at an absurd length.

### kind

`kind` states which frame this is. Frame Kinds states the whole domain of
the field and the rule for every value.

### flags

Flags states the whole domain of the field and the rule for every value.

### code

`code` is one field whose meaning follows `kind`:

| Kind | What `code` holds | The section that owns the space |
|---|---|---|
| REQUEST | the opcode | Opcodes |
| RESPONSE | the result code | Result Codes |
| ERROR | the error class | The Error Class Registry |

Every frame kind this revision assigns draws `code` from a space, so no
frame kind it assigns leaves the field without a meaning.

### metadata length

`metadata length` states the length of the metadata region in bytes.
Every value of the width is a valid encoding of the field. Being `u16`,
the field cannot declare a region above 65535 bytes, and Frame Size
Limits states the bound that applies below that and what a receiver does
with a frame that exceeds it.

An absent metadata region is a `metadata length` of zero.

### payload length

`payload length` states the length of the payload in bytes. Every value
of the width is a valid encoding of the field. Frame Size Limits states
the bound that applies and what a receiver does with a frame that exceeds
it.

### request id

`request id` names the request a frame belongs to. Request Identity
states which peer allocates it and the scope its uniqueness runs over, and
The Reserved Request Id states its valid values and its reserved value.

The field carries no alignment guarantee. A frame begins at an arbitrary
offset in a receive buffer, so no padding in this layout could produce a
useful one.

## Frame Kinds

`kind` is a `u8`. The whole domain is covered here:

| Value | Kind | Sender |
|---|---|---|
| `0x00` | reserved, never valid | none |
| `0x01` | REQUEST | the client |
| `0x02` | RESPONSE | the server |
| `0x03` | ERROR | the server |
| `0x04..0xFF` | reserved | none |

A peer MUST NOT send a frame whose `kind` this revision does not assign.
A receiver that meets one MUST treat the frame as a protocol violation and
close the connection.

A receiver that cannot name a kind cannot tell whether it is owed an
answer or owes one. The header states the frame's length and a receiver
could step over it, but a REQUEST frame stepped over in silence leaves a
client waiting for an answer that never comes.

### Direction

Each assigned kind travels in one direction, stated in the table above. A
peer MUST NOT send a kind the table assigns to the other peer. A receiver
that meets a frame of a kind it sends itself MUST treat the frame as a
protocol violation and close the connection.

The direction is part of what tells a receiver which side of an exchange
owes something. A RESPONSE arriving at a server may name a request that
server is serving, so a receiver that read the request id first would
answer against a request that never failed.

### Reporting a connection-fatal failure

ERROR travels from the server to the client, so the two peers report a
failure differently, and every rule in this document set that closes a
connection is read with this:

A server that detects a connection-fatal failure sends an ERROR frame and
then closes the connection. A client that detects one closes the
connection without sending a frame, because this revision defines no frame
a client sends to report a failure.

## Flags

`flags` is a `u16`. All sixteen bits are reserved in protocol version 0.

A sender MUST set every bit of the field to zero. A receiver that meets a
frame with any non-zero bit in the field MUST treat the frame as a
protocol violation and close the connection.

The field holds two bytes at a fixed offset so that a later revision
carries a flag as a negotiated capability rather than as a protocol
version bump. Validating it is one comparison against zero.

## Frame Length Arithmetic

A frame's total length is:

```text
total length = 20 + metadata length + payload length
```

A receiver MUST compute the total in a width of at least 64 bits. The
widest values the two fields can carry are 65535 and 4294967295, so the
total is at most 4294967350. That exceeds what 32 bits hold, so a receiver
computing the total in 32 bits wraps, reads a small total for a frame that
declares an enormous one, and admits a frame it is required to refuse.

The two field widths are the whole of that proof. A receiver needs no
value from the body and no connection state to run it.

A receiver MUST check the total length against the maximum frame size
before it reserves memory proportional to any declared length. Frame Size
Limits states that bound, the bytes it counts, and what a receiver does
with a frame that exceeds it.

The check reads the header alone. A receiver that reserved a buffer from a
declared length before checking it would let an unauthenticated peer
choose an allocation with 20 bytes.

## Frame Admission Order

A receiver runs the steps below in order on every frame it reads, and
stops at the first one that decides. Two of them are not failures. Each of
the other thirteen names the error class and the failure scope its failure
produces, and step 13 names one pair for each frame kind.

A receiver MUST run the checks in this order. A receiver that checks in a
different order answers a frame violating two rules with the class of the
later step, and two receivers that ordered the steps differently answer
the same bytes with different classes.

This section states the order. The section that owns each check states the
requirement, names the peer it binds, and states its consequence.

```text
 step  condition                                    class                  scope
    1  fewer than 20 bytes held                     none, requires 20 bytes
    2  version is not 0                             unsupported protocol   connection-fatal
                                                    version
    3  kind is a value this revision does not       protocol violation     connection-fatal
       assign
    4  total length above the maximum frame size    resource limit         connection-fatal
    5  flags is not zero                            protocol violation     connection-fatal
    6  metadata length above the maximum metadata   malformed request      connection-fatal
       size
    7  fewer bytes held than the total length       none, requires the total length
    8  the metadata region does not fill exactly    malformed request      connection-fatal
    9  kind travels from the wrong direction        protocol violation     connection-fatal
   10  request id 0 on a kind that names a request  protocol violation     connection-fatal
   11  a frame opening a request already in flight  protocol violation     connection-fatal
   12  a frame naming a request not in flight       protocol violation     connection-fatal
   13  code unassigned for the kind
         REQUEST                                    unsupported operation  request-scoped
         RESPONSE                                   protocol violation     connection-fatal
         ERROR                                      protocol violation     connection-fatal
   14  metadata entries not strictly ascending      invalid argument       request-scoped
   15  a required metadata identifier this          invalid argument       request-scoped
       revision does not assign
```

Reporting a connection-fatal failure states how each peer reports a
failure at a step whose scope is connection-fatal.

### The two steps that are not failures

Steps 1 and 7 state that the receiver does not hold a frame yet. Neither
is a failure, neither produces an ERROR frame, and at either one the
receiver reads more bytes. Each states the total number of bytes the
receiver requires before it holds a frame: 20 bytes at step 1, and the
total length the header states at step 7.

A receiver MUST NOT wait for the bytes of a frame an earlier step has
already refused. Steps 2 to 6 decide from the header alone, so a frame
refused at any of them is refused with no byte of its body in hand.

### Checks that follow the order

A frame that passes all fifteen steps is admitted. The order decides
admission and nothing after it.

A section of this document set that defines a layout for a frame's payload
states its own checks over that payload, and those checks run on an
admitted frame, after step 15. The first failure still wins: a frame that
violates a step of the order and also violates a payload check produces
the class and the scope of the step, because the step decided first.

A receiver reaches no payload check on a frame the order refused. Steps 13
to 15 refuse a frame for what its `code` and its metadata region carry, so
a frame refused at any of them is refused with its payload held and
uninterpreted.

### Where a failure stops being fatal

Steps 1 to 8 decide from the header and the frame's own bytes. Steps 9 to
15 decide against the state the connection holds.

Every failure at steps 1 to 8 is connection-fatal, because a receiver that
cannot trust where the current frame ends cannot find the next one. From
step 9 the frame boundary is known and the receiver still owes an answer,
so a failure can be scoped to one request. Failure Scope states the
principle that decides which scope a class carries.

Steps 9 to 12 are connection-fatal although the frame boundary is known.
Each of them leaves the receiver unable to say which request the frame
belongs to, which is the other half of that principle.

Step 13 is connection-fatal for a RESPONSE and for an ERROR frame, whose
`code` is the whole of what the frame says, and request-scoped for a
REQUEST frame, whose request the receiver can name and answer. The section
that owns each of the three spaces states its own rule.
