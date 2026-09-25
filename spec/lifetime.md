---
title: Request Lifetime
description: The deadline a request may carry, its anchor, its enforcement, and the completion certainty of a request that exceeds it.
protocol_version: 0
revision: v0.6.0
status: draft
order: 10
---

# Request Lifetime

## Scope

This document defines the deadline a request may carry: what the value
means, the instant the duration runs from, what a responder does when the
duration has passed, and the completion certainty of a request that
exceeds it.

It does not define the metadata region or the entry layout, which Metadata
owns, or the error class a deadline produces, which Failures owns.
Metadata assigns the deadline entry and states its encoding, and this
document states what the entry means and what a responder does with it.
Capabilities states the capability that gates the entry and the behavior
of a connection without it.

An engineer implementing this document alone can encode and read a
deadline, bound a request by it, and answer the deadline exceeded class
when the bound is exceeded.

## Deadline

A request may carry a deadline. Metadata assigns identifier `0x0001` to
the deadline entry, states that its value is a little-endian `u32`, and
states that the entry is optional and gated by the deadlines capability.
This section states what the value means.

The value is a duration in microseconds, measured from the anchor below.
Every value of the width is legal. Zero states a deadline that has already
passed at admission. The widest value, 4294967295, is about 71.6 minutes.

The anchor is responder-local and is not carried on the wire. The anchor
is the instant the responder reads the frame. A responder that reads
several frames together MAY anchor every request of that read at the start
of the read, which expires a request no later than the read of its own
frame and never later. An initiator MUST NOT assume an anchor later than
the read of its own frame, because no peer can predict the responder's
read granularity.

The deadline applies to every request that carries one, whatever operation
the request names. A responder that has not begun the work a request names
when the deadline has passed MUST answer the deadline exceeded error class
for that request, MUST perform no work for it, and MUST keep the
connection open. A request that produced its answer before the deadline
passed keeps that answer, and the responder MUST NOT answer the deadline
exceeded class for it.

A responder that answers the deadline exceeded class for a request has
applied no mutation, and Failures states that completion certainty at the
class.

### Absence and Zero

Absence of the deadline entry is not a deadline of zero.

A request carrying no deadline entry MUST be served without a deadline.

An entry carrying identifier `0x0001` whose value length is not four bytes
is an optional entry this revision does not read. Metadata states the rule
and this document does not add one: a receiver skips such an entry,
produces no failure, and serves the request without a deadline. A receiver
MUST NOT treat an entry of the wrong value length as a deadline.

A request carrying a deadline of zero carries a present entry whose value
is four zero bytes. Its deadline has passed at admission: a responder MUST
answer the deadline exceeded class for it, MUST perform no work for it,
and MUST keep the connection open. The frame that carries it is not the
same frame as one that carries no entry, and the two states are not
interchangeable.
