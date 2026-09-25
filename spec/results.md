---
title: Results
description: Where a result code appears in a frame, the whole domain of the result code space, the result codes protocol version 0 assigns and the payload of each, what a receiver does with a result code it does not assign, and what a later revision does when it assigns one.
protocol_version: 0
revision: v0.5.0
status: draft
order: 8
---

# Results

## Scope

This document defines the result code space: where a result code appears
in a frame, the whole domain of the field that carries it, the result
codes protocol version 0 assigns, the payload each one carries, what a
receiver does with a result code this revision does not assign, and what a
later revision does when it assigns one.

A result code states an outcome. It states that an operation ran and what
it produced, including the outcomes where it produced nothing and where it
declined to act. A failure states that a request could not be served, and
The Error Class Registry states that taxonomy separately. Both are read
from the same field, so a caller tells the two apart by the kind of the
frame that carried it.

It defines no operation payload of its own. Handshake defines the payload
the success result code carries for the handshake response, and Operations
defines it for each of the five ungated operations. For any other operation
the success payload is defined by the revision that assigns the operation.

## Result Codes

A result code appears in the `code` field of a RESPONSE frame. Frame
Header states that `code` is a `u16` and that its meaning follows the
frame's kind. No other frame kind this revision assigns carries a result
code.

The whole domain is covered here:

| Value | Result | Payload |
|---|---|---|
| `0x0000` | success | the answer the operation defines |
| `0x0001` | absent | zero bytes |
| `0x0002..0x0003` | reserved | none |
| `0x0004` | value held outside memory | eight bytes |
| `0x0005..0xFFFF` | reserved | none |

A reserved value is one this revision does not assign. A reserved value
that lies between two assigned values is reserved on the same terms as one
above them: nothing in a frame distinguishes the two, and one rule covers
both.

A peer MUST NOT send a result code this revision does not assign. A
receiver that meets one MUST treat the frame as a protocol violation and
close the connection.

Frame Admission Order places this check at step 13.

A result code is the whole of what a RESPONSE frame means, and a RESPONSE
frame retires the request it names. A receiver that cannot name the code
has no outcome to report and cannot retire the request truthfully, which
is the reason The Error Class Registry gives for the same rule over its
own space.

## The Assigned Result Codes

Each code below states its payload exactly. Where a code fixes the length
of its payload, a receiver checks `payload length` against that code
before it takes any slice of the payload.

These checks run on an admitted frame. Checks that follow the order
states where they sit relative to the fifteen steps.

### Success

```text
value     0x0000
payload   the answer the operation defines
```

The operation ran and produced its answer.

The payload: the operation the request named defines it in full, including
whether it may be zero bytes. Handshake defines the payload for the
handshake response, and Operations defines it for each of the five ungated
operations. For any other request the revision that assigns the operation
defines the success payload, and this document states no length a receiver
checks for it.
Within the bound Negotiated maximum frame size states, every value of `payload length`
is a legal encoding of a RESPONSE frame carrying this code at this
revision.

A payload of zero bytes under this code is an answer the operation
produced, and it is not an absent one. Absent is a different result code,
so the two are different frames and a caller never derives one from the
other.

### Absent

```text
value     0x0001
payload   zero bytes
```

The key the request named does not exist.

The payload: zero bytes, always. A peer MUST NOT send a RESPONSE frame
carrying this code whose `payload length` is not zero. A receiver that
meets one MUST treat the frame as a malformed request and close the
connection.

This is an outcome and not a failure. The operation ran, the request
retires with an answer, and the connection keeps serving. A caller
separates it from success by the result code alone and parses nothing to
do it.

### Value held outside memory

```text
value     0x0004
payload   eight bytes
```

The key the request named holds a value the responder keeps outside
memory, and this answer carries none of that value.

The payload:

```text
offset  size  field           type
     0     8  logical length  u64
                              total = 8 bytes
```

`logical length` states the length in bytes of the value the key holds.
Every value of the width is a valid encoding of the field.

A peer MUST NOT send a RESPONSE frame carrying this code whose
`payload length` is not eight. A receiver that meets one MUST treat the
frame as a malformed request and close the connection.

This is an outcome and not a failure. The key exists, the responder states
how large its value is, and it carried none of it. A caller that needs the
value reads it through `GET`, which Operations assigns; a caller that needs
only the size has it here.

## Assigning a Result Code Later

A later revision assigns a result code only under a capability. It never
adds one as a new ungated answer to an operation a previous revision left
ungated.

A peer built on this revision meets a result code it cannot name by the
rule Result Codes states: it closes the connection. A later revision that
assigned an ungated code would break every peer built on this one, on the
first request that produced it.

A revision that replaces an answer a peer without the new capability is
still owed states the substitute that peer receives in its place.

The two spaces a request and its answer draw `code` from are extended
differently, and the difference follows from what each frame's code means.
A responder that cannot name an opcode still holds a well-formed request
and still owes an answer it is able to produce, so Opcodes refuses one
request and keeps the connection. A receiver that cannot name a result
code holds the answer itself, and has nothing left to say about the
request or about the frame after it.
