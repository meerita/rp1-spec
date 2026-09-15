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

## [Unreleased]

Repository foundation. No protocol version is defined yet, and nothing
here is normative.

### Added

- The dual Apache-2.0 or MIT licensing offer.
- A README stating what the repository is, that a conformance claim names
  a revision rather than a protocol version, and that the protocol
  version, the revision, and the server version are independent axes.
- Contribution, security, and conduct policy, including the requirement
  levels a clause uses, the change classification a pull request carries,
  and the rule that a fixture is derived from the contract rather than
  captured from an implementation.
- The Developer Certificate of Origin 1.1, and a check that refuses a
  pull request whose commits carry no matching sign-off.
- An ignore set covering the local working corpus.

No contract is defined in this state. Do not implement against this
repository until a revision is tagged and its documents are marked
stable.
