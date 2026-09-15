# Changelog

This file records changes to the RP-1 Native Protocol contract.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

A revision number tracks the contract:

```text
patch   editorial, clarification
minor   addition, or a defect correction that leaves peers conforming
major   breaking change
```

The revision is not the protocol version. A revision may clarify a
protocol version without changing it, and a protocol version change
always produces a new revision.

Each entry states what changed, whether a peer built against the previous
revision still conforms, and what such a peer must do when it does not.

## [0.1.1]

Classification: clarification. No requirement is added, removed or
altered. A peer built against revision v0.1.0 conforms to this revision
unchanged and has nothing to do.

This revision closes the distance between what the contract requires and
what its corpus can check. Three requirements of v0.1.0 had no fixture
that could fail against them. Two now do, and the third is stated as a
limitation of the contract rather than left for a reader to discover.

### Added

- `errors/text-that-is-not-valid-utf8`. The text region of an ERROR frame
  carries bytes no UTF-8 decoder accepts, and the expected outcome is the
  frame decoded and its error class delivered. The ERROR Frame Payload
  requires that a receiver does not parse the text, and no fixture could
  fail against a receiver that did.
- `errors/text-that-contradicts-the-class`. The text reads "ok" on a frame
  carrying the unsupported operation class. It differs from
  `errors/detail-length-zero-with-text` only in the text region and in the
  payload length that measures it, and both expect the same class, the
  same scope and the same fields. A receiver that branches on the content
  of the text answers the two differently.
- `correlation/partial-frame-retires-no-request`. Nineteen bytes of a
  frame that would name a request the receiver holds in flight, expecting
  the incomplete state, the bytes the receiver requires, and that it
  retires nothing. A Connection That Ends requires that bytes which are
  not a frame retire no request.
- A sixth known limitation in Scope and Status, naming the one requirement
  of this revision that no fixture reaches: that an initiator concludes
  nothing from a connection ending about whether the work a request named
  took effect. A fixture offers bytes and states the outcome a receiver
  produces from them, and this requirement binds what an initiator
  concludes when no bytes arrive at all. The limitation states what the
  contract provides in its place, what an implementer does, and what would
  remove it.

### Changed

- The corpus form states that an `incomplete` fixture carries `retires`
  when its input states `in_flight`, always `0`. The member is carried
  rather than left implicit so that a receiver which retires a request on
  a partial frame fails the fixture instead of passing it. No fixture of
  the previous revision is altered by it: none of the five `incomplete`
  fixtures states `in_flight`.
- Every normative document and every fixture states revision v0.1.1. The
  corpus of revision v0.1.0 is preserved at the tag that published it.
- The corpus is 66 fixtures.

### Unchanged

Every clause of protocol version 0. The frame header, the frame kinds, the
admission order, the metadata region, correlation, the opcode space, the
result codes, the error classes, the bounds and the extension policy are
byte for byte the contract revision v0.1.0 published. Every document still
carries the status `draft`.

## [0.1.0]

The first revision of this specification. It defines protocol version 0
of the RP-1 Native Protocol at the framing and codec surface: how a frame
is laid out on the wire, what every value a frame carries means, and how
a receiver admits or refuses one.

An engineer who reads this revision and nothing else writes an encoder
and a decoder for every frame kind it assigns, and receiver behavior for
every input it permits. The revision defines no exchange, so what it
produces is a codec: it is not a client, and it does not interoperate
with a deployed peer.

Every document of this revision carries the status `draft`. A draft
contract changes without the treatment a classified change carries, and
an implementation built against it is built at the implementer's own
risk.

### Compatibility

No previous revision exists, so no peer was built against one. No
compatibility statement applies to this entry and no change class
describes it, because every class is defined relative to a previous
revision. Every later revision that touches a value space this one
defines carries a classification against it.

### Added

The normative documents of protocol version 0, in reading order:

- Scope and Status. What the revision defines and does not define.
  The requirement levels, adopted from RFC 2119 and RFC 8174, applying
  to every document of the set. The three version axes, none derivable
  from another. The terms the set fixes. Five limitations that follow
  from defining no exchange, each stating what the contract provides in
  its place, what an implementer does in the meantime, and what would
  remove it.
- Framing. Little-endian byte order. A 20-byte header on every frame
  of every kind, carrying `version`, `kind`, `flags`, `code`,
  `metadata length`, `payload length` and `request id`, so that a
  receiver holding 20 bytes knows the frame's kind, its total length,
  and the request it belongs to. Three frame kinds: REQUEST from the
  client, RESPONSE and ERROR from the server. Sixteen reserved flag bits
  a sender writes as zero. A total length of 20 plus the two declared
  lengths, computed in a width of at least 64 bits, and checked against
  the maximum frame size before a receiver reserves memory proportional
  to any declared length. The frame admission order: fifteen steps a
  receiver runs on every frame, stopping at the first that decides, of
  which thirteen name one error class and one failure scope and two
  state that the receiver does not hold a frame yet.
