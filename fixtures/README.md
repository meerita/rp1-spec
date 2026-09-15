---
title: Fixture Corpus (Non-Normative)
description: The form of the fixture corpus published with revision v0.1.0 of protocol version 0.
protocol_version: 0
revision: v0.1.0
normative: false
---

# Fixture Corpus (Non-Normative)

This file is not normative. It states the form of the corpus so that an
implementation in any language can load it.

The fixtures are normative. A fixture carries the same authority as the
prose it exercises. Where this file and a specification document disagree,
the document is the contract.

## What a fixture is

A fixture is the part of the contract a machine can check. The prose
states the rule, and the fixture proves that two implementations read it
the same way.

Every fixture in this corpus is derived from a clause of the
specification. None is captured from the output of any implementation. An
implementation is checked against a fixture; it never produces one.

## Files and identifiers

One fixture per file, encoded as JSON:

```text
fixtures/<identifier>.json
```

The path mirrors the identifier. The identifier names what the fixture
exercises, never the order it was written in, and it is stable: a renamed
fixture is a lost regression.

```text
Example: the identifier header/minimum-legal-frame is the file
fixtures/header/minimum-legal-frame.json
```

## Byte form

Every byte string in this corpus is a JSON string of lowercase
hexadecimal digits, with no separators and no prefix. Its length is even.
A byte string of length zero is the empty string.

```text
"0001000000000000"      eight bytes
""                      no bytes
```

## Fields

Every fixture is a JSON object carrying all eight fields:

| Field | Type | Meaning |
|---|---|---|
| `id` | string | the stable identifier, matching the file path |
| `revision` | string | the specification revision the fixture represents, `v0.1.0` in this corpus |
| `protocol_version` | number | the protocol version the fixture represents, `0` in this corpus |
| `clause` | string | the title of the section that binds the rule the fixture exercises |
| `direction` | string | `decode`, `encode`, or `both` |
| `provenance` | string | `derived-from-specification`, on every fixture |
| `input` | object | what the fixture offers |
| `expect` | object | the outcome the contract requires |

A `clause` value is the title of a section that binds a rule. Every such
title is unique across the document set, so a `clause` value names one
section. `Scope` is the one title the set repeats, and no Scope section
binds a rule, so no fixture carries it.

`provenance` carries one value in this corpus. A fixture whose provenance
cannot be stated is removed rather than kept.

## Input

| Direction | `input` carries |
|---|---|
| `decode` | `bytes`, the byte string offered to the decoder |
| `encode` | `fields`, an object naming each field the encoder is given |
| `both` | `fields`, with the same meaning as for `encode` |

`input` carries two further members:

| Member | Presence | Meaning |
|---|---|---|
| `role` | when the outcome depends on which peer reads the bytes | `client` or `server`, the peer the bytes are offered to |
| `in_flight` | when the outcome depends on which requests are in flight at the receiver | the request ids in flight at the receiver, as an array of numbers, in the order those requests entered flight |

A field name is the name the specification gives the field, with each
space written as an underscore, so `metadata length` is `metadata_length`.
A field's value is the value a decoder extracts, not the name the registry
gives it: a fixture states `"kind": 2`, and the registry is what says that
2 is RESPONSE.

Some outcomes depend on state a connection holds rather than on the bytes
of one frame. `in_flight` is how a fixture states that state. A fixture
that states no such state asserts its outcome for a receiver at which
every check reading connection state passes.

`in_flight` is ordered so that a fixture can fail against a receiver that
correlates a frame by the order requests were sent rather than by the
request id the frame carries. The order carries no other meaning, and an
empty array states that no request is in flight at the receiver.

## Entries of a region

A field whose value is a region of entries is carried as a JSON array, one
object per entry, in wire order. The metadata region is the field
`metadata`, and an absent region is the empty array.

| Member | Meaning |
|---|---|
| `identifier` | the entry's identifier, as a number |
| `value` | the entry's value bytes, as a byte string |

An entry's `value length` is not carried. It is the length of `value`.

An encoder is given what it cannot derive. On `encode` and `both`,
`input.fields` carries `metadata` and `payload` and carries neither
`metadata_length` nor `payload_length`, because an encoder computes both
from what it was given. On `decode`, and in `expect.fields` everywhere,
the two length fields are present, because a decoder reads them. A fixture
whose direction is `both` therefore names a different field set on each
side: what an encoder is handed, and what a decoder extracts.

## Payload fields

`fields` carries the header fields of a frame, and the fields of its
payload when a section of the specification defines a layout for that
payload. A payload the contract leaves opaque contributes no member, and
`payload_length` is then the whole of what the fixture states about it.

A field of a payload is named by the same rule as a header field, and a
byte region of a payload is written in the byte form above.

## Expected outcome

`expect.outcome` carries one of three values. Each one fixes which other
members of `expect` are present.

### `success`

| Member | Presence | Meaning |
|---|---|---|
| `fields` | always | every field value a conforming decoder extracts |
| `bytes_consumed` | on `decode` and `both` | the number of input bytes the frame occupies |
| `bytes` | on `encode` and `both` | the exact byte string a conforming encoder produces |
| `retires` | when `input` states `in_flight` | the request id the frame retires at the receiver, or `0` when it retires none |

A fixture whose direction is `both` asserts both halves: encoding
`input.fields` produces `expect.bytes`, and decoding `expect.bytes`
produces `expect.fields` and consumes every byte of it.

### `failure`

| Member | Presence | Meaning |
|---|---|---|
| `class` | always | the error class, named exactly as the failure registry states it |
| `scope` | always | `request-scoped` or `connection-fatal` |
| `bytes_consumed` | when `scope` is `request-scoped` | the number of input bytes the refused frame occupies |

A failure fixture states the exact class and the exact scope. "Rejects" is
not an outcome: two implementations can reject the same bytes for
different reasons, and one of them is wrong.

`bytes_consumed` is present when the scope is `request-scoped`, because
the connection keeps serving and the next frame begins at that offset. It
is absent when the scope is `connection-fatal`, because no next frame
follows.

### `incomplete`

| Member | Presence | Meaning |
|---|---|---|
| `bytes_required` | always | the total number of bytes the receiver requires before it holds a frame |

An incomplete frame is a decoder state and not a failure, so an
`incomplete` fixture carries no class and no scope. The receiver reads
more bytes.

## Shape

```text
Example, illustrative. A value in angle brackets stands for content a
real fixture carries.

{
  "id": "header/minimum-legal-frame",
  "revision": "v0.1.0",
  "protocol_version": 0,
  "clause": "Frame Header",
  "direction": "decode",
  "provenance": "derived-from-specification",
  "input": {
    "bytes": "<the frame, as hexadecimal digits>"
  },
  "expect": {
    "outcome": "success",
    "fields": { "<field name>": "<field value>" },
    "bytes_consumed": 20
  }
}
```

## Revision binding

Fixtures publish with the revision they represent and are not mixed across
revisions. A published fixture is immutable within its revision: it is the
evidence of what that revision required.

An implementation that disagrees with a fixture is in one of two states.
Either the implementation is wrong, or the fixture does not match the
prose it claims to exercise. Both are settled by reading the clause.
Neither is settled by regenerating output. A fixture found not to match
its clause is corrected as a defect, and the changelog records what it
asserted and why that was wrong.
