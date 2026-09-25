---
title: Extension and Version Policy
description: The property that decides whether a change invalidates a peer built on an earlier revision, the treatment every kind of change carries, the growth treatment of every value space protocol version 0 defines at revision v0.5.0, and the fixture that proves each receiver rule those treatments follow from.
protocol_version: 0
revision: v0.5.0
status: draft
order: 11
---

# Extension and Version Policy

## Scope

This document defines the extension and version policy of protocol
version 0: the property that decides whether a change invalidates a peer
built on an earlier revision, the treatment every kind of change carries,
the growth treatment of every value space this revision defines, and the
fixture that proves the receiver rule each treatment follows from.

It introduces no requirement and carries no requirement-level keyword.
Every rule it names is bound by the section it names beside that rule, and
that section states the rule in full. A reader implements those sections.
This document states what a later revision may do to the spaces they
define.

It introduces no requirement about a capability. Capabilities owns the
capability identifier domain and the offer and acceptance rules, and The
Capability Identifier Space states how that space grows.

Version Axes states that the protocol version, the server version and the
SDK version are independent, and that none of the three is derivable from
another. This document is about the first of them alone.

## The Property That Decides

One question decides the treatment of every change:

```text
Can a peer built on the previous revision meet this change and still
behave as its own revision requires?
```

A peer that can is a peer the change does not invalidate. A peer that
cannot is a peer the change breaks, and a new protocol version is how the
contract announces that.

Every value space this revision defines carries a receiver rule for a
value the revision does not assign, stated in the section that defines the
space. That rule is what answers the question for the space, and one
property separates the answers:

```text
a receiver that still owes an answer can produce one
a receiver that is owed the answer cannot
```

A receiver that meets an unassigned opcode holds a request it owes an
answer to, and Opcodes has it refuse that one request while the connection
keeps serving. A receiver that meets an unassigned result code holds the
answer itself, and Result Codes has it close the connection, because it
has no outcome to report and nothing to say about the frame that follows.

Two treatments follow from the one property:

```text
a space whose rule for an unassigned value leaves the connection serving
  grows with an assignment alone
a space whose rule for an unassigned value closes the connection grows
  only behind a capability
```

## The Version Bump Policy

| The change | What it needs |
|---|---|
| the frame header's layout, or the width of one of its fields | a new protocol version |
| the meaning of a value an earlier revision assigned | a new protocol version |
| the semantics of an operation an earlier revision assigned | a new protocol version |
| a frame kind a peer built on an earlier revision must be able to receive | a new protocol version |
| a flag bit a peer built on an earlier revision must be able to receive set | a new protocol version |
| a new frame kind a capability gates | a capability |
| a new flag bit a capability gates | a capability |
| a new result code | a capability |
| a new error class | a capability |
| any other new optional feature | a capability |
| a new operation | an assignment alone |
| a new optional metadata entry | an assignment alone |
| a new required metadata entry | an assignment alone |

A change this table does not name takes a new protocol version until an
analysis places it in a row. The asymmetry is deliberate: a change wrongly
treated as a new protocol version costs a revision, and a change wrongly
treated as an assignment breaks a deployed peer on the first frame that
carries it.

The first five rows are the changes a peer built on an earlier revision
cannot meet. It decodes every frame against the layout its own revision
fixes, it reads every assigned value with the meaning its own revision
states, and it closes the connection on a frame kind and on a flag bit its
own revision does not assign. Nothing a later revision writes changes any
of that, so the later revision changes the protocol version instead.

The five rows that name a capability are the changes a peer meets only
when it has agreed to them. A capability is offered, accepted, and only
then used, so a peer that did not negotiate one never receives what it
gates and never has to name a value its revision does not assign.

The three rows that need an assignment alone are the spaces whose rule for
an unassigned value leaves the connection serving. A later revision
assigns a value in one of them, a peer built on this revision meets it,
refuses or skips it by the rule its own revision states, and the
connection keeps serving. Growth Without a Capability states what each of
those three costs a later revision in place of one.

## The Version Field's Width

`version` is one byte, at offset 0 of the frame header. Frame Header
states its only valid value at this revision and the rule for every other
one, and states that the field travels on every frame so that a frame
decodes without connection state.

No later revision of protocol version 0 assigns a value of this field.
Every value but 0 names another protocol version, so assigning one is not
growth in a space: it is a new protocol version.

Widening the field, or moving it, changes the header layout and takes a
new protocol version by the first row of the table above. The field's
position is what makes that change survivable at all: a peer built on this
revision refuses a frame of another protocol version at step 2 of Frame
Admission Order, from byte 0 alone, before it reads a byte of any other
field. A protocol version that keeps a version at byte 0 and never writes
0 there is therefore refused correctly by a peer built on this revision,
whatever it did to the rest of the header. One that did not would be read
against this revision's layout, and the frame's declared length would be
taken from bytes that do not carry it.

## The Spaces This Revision Defines

Every space appears here, with the rule it carries and the treatment that
rule produces. The section named in each row states the rule in full,
including the peer it binds and the consequence of violating it.

