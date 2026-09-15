# RP-1 Spec

RP-1 Spec is the public, normative definition of the RP-1 Native
Protocol.

It is the contract between the RP-1 server and every independent
implementation of the protocol. Official and third-party clients are
written against this repository, and conformance is measured against it.

It is not documentation of the RP-1 server, and it is not an SDK guide.

## Status

Revision v0.1.0 defines protocol version 0 at the framing and codec
surface. Every document of that revision carries the status `draft`.

From that revision an implementer writes an encoder and a decoder for
every frame kind it assigns, and receiver behavior for every input it
permits. The revision defines no handshake, no negotiation, no
capability, no operation, and no conformance layer, so what it produces
is a codec: it is not a client, and it does not interoperate with a
deployed peer.

A draft contract changes without the treatment a classified change
carries. An implementation built against revision v0.1.0 is built at the
implementer's own risk, and pins the revision it was built against.

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

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md), which
states the requirement levels a clause uses, the classification every
change carries, and the rule that a fixture is derived from the contract
rather than captured from an implementation.

[CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) applies to every interaction in
this repository.

## Security

A security problem here is a defect in the contract, not in a program: a
clause an implementation cannot satisfy safely. See
[SECURITY.md](SECURITY.md). Report it privately rather than in a public
issue.

## License

Licensed under either of:

* Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
* MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in this repository, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms
or conditions.
