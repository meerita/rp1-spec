---
title: Scope and Status
description: What revision v0.1.1 of protocol version 0 defines and does not define, with the requirement levels, version axes, terminology, and limitations of the document set.
protocol_version: 0
revision: v0.1.1
status: draft
order: 1
---

# Scope and Status

## Scope

This document set defines protocol version 0 of the RP-1 Native Protocol,
at revision v0.1.1.

It defines the framing and codec surface: how a frame is laid out on the
wire, what every value a frame carries means, and how a receiver admits or
refuses one.

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
the receiver behavior for every input this surface permits
the extension and version policy for every value space it defines
```

This revision does not define:

```text
the handshake, or any other exchange that opens a connection
negotiation, or any negotiated value
the capability registry, and it assigns no capability identifier
any operation, and it assigns no opcode
conformance layers, role obligations, and any required minimum
```

An engineer implementing this revision alone writes an encoder and a
decoder for every frame kind it assigns, and receiver behavior for every
input it permits.

That is the whole of what this revision produces. It defines no exchange,
so an implementation of it does not interoperate with a peer and is not a
client. Known Limitations states what that costs and what an implementer
does in the meantime.

Revision v0.1.1 publishes the documents of this set and the fixture corpus
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
is required to be UTF-8, and the protocol interprets neither. This
revision assigns no opcode, so no frame region it defines carries a key or
a value; the two terms are fixed here because the meanings this revision
states for assigned wire values use them.

## Known Limitations

A limitation stated here is a property of this contract. It is not a
statement about what any implementation has built.

This revision defines framing and no exchange. Two peers that implement it
completely still cannot complete a request, because no clause of this
revision defines a request being made or answered. An implementation of
this revision is a codec: it is not a client, and it does not interoperate
with a deployed peer.

### No handshake

What is limited: this revision defines no exchange that opens a
connection. A peer has no defined way to announce itself, and no defined
way to learn what the peer it connected to implements.

What this contract provides: the frame format, and the bounds that apply
with no prior exchange, so a decoder is complete without a handshake.

What an implementer does: implements the codec, and derives no connection
opening sequence from this revision. An implementer who needs to reach a
deployed peer waits for the revision that defines the exchange.

What would remove it: a revision that defines the exchange that opens a
connection.

### No negotiation

What is limited: no value this contract carries is negotiated. Two peers
cannot agree on a larger frame size, and cannot agree on any other
parameter.

What this contract provides: bounds that are constants of this revision
and bind both peers from the first byte of the connection, with no
exchange before them.

What an implementer does: enforces the bounds this revision states, and
implements no negotiated value. The section that states each bound states
what a receiver does with a frame that exceeds it.

What would remove it: a revision that defines negotiation, the negotiated
bound, which peer proposes it, and which peer sets its ceiling.

### No capability

What is limited: this revision assigns no capability identifier, and
defines no mechanism to offer one or to accept one.

What this contract provides: every behavior it defines is available on
every connection, gated by nothing.

What an implementer does: implements no gated behavior, and treats a value
this revision does not assign by the receiver rule stated in the section
that owns that value space.

What would remove it: a revision that defines negotiation together with
the capability registry, which one section owns.

### No operation

What is limited: this revision assigns no opcode, so it names no work a
server performs and no payload a request carries.

What this contract provides: the frame that carries an operation, and the
receiver rule for an opcode this revision does not assign, so a later
revision assigns one without invalidating a peer built on this one.

What an implementer does: implements the frame, and derives no operation
from this revision. An implementer who needs an operation waits for the
revision that assigns one.

What would remove it: a revision that assigns an opcode and states its
payload in both directions.

### No conformance definition

What is limited: this revision defines no conformance layer, no role
obligation, and no required minimum. It states no criterion an
implementation would be measured against, and no form a conformance claim
takes.

What this contract provides: the normative fixture corpus published with
this revision, which an implementation runs against its own encoder and
decoder.

What an implementer does: states which fixtures of this revision's corpus
its implementation passes, rather than claiming conformance to the
revision.

What would remove it: a revision that defines the conformance layers, what
each one requires, and the form a conformance claim takes.
