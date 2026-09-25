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

## [0.5.1]

Classification: clarification. The revision states a requirement that was
already derivable and changes no requirement revision v0.5.0 publishes. A
peer built against v0.5.0 conforms to v0.5.0 and to this revision
unchanged.

This revision states, in the responder's own terms, that a responder is not
required to serve the requests in flight at it in the order it received
them, and that the frames it sends for them may interleave. The initiator
obligation to accept the terminal frames of its requests in any order was
already published; this revision extends it to any interleaving.

### Added

- The multiplexing freedom in Correlation: a responder is not required to
  serve the requests in flight at it in the order it received them, and it
  may interleave the frames it sends for them. An initiator MUST accept the
  frames of several requests in flight at it in any interleaving.
- One fixture, `correlation/interleaved-terminal-frames`, with three
  requests in flight at a client and a terminal frame for the middle one,
  so a receiver that matched by arrival order retires the wrong request.

### Changed

- Every document and every fixture states revision v0.5.1. The corpus of
  revision v0.5.0 is preserved at the tag that published it.
- The corpus is 108 fixtures.

## [0.5.0]

Classification: addition. The revision adds the opcode registry and the
five ungated operations and changes no requirement revision v0.4.0
publishes. A peer built against v0.4.0 conforms to v0.4.0 unchanged. That
revision assigns no operation, so such a peer does not run a request until
it implements the operations this revision adds; against a v0.5.0 request
it answers the unsupported operation class for that request and keeps the
connection open.

This revision defines the five operations a client reaches without
negotiating a capability: the liveness operation `PING`, the reads `GET`
and `EXISTS`, the write `SET`, and the delete `DEL`. It assigns their
opcodes, defines each request and response payload, states the argument
encoding they share, states the result codes each reaches, and states the
class and scope of every malformed payload.

The opcode space is the protocol's cheapest extension point. It assigned
`0x0001` at v0.3.0 and reserved the rest. This revision assigns the next
five values in that reserved range, so a peer built on an earlier revision
meets each new opcode by the rule it already carries and stays able to
serve.

### Added

- The opcodes `PING` `0x0002`, `GET` `0x0003`, `SET` `0x0004`, `DEL`
  `0x0005` and `EXISTS` `0x0006`, and the request and response payload of
  each. The whole opcode domain is covered, and the range `0x0007..0xFFFF`
  stays reserved.
- The shared argument encoding. A key is an opaque byte string and a value
  is an opaque byte string. `GET`, `DEL` and `EXISTS` carry the key alone,
  its length the payload length. `SET` carries a `u32` key length, then the
  key, then the value, whose length is derived; a `SET` whose `4 + key
  length` exceeds the payload length is a malformed request. An empty key
  and an empty value are each a legal value.
- The result codes the operations reach. `GET` answers success, absent and
  the value-held-outside-memory code; a success with a zero-byte payload is
  a present empty value and the absent code is a missing key. `EXISTS` and
  `DEL` answer success and absent. `SET` answers success.
- The class and scope of each malformed payload: a `SET` payload shorter
  than four bytes, a `SET` key length that overruns the payload, and a
  `PING` payload that is not empty are malformed requests, connection-fatal.

### Fixed

- Three fixtures carried opcode `0x0002` as an unassigned value. This
  revision assigns `0x0002` to `PING`, so each now carries `0x0007`, still
  unassigned above the assigned range. Each keeps its identifier, its
  clause, its expected class and its scope.

### Changed

- Scope and Status states that this revision defines the five ungated
  operations and removes the limitation that no operation exists.
- Failures states that `overloaded` and `wrong type` now have producers:
  a write that cannot be admitted answers `overloaded`, and a byte write
  against a key held in another representation answers `wrong type`. `DEL`
  never answers `wrong type`, and `EXISTS` answers presence whatever
  representation a key holds.
- Results states that the success payload of each operation is defined by
  this revision, and that `GET` is the operation that carries a value the
  value-held-outside-memory code states the length of.
- Extension and Version Policy, Correlation and Behavior for Unknown Input
  state the assignments and the rows this revision adds.
- Every document and every fixture states revision v0.5.0. The corpus of
  revision v0.4.0 is preserved at the tag that published it.

### Added fixtures

- 18 fixtures under `operations/`: a request and a response for each of the
  five operations, the two structural failures a `SET` payload can produce,
  a `PING` payload that is not empty, an empty key, and a key that carries a
  NUL byte.
- The corpus is 107 fixtures.

## [0.4.0]

Classification: defect correction. No clause of protocol version 0
changes, no wire value a conforming peer emits or accepts changes, and a
peer built against revision v0.3.0 conforms to this revision unchanged and
has nothing to do. An implementation that loads the fixture corpus updates
the revision it expects and picks up the three corrected fixtures.

