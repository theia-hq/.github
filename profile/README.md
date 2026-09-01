# Publish your address. Keep the keys to the door.

**You are your key. Your services are you. You're the only authority.**

*WireGuard keys, capability tokens, no control plane.*

You run a named service on your machine (a shell, a file, an HTTP fetch, link diagnostics) and hand
someone a signed grant to reach exactly that one service. The grant expires on its own. The holder can
narrow it and pass it on without asking you. You can revoke it any time. Nothing in the middle checks
it: it verifies against your key alone, with no server, no account, and no coordinator in the trust
path. That is the whole idea, and every tool here is one instance of it.

## Reach is easy. The gate is the point.

Opening an encrypted, NAT-traversing pipe to a public key, with no coordinator, is table stakes now.
Even Tailscale came to the same conclusion, shipping [`tailcat`](https://tailscale.com/blog/tailcat):
their data plane, without their control plane.

What a pipe alone cannot give you is a *gate*: a way to publish a stable address, run something behind it
forever, admit your own devices and your delegates by name, and cut one of them off without moving the
address or disturbing anyone else. That gate, and the grants that pass it, is what lives here.

Reaching a peer is not permission to use it. Every service sits behind a gate that verifies against your
own key, with nothing to phone home to, and rules yes or no for one named service. Your devices carry a
badge you signed; a friend carries a scoped grant you handed them. Both check against your key alone.

**The honest limit.** A grant is still a bearer token: whoever holds an unexpired, un-revoked one gets
that one service until it expires or you revoke it. What you gain is that expiry and revocation exist as
backstops, that a grant is scoped to one service and not your whole machine, and that you can mint a
separate one per person. Mint one grant that never expires and share it like a password, and you have
thrown that away.

## Run a service, hand out a key.

A **service** is a named thing your node runs behind its gate: a shell, an HTTP fetch, a file. It
receives an already-authorized stream and does its work; it cannot run for a peer the gate turned away.
Services are private by default: your gate admits your devices and your delegates and turns everyone else
away. `--public` is the one deliberate opt-out, and a shell refuses it outright.

You **serve** names on your node; a peer **calls** one:

```sh
serve ssh=sshd: fetch=fetch:        # publish a shell and an HTTP fetch, both gated
```

The friendly verbs (`ssh`, `fetch`, `ping`, `speed`) are shortcuts: each names one service, so
`swoosh ssh alice/desk` is just `call` reaching the shell on Alice's desk.

| service | what it is |
| --- | --- |
| a keyless shell (`sshd:`) | A real SSH session with no ssh keys to manage. Membership is the login. Never public. |
| HTTP fetch (`fetch:`) | The node fetches a URL for you and streams it back, TLS terminated at the node, scoped to the URL asked for. |
| link diagnostics (`ping:` / `speed:`) | Round-trip time and throughput to a peer, addressed by key. |
| a file or a pipe (`file:` / `fifo:`) | A path's raw bytes, streamed to the peer: a static file, or a live feed written into a named pipe. |
| push a file (`beam:`) | Send a file or directory to a peer, hashed end to end so a truncated or tampered transfer is rejected. |
| a raw forward | Bind any local `host:port` or Unix socket and reach it as a local port on another machine. |

Stop a node you run with `swoosh stop`, or bound its lifetime up front with `serve --for 30m`. *Coming:
`call` (invoke any named service, the long form the friendly verbs shortcut) and the rest of a node's
status surface, over the same reach + auth machine.*

## Try it

Grab a `swoosh` binary from the [latest release](https://github.com/theia-hq/swoosh/releases):

```sh
swoosh serve                 # on one machine: prints its key, stays reachable behind your gate
swoosh ping <their-key>      # from another, anywhere: reach it by key, across NAT
swoosh speed <their-key>     # measure throughput to it over the real internet path
```

Add `--transport quirk` and the same key reaches the same peer over a QUIC stack we wrote ourselves.
Reach is a swappable seam. Full walkthrough:
[swoosh/DEMO.md](https://github.com/theia-hq/swoosh/blob/main/DEMO.md).

## The family

Grouped by what each one does.

**Reach** (address a peer by its ed25519 key over any transport; the same key reaches the same peer
whichever transport you pick)
| | |
| --- | --- |
| [bifrost](https://github.com/theia-hq/bifrost) | Address a peer by key over any transport. Backends: iroh (internet, relay-backed), our own quirk, and an in-process one for tests. |
| [quirk](https://github.com/theia-hq/quirk) | Our own QUIC over UDP, written from scratch. It passes the same conformance suite as iroh. |

**The gate**
| | |
| --- | --- |
| [nauthy](https://github.com/theia-hq/nauthy) | An identity you own admits your devices and your delegates through signed grants that are expiring, revocable, and can only be narrowed, never widened. No server, no PKI, no allowlist to sync. |

**Services and the CLI**
| | |
| --- | --- |
| [swoosh](https://github.com/theia-hq/swoosh) | The one CLI: reach a key, measure the link, ssh in, fetch through a peer, push files, serve and share services. |
| [tightbeam](https://github.com/theia-hq/tightbeam) | The forward primitive and the registry a node serves its services from. |
| [sshh](https://github.com/theia-hq/sshh) | The keyless SSH server: hand it an authorized stream, it runs the shell. |

**Stand up a node**
| | |
| --- | --- |
| [swoosh-action](https://github.com/theia-hq/swoosh-action) | Turn a GitHub Actions runner into a node you reach by key: ssh in, fetch through it, run diagnostics against it, across GitHub's NAT. |
| [qat](https://github.com/theia-hq/qat) | An example you can try right away: an on-demand box, dormant until you dial it, one running machine while you're in, gone when you leave. |

*Planned: relay and NAT-traversal helpers, and a self-owned storage line. Not started.*

## Where we're going

The end goal is a private network whose control plane and exits are your OWN machines, not a vendor's:
a mesh where the roster of who belongs, and the nodes traffic leaves through, are members you run. No
account sits behind it, so it is yours to inherit or fork, and no company can hold it hostage or lose it.
The first step we've designed is the piece nobody ships: a fresh device joins the fleet by learning a
member list one of your own nodes serves, signed by your key, that no vendor ever held. Add a machine and
it knows the others, with nothing to copy by hand. Designed, not yet built.

---

> A scratchpad, and experimental. Wire protocols, CLIs, and identity formats will change; not for
> production yet.
