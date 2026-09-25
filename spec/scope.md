---
title: Scope and Status
description: What revision v0.6.0 of protocol version 0 defines and does not define, with the requirement levels, version axes, terminology, and limitations of the document set.
protocol_version: 0
revision: v0.6.0
status: draft
order: 1
---

# Scope and Status

## Scope

This document set defines protocol version 0 of the RP-1 Native Protocol,
at revision v0.6.0.

It defines the framing and codec surface: how a frame is laid out on the
wire, what every value a frame carries means, and how a receiver admits or
refuses one. It defines the connection surface: the states a connection
occupies, the handshake that moves a connection to a usable state, version
negotiation, the limits the handshake derives, and the capability
mechanism. It defines the operation surface: the five ungated operations,
the request and response payload of each, and the argument encoding they
share.

This revision defines:

```text
byte order
the frame header, every one of its fields, and the meaning of the code
  field for each frame kind
the frame kinds it assigns, and the direction each travels
the flags field and its reserved bits
the total length arithmetic, the width it is computed in, and the point
  the check runs relative to allocation
the metadata region
request identity and correlation
the frame size bounds a peer decodes under, and where they are enforced
the result codes it assigns
the error classes it assigns, each with its scope and its completion
  certainty
failure scope, and the principle that decides it
the order in which a receiver admits a frame, and that the first failure
  wins
the connection states, which frames are legal in each, and the failure a
  frame outside them produces
the handshake that opens a connection, both payloads
version negotiation, and the outcome when no version is mutually supported
the negotiated maximum frame size and maximum metadata size, with their
  floors and their ceilings
the opcode registry, the five ungated operations, and the request and
  response payload of each
the argument encoding the operations share, and the validation each runs
the capability identifier domain, the capability entry layout, and
  capability negotiation, assigning no capability identifier
the receiver behavior for every input this surface permits
the extension and version policy for every value space it defines
```

This revision does not define:

```text
a capability identifier, and gated behavior: the registry mechanism is
  defined and the whole identifier domain is unassigned
any capability-gated operation, and any operation that needs a capability
conformance layers, role obligations, and any required minimum
```

An engineer implementing this revision alone writes an encoder and a
decoder for every frame kind it assigns, completes the handshake, reaches a
usable connection, and runs the five ungated operations.

This revision defines a usable connection and the five ungated operations.
Known Limitations states what the revision leaves for a later one and what
an implementer does in the meantime.

Revision v0.6.0 publishes the documents of this set and the fixture corpus
that accompanies them. A fixture carries the same authority as the prose
it exercises. Supporting material states that it is not normative.

## Status

Every document of this document set carries the status `draft`.

A stable contract changes only through a classified change that states
whether a peer built on the previous revision still conforms. A draft
contract changes without that treatment, and an implementation built
against a draft revision is built at the implementer's own risk. A
document moves from `draft` to `stable` once, and never moves back.

A protocol version alone does not identify a contract, because one
revision of a protocol version may state a requirement an earlier revision
of the same protocol version left unstated. The revision is the identity
an implementation pins.

## Requirement Levels

The key words "MUST", "MUST NOT", "REQUIRED", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only
when, they appear in all capitals, as shown here.

This document set is one contract, and the interpretation above applies to
every document in it.

A keyword in lowercase carries no requirement level. It is ordinary
English and binds nobody.

The two documents this section adopts:

```text
[RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate Requirement
           Levels", BCP 14, RFC 2119, March 1997.
[RFC8174]  Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key
           Words", BCP 14, RFC 8174, May 2017.
```

## Version Axes

Three versions evolve independently of one another:

| Axis | What it names |
|---|---|
| protocol version | the wire contract one document set defines |
| server version | a release of the RP-1 server |
| SDK version | a release of a client library |

None of the three is derivable from another. A server version does not
state which protocol version a connection carries, and a protocol version
does not state which server version serves it.

A revision is one published state of this specification. It identifies the
contract, not the protocol version: a revision may state a protocol
version more precisely without changing it, and a change to a protocol
version always produces a new revision.

## Terminology

Each term below is defined here because this document is where it is first
used. A term that names a subject another section of this document set
defines is defined in that section, and is not restated here.

| Term | Meaning |
|---|---|
| frame | the unit of transfer on the wire |
| peer | either endpoint of a connection |
| client | the peer that opens the connection |
| server | the peer that accepts it |
| connection | one transport session, and the state the two peers keep for it |
| sender | the peer that writes a frame |
| receiver | the peer that reads a frame |

A key is an opaque byte string. A value is an opaque byte string. Neither
is required to be UTF-8, and the protocol interprets neither. The five
ungated operations carry them, and Operations states the encoding.

## Known Limitations

A limitation stated here is a property of this contract. It is not a
statement about what any implementation has built.

This revision defines a usable connection and the five ungated operations.
No capability is assigned, so no gated operation exists, and no operation
in this revision needs one. An implementation of this revision connects,
negotiates, and runs the five operations.

The first two limitations below follow from the absent capability. The last
does not: it states which requirement of this revision no fixture of its
corpus can fail against.

### No capability assigned

What is limited: this revision defines the capability mechanism but
assigns no capability identifier, so no gated behavior exists and no
capability can be exercised beyond being offered and accepted.

What this contract provides: the capability identifier domain, the
capability entry layout, the receiver rule for an unassigned identifier,
the rule for a value of the wrong length, the rule that a responder never
accepts a capability the offerer did not offer, and the dependency rule.

What an implementer does: offers no identifier, accepts none, and treats
an identifier it does not assign by the ignore rule the section that owns
the capability space states.

What would remove it: a revision that assigns a capability identifier
together with the surface the identifier gates.

### No conformance definition

What is limited: this revision defines no conformance layer, no role
obligation, and no required minimum. It states no criterion an
implementation would be measured against, and no form a conformance claim
takes.

What this contract provides: the normative fixture corpus published with
this revision, which an implementation runs against its own encoder,
decoder, and handshake.

What an implementer does: states which fixtures of this revision's corpus
its implementation passes, rather than claiming conformance to the
revision.

What would remove it: a revision that defines the conformance layers, what
each one requires, and the form a conformance claim takes.

### No fixture for a conclusion an initiator draws

What is limited: one requirement of this revision has no fixture that can
fail against it. A Connection That Ends binds an initiator not to conclude
from a connection ending that the work a request named took effect, and
not to conclude that it did not. A fixture offers bytes and states the
outcome a receiver produces from them. This requirement binds what an
initiator concludes when no bytes arrive at all, and no byte sequence
distinguishes an initiator that obeys it from one that does not.

What this contract provides: every requirement whose violation changes
what a peer emits, accepts or refuses carries a fixture that can fail
against it. The corpus also covers the observable half of this one: bytes
that are not a frame retire no request, and a fixture states that for a
receiver holding a partial frame with a request in flight.

What an implementer does: implements the requirement from the prose, and
does not read a complete pass of this corpus as evidence of conforming to
it. A request that ends with no terminal frame received no error class, so
it received no statement about a mutation, and an initiator that reports
one is wrong whatever the corpus says.

What would remove it: a revision that defines the conformance layers, and
with them how a requirement no fixture reaches is claimed and checked.