This revision corrects three fixtures that did not match the clause they
name. Revision v0.3.0 assigns opcode `0x0001` to the handshake. Three
fixtures carried `0x0001` as if it were unassigned and expected the
unsupported operation class, while `lifecycle/repeated-handshake` carries
the same kind and opcode in the same state and expects a protocol
violation. The corpus contradicted the opcode registry and contradicted
itself: no receiver could produce both outcomes for the same kind and
opcode in the same connection state.

### Fixed

- `operations/unassigned-opcode-refused` now carries opcode `0x0002`, a
  value the opcode registry leaves unassigned. It keeps its identifier,
  its clause, its expected unsupported operation class and its
  request-scoped scope, so it still exercises the receiver rule for an
  opcode this revision does not assign.
- `operations/unassigned-opcode-with-a-payload-refused` carries opcode
  `0x0002` and keeps its four-byte payload, its expected class and its
  scope, so it still pins that the opcode is refused whatever the payload
  carries.
- `metadata/admission-order-code-before-required-identifier` carries
  opcode `0x0002` and keeps its expected class and scope, so it still
  pins step 13 before step 15.

### Changed

- Every normative document and every fixture states revision v0.4.0. The
  corpus of revision v0.3.0 is preserved at the tag that published it.

### Unchanged

Every clause of protocol version 0 that revision v0.3.0 published. The
frame header, the frame kinds, the admission order, the metadata region,
correlation, the opcode and capability registries, the result codes, the
error classes, the bounds and the extension policy are byte for byte the
contract v0.3.0 published. The requirement levels are unchanged and every
document still carries the status `draft`.

## [0.3.0]

Classification: addition. The revision adds the connection surface and
changes no requirement revision v0.2.0 publishes. A peer built against
v0.2.0 conforms to v0.2.0 unchanged. That revision defines no exchange, so
such a peer does not interoperate with a v0.3.0 peer until it implements
the handshake and version negotiation this revision adds.

This revision defines how a connection becomes usable: the connection
states, the handshake exchange in both directions, version negotiation, the
bounds the handshake derives, and the capability mechanism. It assigns the
handshake opcode and no capability identifier, so a connection it produces
is usable and runs no operation.

This revision answers a mismatch between the fixed maximum frame size of
v0.2.0 and the value a deployed server negotiates. v0.2.0 fixes 65536 and
defines no negotiation; a v0.3.0 responder negotiates a maximum frame size
of `min(max(proposed, 65536), ceiling)` with a ceiling of at least 65536.
The fixed value becomes the floor of the negotiated value rather than a
protocol-wide cap, so a value above 65536 is legal after the handshake.

### Added

- Handshake. The three connection states a connection occupies, the frames
  legal in each, and the failure a frame outside them produces. The first
  frame of a connection MUST be a REQUEST frame carrying the handshake
  opcode; any other frame before the handshake completes, and a second
  handshake after it, is a protocol violation that closes the connection.
  The handshake request and response payloads, every field with its offset,
  size and type, the capability entry layout, the checks over each payload,
  version selection as the highest version a responder supports inside the
  range the offerer proposed, and the outcome when no version is mutually
  supported.
- Capabilities. The capability identifier domain over its whole `u16`
  width, the offer and acceptance rules, the receiver rule for an
  unassigned identifier and for a value of the wrong length, the rule that
  a responder never accepts a capability the offerer did not offer, and the
  dependency rule. This revision assigns no capability identifier, so the
  accepted set is empty and nothing is gated.
- The negotiated maximum frame size and maximum metadata size, with their
  formulas, floors and ceilings, and which bound is in force before and
  after the handshake.
- The handshake opcode `0x0001`. The rest of the opcode space stays
  reserved, and a REQUEST frame carrying any other opcode is answered with
  the unsupported operation class for that request.

### Changed

- The maximum frame size of 65536 bytes and the maximum metadata size of
  4096 bytes are now the pre-negotiation constants. After the handshake the
  negotiated values are in force. A frame that exceeds the bound in force
  produces the same class it produced at v0.2.0.
- Scope and Status now states that this revision defines a usable
  connection and no operation beyond the handshake.
- Extension and Version Policy states that the capability identifier space
  grows by an assignment.
- Correlation states the request states and the terminal frame of the
  handshake.
- Behavior for Unknown Input gains the rows the connection state and the
  handshake create.
- Every document and every fixture states revision v0.3.0. The corpus of
  revision v0.2.0 is preserved at the tag that published it.

### Added fixtures

- 18 fixtures: 14 under `handshake/`, 2 under `lifecycle/`, and 2 under
  `limits/`. Each states the exact class and scope of a failure it
  exercises, or the exact fields of a successful frame.
