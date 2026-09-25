---
title: Correlation
description: Request identity: the field that names a request, the peer that allocates it, its reserved value, the state each peer holds for it, the frame that retires a request, and what a receiver does with a frame it cannot correlate.
protocol_version: 0
revision: v0.6.0
status: draft
order: 4
---

# Correlation

## Scope

This document defines request identity and correlation: the field that
names a request, the peer that allocates it, the values it carries, the
state each peer holds for it, the frame that retires a request, and what a
receiver does with a frame whose request id it cannot match.

Frame Header owns the position and the width of the `request id` field.
This document owns what the field's values mean and what a receiver does
with each of them.

It states the three checks Frame Admission Order places at steps 10, 11
and 12. Each is stated once, over every frame kind, rather than once per
kind.

Handshake defines the exchange that opens a connection. This revision
assigns the handshake opcode and the five ungated operations, so the
requests it describes name the work those operations perform. What This
Revision Reaches states which of the rules below a conforming peer meets at
this revision.

It defines no bound on the number of requests in flight at a peer. Limits
states the size bounds this revision fixes, and none of them counts
requests.

## Request Identity

`request id` names the request a frame belongs to. Frame Header states
that it is a `u64` and that every frame of every kind carries it.

Two roles belong to each request. They are properties of the request and
not of the connection:

| Role | The peer that holds it |
|---|---|
| initiator | the peer that allocates the request id and sends the frame that opens the request |
| responder | the peer that receives that frame and sends the request's terminal frame |

REQUEST is the only frame kind this revision assigns that opens a request,
and Frame Kinds states that it travels from the client. At this revision
the client is therefore the initiator of every request and the server is
the responder for every request.

The initiator allocates the request id. A responder MUST NOT allocate one.
A responder that allocated one would send a frame naming a request that is
not in flight at its peer, and A request id not in flight states what a
receiver does with such a frame.

A request id is unique among the requests in flight on one connection. It
is not unique over the life of that connection, and Reusing a Request Id
states when an initiator may allocate one again.

The uniqueness runs over one connection and no further. Each peer begins a
connection holding no request in flight, a request id an initiator
allocated on one connection names nothing on another, and no state this
document defines survives the connection that carried it.

## The Reserved Request Id

Request id 0 is reserved. It states that the frame carrying it belongs to
no request.

A valid request id is any value from 1 to 18446744073709551615. An
initiator MUST NOT allocate 0, because every frame it then sent for that
request would carry 0 and the rule below refuses each of them.

Whether a frame names a request follows from its kind:

| Kind | Does the frame name a request? |
|---|---|
| REQUEST | always: the request it opens |
| RESPONSE | always: the request it answers |
| ERROR | The request id an ERROR frame carries states when |
| WITHDRAWAL | always: the request it withdraws |

A peer MUST NOT send a frame that names a request and carries request id
0. A receiver that meets one MUST treat the frame as a protocol violation
and close the connection. Frame Admission Order places this check at step
10.

A frame whose whole purpose is to open, answer, or report on a request
cannot belong to none. The combination is meaningless rather than
ambiguous, so the contract forbids it rather than interpreting it.

The rule runs over every frame kind rather than once per kind. A revision
that assigns a frame kind carrying a meaning for request id 0 states that
kind's row in the table above and restates nothing else here.

ERROR is the one kind whose row depends on another field of the same
header. A receiver reads the error class from `code` and applies the rule
to an ERROR frame whose class is request-scoped. An ERROR frame carrying
an error class this revision does not assign is refused at step 13 as a
protocol violation that closes the connection, which is the class and the
scope step 10 produces, so the order between the two steps changes no
outcome for such a frame.

## In Flight

A request is in flight at a peer while that peer owes the request's
terminal frame or is owed it.

A request id is in one of two states at a peer. It is not in flight when
the peer holds no request under it, and in flight when the peer holds one.
The transitions between the two states are:

| Role | The request enters flight | The request leaves flight |
|---|---|---|
| initiator | when the initiator has written the whole frame that opens it | when the initiator has read the request's terminal frame |
| responder | when the responder has carried the frame that opens it through step 12 of Frame Admission Order | when the responder has written the whole terminal frame that retires it |