- Metadata. A region following the header and occupying exactly
  `metadata length` bytes, carried on every frame kind in either
  direction. Entries of `identifier`, `value length` and value, filling
  the region exactly, in strictly ascending order of identifier, so that
  a set of entries has one encoding and a duplicate identifier is
  refused by the same rule. Bit 15 of the identifier divides the whole
  domain: an unassigned optional identifier is skipped and produces no
  error, and an unassigned required identifier fails one request and
  leaves the connection serving. This revision assigns no identifier in
  either range.
- Correlation. The `request id` as the name of a request: allocated
  by the initiator, unique among the requests in flight on one
  connection and no further, and allocated again only after the request
  it named is retired. Request id 0 is reserved and states that the
  frame carrying it belongs to no request. Which frames name a request,
  when a request is in flight at a peer, which frame retires it, and
  that a receiver correlates a frame by its request id alone and never
  by the order frames arrive in.
- Operations. The opcode space, read from `code` on a REQUEST frame.
  Protocol version 0 assigns no opcode at this revision and every value
  of the field is reserved. A REQUEST frame whose opcode this revision
  does not assign is answered with the unsupported operation class for
  that request, and the connection keeps serving.
- Results. The result code space, read from `code` on a RESPONSE
  frame. Three codes are assigned: `0x0000` success, whose payload is
  the answer the operation defines; `0x0001` absent, whose payload is
  zero bytes; and `0x0004` value held outside memory, whose payload is
  eight bytes. A result code this revision does not assign is a protocol
  violation and closes the connection.
- Failures. One principle deciding the scope of every failure: a
  failure that leaves the stream position trustworthy fails one request,
  and a failure that does not leaves the connection unusable. Nine error
  classes, each carrying exactly one scope in every condition: malformed
  request, unsupported protocol version, resource limit and protocol
  violation close the connection; unsupported operation, invalid
  argument, overloaded, internal error and wrong type fail one request
  and leave the connection serving. The ERROR frame payload, as a detail
  length, the detail bytes the error class defines, and human readable
  text that no requirement of this contract constrains and no receiver
  parses. How each peer reports a failure of each scope, given that
  ERROR travels from the server to the client.
- Limits. Two bounds fixed by protocol version 0: a maximum frame
  size of 65536 bytes and a maximum metadata size of 4096 bytes. Neither
  is carried on the wire, neither is negotiated, and each binds both
  peers on every frame of every connection from that connection's first
  byte, so a receiver bounds its first allocation with no prior
  exchange. Where each bound is enforced, and the bounds the field
  widths enforce on their own.
- Extension and Version Policy. The property that decides the
  treatment of every change: whether a peer built on the previous
  revision can meet the change and still behave as its own revision
  requires. A table over thirteen kinds of change, five of which take a
  new protocol version, five a capability, and three an assignment
  alone, with a change the table does not name taking a new protocol
  version until an analysis places it in a row. The rule for an
  unassigned value in every space this revision defines, and what each
  space that grows by assignment alone costs the revision that grows it.
- Behavior for Unknown Input. One table collecting the outcome of
  every input this revision permits a peer to receive, ordered as a
  receiver meets them, each row naming the input, the error class, the
  failure scope, the section that binds that outcome, and the fixtures
  that prove it. The document introduces no requirement; where a row and
  the section it names differ, the section is the contract.

The normative fixture corpus for revision v0.1.0:

- 63 fixtures under `fixtures/`, one per file, each naming the clause it
  exercises, the bytes or the fields it offers, and the exact outcome the
  contract requires. A failure fixture states the exact error class and
  the exact failure scope, because two implementations can refuse the
  same bytes for different reasons and one of them is wrong.
- Every fixture is derived from a clause of the specification. None is
  captured from the output of an implementation. A fixture carries the
  same authority as the prose it exercises.
- `fixtures/README.md` states the form of the corpus and is not
  normative.

Published with the revision:

- The dual Apache-2.0 or MIT licensing offer, carried by the documents
  and the fixture corpus alike.
- A README stating what the repository is, that a conformance claim
  names a revision rather than a protocol version, and that the protocol
  version, the revision, and the server version are independent axes.
- Contribution, security, and conduct policy, including the requirement
  levels a clause uses, the change classification a pull request
  carries, and the rule that a fixture is derived from the contract
  rather than captured from an implementation.
- The Developer Certificate of Origin 1.1, and a check that refuses a
  pull request whose commits carry no matching sign-off.
- An ignore set covering the local working corpus.

### Not defined by this revision

- The handshake, or any other exchange that opens a connection.
- Negotiation, and every negotiated value. Both bounds this revision
  states are constants of protocol version 0.
- The capability registry. No capability identifier is assigned.
- Every operation. No opcode is assigned, so no work a server performs
  and no payload a request carries is named.
- Conformance layers, role obligations, and any required minimum. An
  implementation states which fixtures of this revision's corpus it
  passes rather than claiming conformance to the revision.