- The corpus is 89 fixtures.

### Unchanged

Every clause of protocol version 0 that v0.2.0 published and this revision
does not touch: the frame header layout, the frame kinds, the admission
order, the metadata region rules, request correlation, the result codes and
the error classes are the contract v0.2.0 published. The requirement levels
are unchanged. Every document still carries the status `draft`.

## [0.2.0]

Classification: defect correction. No clause of protocol version 0
changes, no wire value changes, and no fixture changes what it asserts
about the wire. A peer built against revision v0.1.1 conforms to this
revision unchanged and has nothing to do. An implementation that loads
the fixture corpus updates its loader for one rule.

This revision corrects the form of the corpus and publishes the header
encode coverage the contract has listed since revision v0.1.0.

The corpus carried every field value as a JSON number. A JSON number is
interoperable only across the integers a reader represents exactly, and a
reader that uses an IEEE 754 binary64 double agrees on -(2^53)+1 to
(2^53)-1 and on no others. The maximum request id, 18446744073709551615,
is far above that range: such a reader loaded it as a value the field
cannot carry, and reported no error. The form could not state a legal
value of a field this contract defines, so no fixture could reach the
maximum of `request id`.

The corpus also published two fixtures whose direction was `both`, and
each offered an encoder a frame the contract forbids a peer to send.
Protocol version 0 assigns no metadata identifier, and a peer MUST NOT
send an entry whose identifier this revision does not assign.

### Changed

- A field whose wire type is `u64` is carried as a JSON string of decimal
  digits, with no sign, no separators, and no leading zero except for
  zero itself. The form follows the type of the field and never the
  magnitude of the value, so `"1"` and `"18446744073709551615"` are both
  strings and a loader maps the field to its 64-bit integer type once,
  from the field's name. Protocol version 0 types two fields that way:
  the `request id` of the frame header, and the `logical length` of the
  value held outside memory result code. The two corpus members whose
  values are request ids, `in_flight` and `retires`, take the same form.
  Every other integer stays a number, and none of them can exceed the
  maximum frame size. Twenty fixtures state a member in the new form, and
  each one states the value it stated before.
- The corpus form states that the input of a fixture whose direction is
  `encode` or `both` is a frame a peer may send at the revision the
  fixture represents, and that a frame the contract forbids a peer to
  send is stated by a decode fixture instead.
- `metadata/single-entry` and `metadata/two-entries-ascending` are decode
  fixtures. Each keeps its identifier, its clause, its byte string and
  every field it asserts, so the coverage of the entry layout and of
  entry order is unchanged. Until a revision assigns a metadata
  identifier, no fixture pins the bytes an encoder writes for an entry.
- `header/minimum-legal-frame` states both directions. It pins the bytes
  an encoder writes for the minimum legal value of every header field a
  frame of this revision carries.
- The corpus form states that an encoder handed a payload whose layout a
  section defines is given that payload's fields rather than its bytes.
- Every normative document and every fixture states revision v0.2.0. The
  corpus of revision v0.1.1 is preserved at the tag that published it.
- The corpus is 71 fixtures.

### Added

- `header/request-id-at-the-maximum`, a RESPONSE frame carrying
  18446744073709551615, the highest valid request id. It is the fixture
  the corpus form blocked, and it fails against an encoder or a decoder
  that carries the field in any width below 64 bits.
- `limits/the-widest-legal-frame`, a RESPONSE frame carrying the success
  result code and a payload of 65516 bytes, whose total length is the
  maximum frame size exactly. It fails against an encoder that caps its
  output below the bound the contract permits.
- `results/the-highest-assigned-result-code`, the value held outside
  memory code, whose eight-byte payload carries a logical length at the
  maximum of its width.
- `errors/the-lowest-assigned-error-class`, a malformed request frame.
  The class is connection-fatal and the frame names no request, so the
  fixture also pins the reserved request id on a frame that may carry it.
- `errors/the-highest-assigned-error-class`, a wrong type frame, which is
  request-scoped and names the request it fails.
- A statement in the corpus form of the three header field boundaries no
  frame of this revision reaches: `kind` at REQUEST, because no opcode is
  assigned; `code` at `0xFFFF`, unassigned in all three of the spaces the
  field draws from; and `metadata length` above zero, because no metadata
  identifier is assigned. Each one is forbidden by the contract rather
  than unstatable by the corpus, each is covered on the decode side, and
  each names the revision that removes it.

### Unchanged

Every clause of protocol version 0. The frame header, the frame kinds,
the admission order, the metadata region, correlation, the opcode space,
the result codes, the error classes, the bounds and the extension policy
are byte for byte the contract revision v0.1.1 published, and the
requirement levels are unchanged. Every document still carries the status
`draft`.

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
