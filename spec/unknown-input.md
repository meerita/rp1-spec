---
title: Behavior for Unknown Input
description: Every input protocol version 0 permits a peer to receive at revision v0.1.2, in the order a receiver meets them, with the error class and the failure scope each one produces, the section that binds that outcome, and the fixtures that prove it.
protocol_version: 0
revision: v0.1.2
status: draft
order: 10
---

# Behavior for Unknown Input

## Scope

This document collects the outcome of every input this revision permits a
peer to receive: every reserved value, every value a space leaves
unassigned, every region that does not decode, every frame a receiver
cannot correlate, and the two states in which a receiver does not hold a
frame yet. For each one it names the error class, the failure scope, the
section that binds them, and the fixtures that prove the outcome.

It introduces no requirement and carries no requirement-level keyword.
Every outcome below is bound by the section named beside it, and that
section states the rule in full, including the peer it binds and the
consequence of violating it. A reader implements those sections.
Where a row below and the section it names differ, the section is the
contract and the row is a defect.

It adds no input. A frame kind, a value space, or a region this revision
does not define has no row here, and Known Limitations states what this
revision leaves undefined.

## Reading a Row

The rows of Every Input the Order Decides appear in the order of Frame
Admission Order, so a reader meets them in the order a receiver runs
them. A receiver stops at the first step that decides, so a frame that
matches more than one row produces the outcome of the row it meets first.

Three rows carry `none` in the class column and the scope column, and
none of the three is a failure. At steps 1 and 7 the receiver does not
hold a frame yet and reads more bytes. An entry carrying an unassigned
optional metadata identifier is one the receiver skips. Every other row
names one class and one scope.

A row's scope states what the outcome costs. Reporting a connection-fatal
failure states how each peer reports a row whose scope is
connection-fatal, and Reporting a request-scoped failure states how each
peer answers a row whose scope is request-scoped.

A fixture carries the same authority as the prose it exercises. The
fixtures named in a row are the frames that decide whether an
implementation produces that row's outcome.

## Every Input the Order Decides

| Step | The input | The class | The scope | The section that binds it | The fixtures |
|---|---|---|---|---|---|
| 1 | fewer than 20 bytes held | none, the receiver requires 20 bytes | none | The two steps that are not failures | `framing/incomplete-header-empty-buffer`, `framing/incomplete-header-one-byte-short`, `correlation/partial-frame-retires-no-request` |
| 2 | `version` carries a value other than 0 | unsupported protocol version | connection-fatal | Frame Header | `header/version-field-is-one`, `header/version-field-at-maximum` |
| 3 | `kind` carries a value this revision does not assign | protocol violation | connection-fatal | Frame Kinds | `header/frame-kind-reserved-zero`, `header/unassigned-frame-kind` |
| 4 | the total length exceeds the maximum frame size | resource limit | connection-fatal | The maximum frame size | `limits/frame-one-byte-past-the-maximum`, `limits/metadata-length-exceeds-the-frame`, `framing/oversize-frame-refused-before-the-body-arrives` |
| 4 | the total length exceeds the maximum frame size and wraps a computation narrower than 64 bits | resource limit | connection-fatal | The maximum frame size | `framing/total-length-that-wraps-a-32-bit-computation` |
| 5 | `flags` carries a non-zero bit | protocol violation | connection-fatal | Flags | `header/flags-single-reserved-bit-set`, `header/flags-all-bits-set` |
| 6 | `metadata length` exceeds the maximum metadata size | malformed request | connection-fatal | The maximum metadata size | `limits/metadata-region-one-byte-past-the-maximum` |
| 7 | fewer bytes held than the total length | none, the receiver requires the total length | none | The two steps that are not failures | `framing/incomplete-body-one-byte-short`, `limits/frame-at-the-maximum`, `limits/metadata-region-at-the-maximum` |
| 8 | the entries of the metadata region do not fill it exactly | malformed request | connection-fatal | Region Fill | `metadata/region-ends-mid-entry`, `metadata/entry-claims-more-than-the-region-holds` |
| 9 | `kind` travels from the wrong direction | protocol violation | connection-fatal | Direction | `header/frame-kind-from-the-wrong-direction-request-at-a-client`, `header/frame-kind-from-the-wrong-direction-response-at-a-server` |
| 10 | `request id` is 0 on a frame that names a request | protocol violation | connection-fatal | The Reserved Request Id | `correlation/request-frame-with-the-reserved-id`, `correlation/response-frame-with-the-reserved-id`, `correlation/error-frame-with-the-reserved-id-and-a-request-scoped-class` |
| 11 | a frame opens a request whose `request id` is already in flight at the receiver | protocol violation | connection-fatal | A request id already in flight | `correlation/duplicate-in-flight-id` |
| 12 | a frame names a request that is not in flight at the receiver | protocol violation | connection-fatal | A request id not in flight | `correlation/response-naming-an-id-not-in-flight`, `correlation/second-terminal-frame-for-a-retired-id` |
| 13 | `code` on a REQUEST frame carries an opcode this revision does not assign | unsupported operation | request-scoped | Opcodes | `operations/unassigned-opcode-refused`, `operations/opcode-zero-refused`, `operations/opcode-at-the-maximum-refused`, `operations/unassigned-opcode-with-a-payload-refused`, `correlation/request-frame-reusing-a-retired-id` |
| 13 | `code` on a RESPONSE frame carries a result code this revision does not assign | protocol violation | connection-fatal | Result Codes | `results/unassigned-result-code-refused`, `results/unassigned-result-code-at-the-maximum-refused` |
| 13 | `code` on an ERROR frame carries an error class this revision does not assign | protocol violation | connection-fatal | The Error Class Registry | `errors/unassigned-error-class-refused`, `errors/error-class-zero-refused`, `errors/unassigned-error-class-at-the-maximum-refused` |
| 14 | the entries of the metadata region are not strictly ascending | invalid argument | request-scoped | Entry Order | `metadata/entries-out-of-order`, `metadata/duplicate-identifier` |
| 15 | an entry carries an identifier in `0x8000..0xFFFF` that this revision does not assign | invalid argument | request-scoped | The Required Range | `metadata/unassigned-required-identifier-refused` |
| none | an entry carries an identifier in `0x0000..0x7FFF` that this revision does not assign | none, the receiver skips the entry | none | The Optional Range | `metadata/unassigned-optional-identifier-skipped` |

