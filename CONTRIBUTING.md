# Contributing to RP-1 Spec

Thank you for your interest in RP-1 Spec.

This document states what the repository expects from a change. Read
[CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) as well.

## Before you start

RP-1 Spec is the public, normative definition of the RP-1 Native
Protocol. It is the contract between the RP-1 server and every
independent implementation. It is not documentation of the server, and
it is not an SDK guide.

No protocol version is published yet. The repository is being built, so
open an issue before you write a large change.

## The reader

Every clause is written for one reader: an engineer implementing the
protocol from this repository alone, with no access to the RP-1 server,
no access to any SDK, and no way to ask a question.

Before you propose a clause, decide what that reader must be able to do
after reading it. If the clause does not let them do it, it is not ready.

## What a requirement looks like

Requirement levels follow
[BCP 14](https://www.rfc-editor.org/info/bcp14) (RFC 2119, RFC 8174).
Only an uppercase keyword is normative. A lowercase "should" carries no
requirement level and does not belong in normative text.

Use `MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, and `MAY`. Do not use
`SHALL`; it carries no meaning `MUST` lacks.

A keyword answers one question. A requirement answers three:

```text
who does this bind?          client, server, or both
what exactly must they do?   one observable behavior
what happens otherwise?      a named, defined consequence
```

This is why a keyword alone is not enough:

```text
incomplete
  A client MUST NOT reuse a request id while a request with that id is
  in flight.

complete
  A client MUST NOT reuse a request id while a request with that id is
  in flight. A server that receives a duplicate in-flight request id
  MUST treat it as a protocol violation and close the connection.
```

The first sentence binds the client and leaves the server
unimplementable. A prohibition with no stated enforcement cannot be
tested.

Two further rules keep the levels honest:

- Every `SHOULD` states the valid reason to deviate and the cost of
  deviating. A `SHOULD` with neither is a `MUST` the author was reluctant
  to write, and implementers will split on it.
- Every `MAY` states what the other peer `MUST` accept because of it. A
  `MAY` with no matching obligation on the receiver is an
  interoperability defect.

Do not use `SHOULD` to avoid a decision. An undecided behavior is an open
question, and it is better recorded as one.

## What does not belong in a clause

```text
"appropriately"     state the behavior
"as expected"       state the expectation
"handles"           state what handling means on the wire
"is an error"       name the class and the scope
"undefined"         state the receiver behavior
"implementation-defined" alone   state the bound on the freedom
```

Each of these reads as complete and fails the only test that matters: two
independent implementers produce incompatible code from it.

## Classify your change

Every change carries exactly one class. State it in the pull request.

```text
editorial
  wording, structure, ordering, terminology, examples.
  No conforming peer changes. No fixture changes.

clarification
  a requirement that was already derivable is now stated.
  No conforming peer changes. Fixtures may be added, never altered.

defect correction
  published text was wrong, contradictory, or not implementable.
  A conforming peer may change. Fixtures may be altered.

addition
  new optional surface, reached through negotiation.
  A peer built against the previous revision remains conforming.

breaking change
  a peer built against the previous revision no longer conforms.
```

`editorial` is the most over-claimed class. Before you claim it, ask
whether a reader could have implemented differently from the old text,
whether any fixture changes, and whether the change adds, removes, or
alters an uppercase keyword. A yes to any of them means the change is a
clarification or larger. Removing an ambiguity is never editorial: the
ambiguity was a real state the contract was in, and implementations were
built inside it.

A change whose class is unclear is treated as breaking until the analysis
says otherwise.

## Fixtures

Normative fixtures carry the same authority as the prose they exercise.

A change that alters what a conforming peer emits or accepts updates its
fixtures in the same change. A new requirement arrives with the fixtures
that can fail against it.

**A fixture is derived from the contract. It is never captured from an
implementation.** Running an implementation to check a fixture is useful.
Generating a fixture from an implementation's output encodes that
implementation's defects as the contract, and every later implementation
inherits them.

A decode fixture states the exact failure class and the exact failure
scope. "Rejects" is not an expected outcome: two implementations can
reject for different reasons and one of them is wrong.

An encode fixture pins bytes exactly, which is only possible once the
contract has removed every encoding freedom for that frame. If you cannot
write the fixture because the clause leaves a choice open, the clause is
not finished.

## Implementation behavior

The RP-1 server and the official SDKs are evidence. None of them is the
contract.

An implementation can show you that a question exists, that a clause is
ambiguous, that a clause is not implementable, or that a candidate answer
has a working precedent and a known cost. Each is a good reason to
propose a change.

These are not reasons:

```text
an implementation already behaves differently
changing the implementation would be expensive
another implementation does it this way
```

When the contract is right and an implementation is wrong, the outcome is
a report to that implementation, not a change here.

Do not include private source paths, module names, types, or symbols in
a change, a commit message, or a pull request description.

## Branches and commits

`master` is the integration branch. Work on a topic branch that starts
from `master`, and name it after its objective:

```text
spec/       new or changed normative contract
fix/        correction of a defect in existing normative text
fixtures/   normative test vector work
editorial/  wording and structure, with no contract change
chore/      repository, tooling, and maintenance work
```

The `editorial/` prefix is a claim. A branch carrying it must not change
what any conforming implementation emits, accepts, or rejects.

Write commit subjects as prose imperative sentences, for example
`Define the frame header`. This repository does not use Conventional
Commits.

A commit message states what changed, why it changed, the change
classification, the fixtures added or changed, and what you ran. It
stands on its own for a reader who has only the repository.

Do not credit an AI model, an agent, or an AI process for the work, in a
commit, a pull request, or a comment. This covers the subject, the body,
every trailer, and every co-author line. Naming a tool is not
attribution.

## Pull requests

Open a pull request against `master`, as a draft, and mark it ready when
the change is complete.

State in the description:

- what changed, and why
- the change classification
- the three questions, answered for every new requirement
- the fixtures added or changed
- the effect on a peer built against the previous revision
- any known gap, rather than leaving it to be found

One pull request carries one coherent contract change. Do not combine a
breaking change with an editorial pass: the editorial noise hides the
part that matters.

## Reporting a problem

Open an issue for a clause that is ambiguous, contradictory, or not
implementable. Quote the clause and state the two readings you can see.
"This is unclear" is not actionable; the sentence and the two readings
are.

For a security-relevant defect in the contract, follow
[SECURITY.md](SECURITY.md) instead of opening a public issue.

## Licensing of contributions

Unless you explicitly state otherwise, any contribution you intentionally
submit for inclusion in this repository, as defined in the Apache-2.0
license, is dual licensed as Apache-2.0 or MIT at the recipient's option,
without any additional terms or conditions.