| Space | The section that states its rule | The rule for a value this revision does not assign | How the space grows |
|---|---|---|---|
| `version` | Frame Header | unsupported protocol version, connection-fatal | a new protocol version |
| `kind` | Frame Kinds | protocol violation, connection-fatal | a capability |
| `flags` | Flags | protocol violation, connection-fatal | a capability |
| `code` on a REQUEST frame | Opcodes | unsupported operation, request-scoped | an assignment alone |
| `code` on a RESPONSE frame | Result Codes | protocol violation, connection-fatal | a capability |
| `code` on an ERROR frame | The Error Class Registry | protocol violation, connection-fatal | a capability |
| `identifier` in `0x0000..0x7FFF` | The Optional Range | skipped, and no failure | an assignment alone |
| `identifier` in `0x8000..0xFFFF` | The Required Range | invalid argument, request-scoped | an assignment alone |
| `request id` 0 | The Reserved Request Id | protocol violation, connection-fatal, on a frame that names a request | with the frame kind that gives it a meaning |
| `capability id` | Capabilities | ignored, and no failure | an assignment alone |

The last row is the one space whose growth is not its own. Request id 0
states that a frame belongs to no request, and which frames may carry it
follows from the frame kind. A later revision that gives the value a
meaning does so by assigning a frame kind that carries it, and the frame
kind row decides what that assignment takes.

### Growth Without a Capability

Three spaces grow without a capability and without a new protocol version.
This revision assigns no value in the two metadata spaces and assigns only
the handshake and the five ungated operations in the opcode space, so every
other value of each space is available to a later revision.

The opcode space grows at the cost of one refused request. Assigning an
Opcode Later states it: a peer built on this revision answers an opcode it
does not assign with the unsupported operation class for that request and
keeps serving. A peer built on the later revision learns that its peer
does not serve the operation by sending the request and reading the
failure. That trial is the whole of the cost, and it is one request.

The metadata region's optional range grows at no cost. The Optional
Range states it: a receiver skips an entry whose identifier it does not
assign, produces no failure, resumes at the next entry, and serves the
frame.

The metadata region's required range grows at the cost of one refused
request per frame that carries the new entry. The Required Range states
it: a receiver answers the invalid argument class for that request and
keeps serving, so the sender is told the receiver could not serve the
frame rather than having the entry ignored.

### Growth Behind a Capability

Four spaces carry a rule that closes the connection, so a later revision
assigns a value in one of them only behind a capability, and a peer that
did not negotiate that capability never receives the value.

Three of them carry assignments this revision made. Result Codes assigns
three result codes, The Error Class Registry assigns nine error classes,
and Frame Kinds assigns three frame kinds. Each of the three reserves the
rest of its space. Assigning a Result Code Later states the treatment for
the result code space, and states what a later revision does when the new
value replaces an answer a peer without the capability is still owed.

The fourth is `flags`, and this revision assigns no bit of it, because it
defines no behavior a flag would carry. Flags states why the field holds
two bytes at a fixed offset regardless.

The asymmetry between the two groups follows from the property. A value
left out of a space that grows with an assignment alone costs a later
revision nothing but that assignment. A value left out of a space whose
rule closes the connection costs
a later revision a capability, and costs every peer built before that
capability the answer it would have received. This revision therefore
assigns where deferring is expensive and defers where assigning is cheap,
which is why it assigns the six ungated opcodes and no metadata
identifier, and assigns in both code spaces.

## The Fixtures That Prove the Rules

Each rule named above has a fixture in this revision's corpus. A fixture
carries the same authority as the prose it exercises.

| The rule | The fixture |
|---|---|
| a `version` this revision does not assign | `header/version-field-is-one` |
| a `kind` this revision does not assign | `header/unassigned-frame-kind` |
| a non-zero `flags` | `header/flags-single-reserved-bit-set` |
| an opcode this revision does not assign | `operations/unassigned-opcode-refused` |
| a result code this revision does not assign | `results/unassigned-result-code-refused` |
| an error class this revision does not assign | `errors/unassigned-error-class-refused` |
| an optional metadata identifier this revision does not assign | `metadata/unassigned-optional-identifier-skipped` |
| a required metadata identifier this revision does not assign | `metadata/unassigned-required-identifier-refused` |
| `request id` 0 on a frame that names a request | `correlation/request-frame-with-the-reserved-id` |

## The Capability Identifier Space

Capabilities defines the capability identifier domain over its whole
width, the entry layout, and the offer and acceptance rules. This revision
assigns no identifier in that domain, so the accepted set is empty on
every connection and nothing is gated.

The domain grows by an assignment alone: a receiver ignores an identifier
it does not assign and produces no failure, so a later revision assigns an
identifier together with the surface it gates without invalidating a peer
built on this revision. That is the mechanism the four spaces whose rule
closes the connection need. From the revision that assigns an identifier,
a new frame kind, flag, result code or error class grows behind it, and a
peer that did not negotiate the identifier never receives the value.

The three spaces that grow without a capability are open from this
revision onward: the opcode space and both ranges of the metadata
identifier space. A later revision adds an operation, or adds metadata to
a frame of any kind, with an assignment alone. It adds an outcome, a
failure, a frame kind or a flag only with a capability.
