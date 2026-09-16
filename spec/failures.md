---
title: Failures
description: The error classes protocol version 0 assigns, the scope and completion certainty of each, the ERROR frame payload, and the rule for an unassigned class.
protocol_version: 0
revision: v0.2.0
status: draft
order: 7
---

# Failures

## Scope

This document defines the error classes protocol version 0 assigns, the
scope each one carries, what a caller may conclude about a mutation from
each one, the payload of an ERROR frame, and what a receiver does with an
error class this revision does not assign.

It does not define which condition produces a class. Every requirement of
this document set states the class its violation produces, in the section
that binds the requirement.

A failure states that a request could not be served. An answer that
reports work a responder performed is not a failure, and this document
does not define one.

## Failure Scope

One principle decides the scope of every failure:

```text
A failure that leaves the stream position trustworthy fails one request.
A failure that does not leaves the connection unusable.
```

A receiver that can still find where the current frame ends knows what the
next frame is, and still owes an answer to the request it holds. It
answers, and the connection keeps serving. A receiver that cannot find the
frame boundary, or cannot trust which request a frame belongs to, has
nothing it can say about anything that follows.

Every error class this revision assigns carries exactly one scope:

| Scope | Effect on the request | Effect on the connection |
|---|---|---|
| request-scoped | the request fails, and no further answer to it follows | it keeps serving |
| connection-fatal | this frame retires no request | the sender closes it after sending this frame |

The scope of a failure follows from its error class. No other field of a
frame states it, and a receiver reads it from the class alone.

A class carries one scope in every condition. A later revision that needs
a condition scoped differently assigns a different class; it does not give
an assigned class a second scope.

### Reporting a request-scoped failure

ERROR travels from the server to the client, so the two peers answer a
request-scoped failure differently, and every rule in this document set
that answers an error class for a request is read with this.

A server that detects one answers with an ERROR frame carrying the class
and keeps the connection open. The request id an ERROR frame carries
states which request that frame names.

A client that detects one fails the request it holds, sends no frame, and
keeps the connection open, for the reason Reporting a connection-fatal
failure states. Its answer is local: the request retires as a failure of
the class the client read, and no further answer to it follows.

### The request id an ERROR frame carries

An ERROR frame carrying a request-scoped error class names the request it
fails, and it always names one. A request-scoped class states that one
request failed and that the connection keeps serving, so a frame carrying
that class and naming no request states nothing: there is no request to
retire and nothing to report.

An ERROR frame carrying a connection-fatal class names the request the
failure belongs to when its sender can attribute the failure to one, and
names none when it cannot.

The Reserved Request Id states what a peer may not send when a frame names
a request, and what a receiver does when it meets one.

## The ERROR Frame Payload

```text
offset  size  field          type
     0     2  detail length  u16
     2     n  detail bytes, the structure defined by the error class
   n+2    ..  text, UTF-8, human readable, may be empty
```

The text occupies the rest of the payload:

```text
text length = payload length - 2 - detail length
```

A receiver MUST check that `2 + detail length` does not exceed the
payload length before it takes either slice. A receiver that meets an
ERROR frame whose payload is shorter than two bytes, or whose
`detail length` exceeds the bytes that follow it, MUST treat the frame as
a malformed request and close the connection.

This check runs on an admitted frame. Checks that follow the order states
where it sits relative to the fifteen steps.

The text is not contractual. A receiver MUST NOT parse the text. A
receiver MUST NOT depend on its content. No requirement of this contract
constrains the text, so a peer that branches on it is not interoperable
with a conforming peer that changes a message or sends none.

## The Error Class Registry

An error class is a `u16`. The whole domain is covered here:

| Value | Class | Scope |
|---|---|---|
| `0x0000` | reserved | none |
| `0x0001` | malformed request | connection-fatal |
| `0x0002` | unsupported protocol version | connection-fatal |
| `0x0003` | unsupported operation | request-scoped |
| `0x0004` | invalid argument | request-scoped |
| `0x0005` | resource limit | connection-fatal |
| `0x0006..0x0007` | reserved | none |
| `0x0008` | overloaded | request-scoped |
| `0x0009..0x000A` | reserved | none |
| `0x000B` | internal error | request-scoped |
| `0x000C` | protocol violation | connection-fatal |
| `0x000D` | reserved | none |
| `0x000E` | wrong type | request-scoped |
| `0x000F..0xFFFF` | reserved | none |

A reserved value is one this revision does not assign. A reserved value
that lies between two assigned values is reserved on the same terms as one
above them: nothing in a frame distinguishes the two, and one rule covers
both.

A peer MUST NOT send an error class this revision does not assign. A
receiver that meets one MUST treat the frame as a protocol violation and
close the connection.

A receiver cannot act on a class it cannot name. It has no outcome to
report, cannot retire the request truthfully, and cannot state whether
anything was written, which every assigned class states.

Each class below states its scope. Failure Scope states what each scope
means for the request and for the connection.

## The Assigned Classes

### Malformed request

```text
value   0x0001
scope   connection-fatal
```

The receiver could not parse the frame as this contract defines it.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Unsupported protocol version

```text
value   0x0002
scope   connection-fatal
```

The frame names a protocol version the receiver does not implement.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Unsupported operation

```text
value   0x0003
scope   request-scoped
```

The receiver does not serve the operation the request names.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Invalid argument

```text
value   0x0004
scope   request-scoped
```

The receiver parsed the request and cannot accept a value it carries.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Resource limit

```text
value   0x0005
scope   connection-fatal
```

A frame's total length exceeds the maximum frame size in force on the
connection.

This is the whole meaning of the class.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Overloaded

```text
value   0x0008
scope   request-scoped
```

The receiver could not admit the resources the request needs.

This revision defines no surface that produces this class.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Internal error

```text
value   0x000B
scope   request-scoped
```

The receiver met a condition it did not anticipate.

A mutation: it may or may not have been applied. This is the only class
this revision assigns that does not state that nothing was written, and a
caller that must know re-reads what the request would have changed. The
class exists for conditions a responder did not anticipate, and a promise
about what such a condition did would be one the responder cannot keep.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Protocol violation

```text
value   0x000C
scope   connection-fatal
```

The frame breaks a requirement of this contract in a way that leaves the
receiver unable to trust what follows it.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.

### Wrong type

```text
value   0x000E
scope   request-scoped
```

The request names a key that is not held in a representation the operation
acts on.

This revision defines no surface that produces this class.

A mutation: nothing was written.

The detail region: this revision defines none for this class, and the
`detail length` of an ERROR frame carrying it is zero.