Every peer begins a connection with every request id not in flight, and a
request that leaves flight returns its request id to that state. There is
no third state. A peer holds nothing that separates a request id whose
request has left flight from one no request on this connection has used,
and A request id not in flight states what follows from that.

Step 12 is where a responder's set gains the request because it is the
last step whose failure is connection-fatal for a frame that opens one.
Every step through 12 that refuses such a frame closes the connection, so
the request never enters flight and nothing answers it. Every step after
12 that refuses one refuses it for that request alone, and the responder
answers with a terminal frame naming the request, which it can only do for
a request in flight at it.

In flight is defined by frame exchange. No elapsed time enters either
transition, and no peer reads a clock to decide whether a request is in
flight.

Each peer holds the set of requests in flight at it. That set is what the
two checks of Correlating a Frame read.

## Terminal Frames

A terminal frame retires the request it names. The request leaves flight,
and no further frame names it.

| Kind | Terminal |
|---|---|
| REQUEST | no |
| RESPONSE | yes |
| ERROR carrying a request-scoped error class | yes |
| ERROR carrying a connection-fatal error class | no |

Failure Scope states that a frame carrying a connection-fatal class
retires no request and that its sender closes the connection after sending
it. A Connection That Ends states what becomes of a request in flight when
that happens.

A responder MUST send exactly one terminal frame for each request in
flight at it.

A responder that sends none leaves the request in flight at its initiator
for as long as the connection lasts. This revision defines no timeout.
Request Lifetime defines the withdrawal frame, which stops one request but
produces no frame of its own, so only the request's terminal frame or the
connection ending retires it.

A responder that sends a second sends a frame naming a request that is not
in flight at it. A request id not in flight forbids that frame, and a
receiver that meets it refuses it as a protocol violation and closes the
connection. The second terminal frame therefore needs no rule of its own.

## Correlating a Frame

A receiver correlates a frame by the request id the frame carries, against
the set of requests in flight at the receiver. Two checks read that set,
and which one applies follows from what the frame does with the request it
names:

| Kind | What the frame does with the request it names |
|---|---|
| REQUEST | opens it |
| RESPONSE | names a request already in flight |
| ERROR that names a request | names a request already in flight |
| WITHDRAWAL | names a request, whether or not it is in flight at the receiver |

A frame carrying request id 0 names no request, so neither check applies
to it.

The withdrawal row is the one entry that does not require the request to be
in flight. A request id not in flight states the exception below.

### A request id already in flight

An initiator MUST NOT open a request whose request id is in flight at the
initiator. A receiver that meets a frame opening a request whose request
id is already in flight at the receiver MUST treat the frame as a protocol
violation and close the connection. Frame Admission Order places this
check at step 11.

Two requests in flight under one request id make every terminal frame that
names it ambiguous, and no field of any later frame resolves the
ambiguity.

### A request id not in flight

A peer MUST NOT send a frame that names a request that is not in flight at
that peer. A receiver that meets a frame naming a request that is not in
flight at the receiver MUST treat the frame as a protocol violation and
close the connection. Frame Admission Order places this check at step 12.
The rule binds both peers, and a receiver applies it to a frame arriving
from either direction.

The withdrawal frame is the one exception. A peer MAY send a withdrawal
for a request id the receiver does not hold, and a receiver MUST NOT treat
it as a failure. Request Lifetime states the withdrawal's behavior in
full: a withdrawal naming a request id the receiver does not hold stops
nothing, produces no frame, and is not an error.

Two conditions produce such a frame, and this contract does not
distinguish them:

```text
the request id was never in flight at the receiver
the request id was in flight at the receiver and has left flight
```

A receiver holds no record of a request that has left flight. Such a
record grows with every request the peer sends and has no bound the
receiver controls, so a receiver that kept one would hold unbounded state
its peer chooses the size of. The two conditions therefore reach the
receiver as one input: a frame naming a request id the receiver does not
hold. A contract that gave them different rules would require of a
receiver a distinction no conforming receiver can make.

