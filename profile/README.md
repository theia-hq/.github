# Cogito.

*I think, therefore I am. You are your key: identity, and the right to communicate, root in you, not a
server.*

**Reach, name, and run code at a public key, over any transport, across any NAT.** You address _who_, not
_where_. Identity is an ed25519 public key, and everything here is built on that one primitive.

## The principle

Communication should be decentralized: reach anyone by who they are, with no middleman in the path. And so
should the *right* to it: authorization is a capability you hold and delegate, verified against your own
key, offline, with no coordinator. Decentralize both halves, and every application collapses to the same
small thing, a permissioned stream to a key: a shell, a file, a fetch, a woken machine. The repos here are
the first proofs. The substrate is the point.

## Try it

Grab a `swoosh` binary from the [latest release](https://github.com/theia-hq/swoosh/releases):

```sh
swoosh serve                 # on one machine: prints its key, stays reachable
swoosh ping <their-key>      # from another, anywhere: reach it by key, across NAT
```

Run the same command with `--transport quirk` instead of the default, and the same key reaches the same
peer over a completely different QUIC stack. Reach is a seam. Fuller walkthrough:
[swoosh/DEMO.md](https://github.com/theia-hq/swoosh/blob/main/DEMO.md).

## The family

Grouped by the layer each one lives in.

**Reach:** decentralized communication
| | |
| --- | --- |
| [bifrost](https://github.com/theia-hq/bifrost) | The substrate: address a peer by key over any transport. Backends: iroh, mem, and our own quirk. |
| [quirk](https://github.com/theia-hq/quirk) | Our own QUIC over UDP, written from scratch to learn the internals. |

**Auth:** decentralized authorization
| | |
| --- | --- |
| [nauthy](https://github.com/theia-hq/nauthy) | The gate: a signet you own admits your devices and anyone you delegate to, with attenuable, expiring, revocable capability tokens. No server, no PKI. |

**Apps:** built on reach and auth
| | |
| --- | --- |
| [swoosh](https://github.com/theia-hq/swoosh) | The one human CLI: every operation a verb, one binary, one identity. |
| [tightbeam](https://github.com/theia-hq/tightbeam) | The exposer primitive: serve a local service by key, gated by who you allow. |
| [sshh](https://github.com/theia-hq/sshh) | A keyless SSH server: hand it an already-authenticated stream, it runs the shell. |
| [iris](https://github.com/theia-hq/iris) | A verified file courier: send to a key, checked end to end. |
| [swoosh-action](https://github.com/theia-hq/swoosh-action) | `swoosh ssh` into a GitHub Actions runner, by membership, across the NAT. |
| [qat](https://github.com/theia-hq/qat) | A template for an on-demand box you dial into: dormant until you call it, gone when you leave. |

_Planned: relay and NAT-traversal helpers, and a self-owned storage line. Not started._

## Where we're going

The substrate wants more than tunnels. On the frontier: presence (something online when you are not),
authorized wake-on-knock compute (dormant names that cost nothing at rest), self-owned storage (your
media, held by you), region-picked egress over your own trust graph, and pubkey-native names. Arbitrary
applications, on communication and auth that answer to no one but you.

---

> A scratchpad, and experimental. Wire protocols, CLIs, and identity formats will change; not for
> production yet.
