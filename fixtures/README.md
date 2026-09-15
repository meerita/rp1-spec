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

Section titles are unique across the document set, so a `clause` value
names one section.

`provenance` carries one value in this corpus. A fixture whose provenance
cannot be stated is removed rather than kept.

## Input

| Direction | `input` carries |
|---|---|
| `decode` | `bytes`, the byte string offered to the decoder |
| `encode` | `fields`, an object naming each field the encoder is given, under the field names of the section named in `clause` |
| `both` | `fields`, with the same meaning as for `encode` |

## Expected outcome

`expect.outcome` carries one of three values. Each one fixes which other
members of `expect` are present.

### `success`

| Member | Presence | Meaning |
|---|---|---|
| `fields` | always | every field value a conforming decoder extracts |
| `bytes_consumed` | on `decode` and `both` | the number of input bytes the frame occupies |
| `bytes` | on `encode` and `both` | the exact byte string a conforming encoder produces |

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