The condition is not the failure of a request. The request the frame names
either never existed or has already retired and been reported, so there is
no request to fail. It states that the two peers hold different sets of
requests in flight, and that every frame that follows is correlated
against a set the receiver does not share. Ignoring the frame would leave
that disagreement in place, and would strand a request permanently when
the ignored frame was the one that would have retired it.

One frame that violates the sender obligation therefore ends every request
in flight on that connection, including requests that would have
completed. That cost is paid only against a peer that violated it.

## Reusing a Request Id

An initiator MAY reuse a request id once the request that carried it has
left flight at the initiator. A request id already in flight states what
an initiator may not do before that.

A receiver MUST NOT refuse a frame that opens a request on the ground that
its request id names a request the receiver has already retired. A
receiver that refused one would hold the record of retired request ids
that A request id not in flight states no receiver holds.

An initiator that allocates a request id it has not used before never
reuses one within the life of a connection, and needs no reuse logic at
all. The field is 64 bits wide.

## Correlation Is by Request Id Alone

A receiver MUST match a frame to a request by the request id the frame
carries. A receiver MUST NOT match a frame to a request by the order the
frame arrived, by the order the requests were sent, or by any boundary the
transport delivered the bytes on.

A responder MAY send the terminal frames of the requests in flight at it
in any order, and that order is not required to be the order the requests
arrived. A responder is not required to serve the requests in flight at it
in the order it received them, and the frames it sends for them may
interleave. An initiator MUST accept a terminal frame for any request in
flight at the initiator, whatever the order it sent those requests in, and
MUST accept the frames of several requests in flight at it in any
interleaving.

A peer that matched by arrival order retires a request against a frame
that belongs to another request, and reports that frame's content as the
first request's answer. Nothing on the wire tells its peer that it did:
the bytes are the bytes a conforming peer sent, and the failure is local
to the peer that made it.

## A Connection That Ends

A connection ending retires no request. Every request in flight at a peer
when the connection ends receives no terminal frame.

A receiver holding part of a frame when the connection ends retires no
request with those bytes. Frame Admission Order states that steps 1 and 7
are decoder states and not failures; a connection ending at either of them
leaves the receiver holding bytes that are not a frame, and bytes that are
not a frame name nothing.

An initiator MUST NOT conclude from a connection ending that the work a
request named took effect, and MUST NOT conclude that it did not. A
request that ends with no terminal frame carries no completion certainty:
each error class states what a caller may conclude about a mutation, and a
request that received no class received no statement. An initiator that
must know re-reads what the request would have changed, on a new
connection.

This revision assigns the handshake opcode and the five ungated operations.
The rule is stated here because it is a property of the correlation
mechanism and not of any operation. It holds for every operation a later
revision assigns, and an initiator built on this revision that concluded
otherwise would be wrong from the first one.

## What This Revision Reaches

A REQUEST frame carrying the handshake opcode is the handshake request.
Before the Handshake states that it is legal only in the pre-negotiation
state, and that a REQUEST frame carrying any other opcode in that state is
a protocol violation.

Frame Admission Order places the `code` check at step 13. In the
negotiated state, a server answers a REQUEST frame carrying an opcode this
revision does not assign with the unsupported operation error class for
that request and keeps the connection open. That ERROR frame is the
request's terminal frame.

At this revision, therefore:

```text
a request enters flight at a client when the client has written its
  REQUEST frame, and leaves flight when the client has read its terminal
  frame
a request enters flight at a server when the server has carried the
  REQUEST frame through step 12 of Frame Admission Order, and leaves
  flight when the server has written the terminal frame that retires it
the handshake request is answered by a RESPONSE frame carrying the
  success result code, or by an ERROR frame when the handshake fails
each of the five ungated operations is answered by a RESPONSE frame
  carrying a result code, or by an ERROR frame
a REQUEST frame carrying an opcode this revision does not assign is
  answered by an ERROR frame carrying the unsupported operation class
no conforming server sends a RESPONSE frame for an opcode this revision
  does not assign
```

A client implements the rules above for a RESPONSE frame whether or not a
conforming server sends one at this revision. A client that received one
would have to decide what to do with it, and this document is where it
reads the decision.