The last row is not a step of the order. A receiver that skips an entry
has not stopped at it: it resumes at the next entry and continues to
apply Entry Order and Region Fill to the rest of the region. The row
appears here because the entry carries an identifier this revision does
not assign, which is the subject of this document, and because the two
ranges of that identifier are read together.

Step 4 carries two rows because one frame separates two receivers that
both enforce the bound. Frame Length Arithmetic states the width a
receiver computes the total length in. The frame of the second row
declares a `payload length` of 4294967295, so its total length is
4294967315, and a computation narrower than 64 bits reads that total as
19: below the bound, and below the 20 bytes the receiver already holds.

The Fixtures That Pin the Order names the fixtures that prove a receiver
meets the rows in the order above.

## Every Input Decided After the Order

A frame that passes all fifteen steps is admitted. A section that defines
a layout for a payload states its own checks over that payload, and
Checks that follow the order places them after step 15.

| The input | The class | The scope | The section that binds it | The fixtures |
|---|---|---|---|---|
| an ERROR frame whose payload is shorter than two bytes | malformed request | connection-fatal | The ERROR Frame Payload | `errors/payload-shorter-than-the-detail-length-field` |
| an ERROR frame whose `detail length` exceeds the bytes that follow it | malformed request | connection-fatal | The ERROR Frame Payload | `errors/detail-length-exceeds-the-payload` |
| a RESPONSE frame carrying the absent result code whose `payload length` is not zero | malformed request | connection-fatal | Absent | `results/absent-with-a-payload-refused` |
| a RESPONSE frame carrying the value held outside memory result code whose `payload length` is not eight | malformed request | connection-fatal | Value held outside memory | `results/value-held-outside-memory-with-a-short-payload-refused` |

A frame refused by one of these rows was admitted by the order, and a
frame refused by the order reaches none of them. The order decides first
in every case.

This revision assigns no opcode, so it defines no payload for a REQUEST
frame and no check over one.

## The Fixtures That Pin the Order

A receiver that ran the steps in another order would answer a frame
violating two of them with the class of the later step. Five fixtures
offer such a frame and state the outcome the earlier step produces.

| The steps | The frame it offers | The fixture |
|---|---|---|
| 2 before 4 | `version` is 1, and the total length exceeds the maximum frame size | `framing/admission-order-version-before-size` |
| 4 before 5 | the total length exceeds the maximum frame size, and a flag bit is set | `framing/admission-order-size-before-flags` |
| 5 before 6 | a flag bit is set, and `metadata length` exceeds the maximum metadata size | `framing/admission-order-flags-before-metadata-size` |
| 9 before 10 | a RESPONSE frame arrives at a server, and its `request id` is 0 | `framing/admission-order-direction-before-request-id` |
| 13 before 15 | `code` carries an unassigned opcode, and the metadata region carries an unassigned required identifier | `metadata/admission-order-code-before-required-identifier` |

Steps 9 and 10 produce one class and one scope, so the fourth fixture
states the outcome of that frame without separating the two steps. The
other four each separate two outcomes: a receiver that ran the later step
first would answer a different class, and the fixture fails against it.
