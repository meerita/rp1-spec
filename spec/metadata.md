---
title: Metadata
description: The metadata region of a frame: where it sits, the entry layout, the order entries appear in, the whole identifier domain, and what a receiver does with an identifier protocol version 0 does not assign.
protocol_version: 0
revision: v0.5.1
status: draft
order: 3
---

# Metadata

## Scope

This document defines the metadata region: where it sits in a frame, how
long it is, the layout of an entry, the order entries appear in, the whole
domain of the identifier field, and what a receiver does with an entry
whose identifier this revision does not assign.

This revision assigns identifiers `0x0001`, the deadline, and `0x0002`,
the request class, in the optional range. Both ranges carry the rule each
one states for an identifier this revision does not assign, so a later
revision assigns a further identifier without invalidating a peer built on
this one.

It does not define the bound on the region's length. Limits states that
bound, the bytes it counts, and what a receiver does with a frame that
exceeds it.

## The Metadata Region

The metadata region follows the frame header and occupies exactly
`metadata length` bytes. Frame Header owns that field and states that
an absent region is a `metadata length` of zero. A region of zero bytes
carries no entries and the payload begins immediately after the header.

The region is a sequence of entries. Entry Order states the order they
appear in and Region Fill states that they fill the region exactly.

Every frame kind this revision assigns MAY carry a metadata region, in
either direction. A receiver MUST accept a region on every frame kind it
admits, and MUST apply the rules of this document to it whatever the kind
of the frame that carried it. A receiver MUST NOT treat a region on a
frame kind as a failure on the ground of the kind alone.

The header carries a `metadata length` on every frame of every kind, so
the region exists on every kind whether or not a revision assigns it a
use. A revision that forbade a region on a kind would have to permit it
again later, and permitting it again makes a peer built on the earlier
revision refuse a frame the later one allows. A receiver pays one walk of
a region that is empty on every frame a peer implementing this revision
alone sends.

## The Entry Layout

Every entry has the same layout:

```text
offset  size  field         type
     0     2  identifier    u16
     2     2  value length  u16
     4     n  value bytes
                            total = 4 + value length
```

An entry occupies `4 + value length` bytes. The offsets above are relative
to the start of the entry, and the first entry of a region starts at the
first byte of the region.

`identifier` states which entry this is. Metadata Identifiers states the
whole domain of the field and the rule for every value.

`value length` states the length of the entry's value bytes. Every value
of the width is a valid encoding of the field, and Region Fill states the
check a receiver runs against it before it takes a slice.

A `value length` of zero is an entry that carries no value bytes. It is
not the same as an absent entry: the entry is present, its identifier is
present, and a receiver acts on it by its identifier.

The value bytes are opaque. A receiver that does not recognise an entry's
identifier interprets none of them. The Assigned Identifiers states the
meaning this revision gives to the value bytes of each entry it assigns.

## Region Fill

The entries of a region MUST fill it exactly. The first entry starts at
the first byte of the region, each following entry starts where the
previous one ends, and the last one ends at the last byte of the region.

A receiver MUST check that `4 + value length` does not exceed the bytes
remaining in the region before it takes the entry's value bytes.

A peer MUST NOT send a region whose entries do not fill it exactly. A
receiver that meets a region that ends part way through an entry, or an
entry whose `4 + value length` exceeds the bytes remaining in the region,
MUST treat the frame as a malformed request and close the connection.

Frame Admission Order places this check at step 8, before every check that
reads connection state.

The region's extent is what tells a receiver where the payload begins. A
receiver that met a region it could not walk to its last byte would have
to guess the payload's first byte, and a guess there loses the frame
boundary and every frame after it. That is why this failure closes the
connection while the two below do not.

## Entry Order

Entries MUST appear in strictly ascending order of identifier. A receiver
compares each identifier against the previous one and requires a strict
increase.

A duplicate identifier follows from that rule and is not a second one: two
equal identifiers are not a strict increase, so a region carrying the same
identifier twice is a region that is not strictly ascending.

A peer MUST NOT send a region whose entries are not strictly ascending. A
receiver that meets one MUST answer the invalid argument error class for
that request and MUST keep the connection open. Reporting a
request-scoped failure states what each peer does to answer.

Frame Admission Order places this check at step 14.

The rule gives a set of entries exactly one encoding. Without it a region
carrying two entries would have two byte sequences that decode to the same
content, neither more correct than the other, and no fixture could state
which one an encoder produces. It costs an encoder that assembles entries
out of order one sort before it writes them, and it costs a receiver one
comparison against a value it has already read.

The order runs over the whole identifier, so every optional entry of a
region precedes every required one. That follows from the rule and from
the ranges below; it is not a separate requirement.

## Metadata Identifiers

`identifier` is a `u16`. Bit 15 states the class of the entry, and the two
values of that bit divide the whole domain:

| Range | Bit 15 | Class | The rule for an identifier this revision does not assign |
|---|---|---|---|
| `0x0000..0x7FFF` | clear | optional | The Optional Range |
| `0x8000..0xFFFF` | set | required | The Required Range |

