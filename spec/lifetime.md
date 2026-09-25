---
title: Request Lifetime
description: The deadline a request may carry, its anchor and enforcement, the withdrawal frame, the race between a withdrawal and the work, and the completion certainty of each outcome.
protocol_version: 0
revision: v0.6.0
status: draft
order: 10
---

# Request Lifetime

## Scope

This document defines the request lifetime surface: the deadline a request
may carry, what its value means, the instant the duration runs from, what a
responder does when the duration has passed, and the completion certainty
of a request that exceeds it; and the withdrawal frame, what a withdrawal
does to the request it names, the race between a withdrawal and the work,
and the completion certainty of each outcome.

It does not define the metadata region or the entry layout, which Metadata
owns; the frame kind and its control code, which Framing owns; the
capability that gates the withdrawal and the deadline, which Capabilities
owns; or the error classes a deadline and a withdrawal produce, which
Failures owns. This document states what each surface means and what a
responder does with it.

An engineer implementing this document alone can encode and read a
deadline, bound a request by it, send and receive a withdrawal, decide
what a request produces when it races a withdrawal, and state the
completion certainty of every outcome.

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

## Withdrawal

The withdrawal frame is frame kind `0x07`. The client sends it and the
cancellation capability gates it. Framing states its kind, its direction,
and the rule for its `code` field; this section states what it does.

A withdrawal names the request to withdraw in its `request id` field. Its
payload is empty and its metadata region is empty. A receiver MUST check
both for emptiness before it applies Entry Order and the Required Range to
the region. A withdrawal frame carrying a non-empty payload or a non-empty
metadata region is a malformed request, and a receiver that meets one MUST
close the connection. Frame Admission Order places this check at step 8,
before the two request-scoped metadata checks. The frame produces no frame
of its own, so a request-scoped failure would have no request to answer and
no frame to carry the answer, and the connection-fatal scope is the one
that needs no request to answer.

A withdrawal produces no frame of its own. The request it names produces
exactly one terminal frame, by the rule Terminal Frames states.

A withdrawal naming request id 0 is a protocol violation: the frame names
a request, and The Reserved Request Id forbids a frame that names a request
from carrying 0. A receiver that meets one MUST treat the frame as a
protocol violation and close the connection.

A withdrawal naming a request id the receiver does not hold stops nothing,
produces no frame, and is not an error. Correlation states that a receiver
holds no record of a request that has left flight, so a request that has
already retired and a request that never existed reach the receiver as one
input. A withdrawal is not subject to the rule A request id not in flight
states, in either direction: a peer MAY send one for a request id the
receiver does not hold, and a receiver MUST NOT treat it as a failure.

A repeated withdrawal is the same as the first: the request is stopped
once and answers once. A later withdrawal naming a request id no longer in
flight is the unheld case above.

A withdrawal names a request whatever work the request is doing, and the
race below states what the request produces in each case.

## The Race Rule

A withdrawal and the work of the request it names race. Exactly one
terminal frame retires the request in every outcome:

| The state at the withdrawal | The terminal frame |
|---|---|
| the work committed before the withdrawal | the work's own RESPONSE frame |
| the work had not committed when the withdrawal was observed | ERROR `0x0007` cancelled |
| the request already retired | none: the request id is unheld |
| the request never existed | none: the request id is unheld |

The commit wins. A withdrawal never reports a write that ran, and a
withdrawal never rolls a committed mutation back. ERROR `0x0007` cancelled
states that the request produced no result; any other terminal frame states
the true outcome.

A conforming initiator MUST accept either the ordinary terminal frame or
the cancelled class for a request it withdrew, and MUST NOT require the
cancelled class after it sends a withdrawal. An initiator that required the
cancelled class would wait for an outcome the protocol never sends for a
request that committed.

The same rule reads from the deadline side: a responder that has committed
a write and then observes the deadline answers the committed write's own
RESPONSE frame, because the request already produced its result. Deadline
states that a request that produced its answer keeps it.

## Completion Certainty

A read makes no mutation, so a cancelled read leaves nothing behind and
answers the cancelled class.

A write is stopped before it commits or it is not stopped at all. The
completion certainty of each outcome is:

| The terminal frame | What a caller may conclude |
|---|---|
| ERROR `0x0007` cancelled | the mutation was not applied |
| ERROR `0x0006` deadline exceeded | the mutation was not applied |
| any RESPONSE frame | the true outcome, which may have committed |
| the connection ending | no completion certainty |

Neither the cancelled class nor the deadline exceeded class ever reports a
write that ran, and neither rolls a committed mutation back. A request that
committed before the withdrawal or the deadline keeps its own response. A
connection that ends retires no request and carries no completion
certainty, as A Connection That Ends states.

Failures states the completion certainty of the cancelled class and of the
deadline exceeded class at each class, and this section states it for the
surface as a whole.
