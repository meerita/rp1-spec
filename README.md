# RP-1 Spec

RP-1 Spec is the public, normative definition of the RP-1 Native
Protocol.

It is the contract between the RP-1 server and every independent
implementation of the protocol. Official and third-party clients are
written against this repository, and conformance is measured against it.

It is not documentation of the RP-1 server, and it is not an SDK guide.

## Status

Early development. No protocol version is published yet.

Do not implement against this repository until a revision is tagged and
its documents are marked stable.

## Scope

When published, a revision carries:

```text
the normative documents for one protocol version
the normative fixture corpus for that revision
the changelog entry for that revision
```

Nothing else in this repository is normative.

An implementation claims conformance against a revision, not against a
protocol version alone, and not against a server release.

## Versioning

Three versions evolve independently and are never inferred from one
another:

```text
protocol version   the wire contract a document defines
revision           one published state of this repository
server version     a release of the RP-1 server
```

## Contributing

Contributions are welcome. A change to published normative text carries
a compatibility classification, and a change that alters what a
conforming peer emits or accepts updates its fixtures in the same change.

## Security

Report a suspected protocol-level security defect privately rather than
in a public issue.

## License

Licensed under either of:

* Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
* MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in this repository, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms
or conditions.