A receiver computes the class from the identifier alone: the entry is
required when `identifier & 0x8000` is non-zero and optional otherwise. It
reads no other field of the entry and holds no state to decide.

This revision assigns identifiers `0x0001`, the deadline, and `0x0002`,
the request class, in the optional range. The Assigned Identifiers states
their encoding and the meaning of their values.

A peer MUST NOT send an entry whose identifier this revision does not
assign. A receiver that meets one applies the rule of the range the
identifier lies in: The Optional Range for an optional identifier, and The
Required Range for a required one.

The two rules below are the whole of what a receiver does with a region
whose entries this revision does not assign. A peer implementing a later
revision has entries to send, and a receiver built on this revision meets
them.

`0x0000` is an unassigned optional identifier and carries the optional
range's rule. No identifier is reserved as never valid: both rules leave
the connection usable, so no value in this domain needs a rule that ends
it.

### The Assigned Identifiers

| Identifier | Range | Entry | Value | Gated by |
|---|---|---|---|---|
| `0x0001` | optional | the deadline | four bytes, a little-endian `u32` count of microseconds | the deadlines capability |
| `0x0002` | optional | the request class | one byte, one of four assigned values | the request classes capability |

An entry carrying identifier `0x0001` is a deadline. The entry is optional:
a receiver that does not read it serves the frame without it. Request
Lifetime states what the value means, the instant the duration runs from,
and what a responder does with it.

The entry's value is exactly four bytes. A receiver that meets an entry
carrying identifier `0x0001` whose value length is not four bytes MUST
treat the entry as one this revision does not read: it skips the entry by
the rule of The Optional Range, produces no failure, and serves the
request without a deadline.

Metadata owns the identifier and the encoding. Request Lifetime owns the
meaning of the value and the behavior a deadline produces, and
Capabilities owns the capability that gates the entry.

### The Request Class Entry

| Value | Class |
|---|---|
| `0x00` | latency |
| `0x01` | normal |
| `0x02` | bulk |
| `0x03` | background |
| `0x04..0xFF` | unassigned |

An entry carrying identifier `0x0002` is a request class: one byte that
states the initiator's preference for how the responder schedules the
request. The entry is optional: a receiver that does not read it serves
the request at its own default.

A request class is a preference and never a guarantee. Server policy wins
over the client preference. A client MUST NOT depend on a request class to
obtain a guarantee, because no peer is required to honour it and a
responder that schedules the request another way is conforming.

Absence of the entry states the normal class. An absent entry is not the
same as an entry whose value is `0x01`: the two states are not
interchangeable, and a receiver MUST treat a request carrying no entry as
the normal class.

The entry's value is exactly one byte. A receiver that meets an entry
carrying identifier `0x0002` whose value length is not one byte MUST treat
the entry as one this revision does not read: it skips the entry by the
rule of The Optional Range, produces no failure, and serves the request at
the responder's default.

A receiver that meets the entry carrying a value in `0x04..0xFF` MUST skip
the entry, MUST serve the request at the responder's default, and MUST NOT
produce a failure. The value is unassigned, and a receiver that produces a
failure for a preference it does not assign would refuse a request it can
serve.

Metadata owns the identifier and the encoding. Capabilities owns the
capability that gates the entry.

### The Optional Range

An entry whose identifier is in `0x0000..0x7FFF` is optional. Its sender
states that a receiver which does not recognise the identifier can serve
the frame without it.

A receiver that meets an identifier in this range that this revision does
not assign MUST skip the entry and MUST NOT produce an error. It resumes
at the next entry, `4 + value length` bytes from the first byte of the
entry it skipped, and continues to apply Entry Order and Region Fill to
the rest of the region.

Skipping an entry is not the same as stopping at it. A receiver walks
every entry of the region, whatever it does with each one.

A later revision assigns an identifier in this range at no cost to a peer
built on this one: that peer meets the entry it does not know, skips it,
and serves the frame. What the range cannot express is a demand. A sender
that needs a receiver to act on an entry learns nothing from an answer
that would be identical if the entry had been skipped.

### The Required Range

An entry whose identifier is in `0x8000..0xFFFF` is required. Its sender
states that a receiver which does not recognise the identifier cannot
serve the frame.

A receiver that meets an identifier in this range that this revision does
not assign MUST answer the invalid argument error class for that request
and MUST keep the connection open. Reporting a request-scoped failure
states what each peer does to answer.

Frame Admission Order places this check at step 15.

A later revision assigns an identifier in this range at the cost of one
refused request per frame that carries it to a peer built on this one. The
connection survives, the receiver reports a class the sender can name, and
no frame is lost in silence.

At this revision a server reaches this check only on a REQUEST frame
carrying the handshake opcode. Frame Admission Order places the `code`
check at step 13, and a REQUEST frame carrying any other opcode is refused
there, so the handshake request is the only request that reaches step 15.
A client reaches this check on a RESPONSE or an ERROR frame carrying a
region.
