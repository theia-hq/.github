# Publish your address. Keep the keys to the door.

**You are your key. Your services are you. Your only authority.**

You run a named service on your machine (a shell, a file, an HTTP fetch, link diagnostics), and you
hand someone a signed grant to reach exactly that one service. The grant expires on its own, the holder
can narrow it and pass it on without asking you, and you can revoke it at any time. Nothing in the
middle checks it: it verifies offline against your key alone, with no server, no account, and no
coordinator in the trust path. That is the whole idea, and every tool here is one instance of it.

Reaching a peer is table stakes now. Anyone can open an encrypted, NAT-traversing pipe to a public key
with no coordinator (even Tailscale unbundled it, as `tailcat`). What that pipe alone cannot give you
is a *gate*: a way to publish a stable address, run something behind it forever, admit your own devices
and your delegates by name, and cut one of them off without moving the address or disturbing anyone
else. That gate, and the grants that pass it, is what lives here.

## The foundation: reach, then the gate

Everything stands on two things: reaching a key, and deciding who gets in. The first is table stakes;
the second is the point.

**A gate, and grants that pass it.** Reaching a peer is not permission to use it. Every service sits
behind a gate that verifies offline against your own identity, with nothing to phone home to, and rules
yes or no for one specific service. Your devices carry a badge you signed; a friend carries a scoped
grant you handed them. Both are checked against your key alone.

| | |
| --- | --- |
| [nauthy](https://github.com/theia-hq/nauthy) | The gate at the door. An identity you own admits your own devices and anyone you delegate to, through signed tokens that are expiring, revocable, and can only be narrowed, never widened. No server, no PKI, no allowlist to sync. |

**Reach.** Give it an ed25519 public key and it opens a mutually authenticated, encrypted,
NAT-traversing byte-stream to exactly that key. You address _who_, not _where_. The transport underneath
is swappable and the same key reaches the same peer whichever one you pick.

| | |
| --- | --- |
| [bifrost](https://github.com/theia-hq/bifrost) | Address a peer by its public key over any transport. Backends: iroh (internet, relay-backed), our own quirk, and an in-process one for tests. |
| [quirk](https://github.com/theia-hq/quirk) | Our own QUIC over UDP, written from scratch to own the internals. It passes the same conformance suite as iroh. |

**The honest limit.** A leaked grant is still a bearer token: whoever holds an unexpired, un-revoked
grant gets that one service until it expires or you revoke it. The win is that expiry and revocation
exist as backstops, that a grant is scoped to one service and not your whole machine, and that you can
mint a separate one per person. Mint one grant that never expires and share it like a password and you
have thrown that away.

## The services: what stands on the foundation

A **service** is a named thing a node runs behind its gate. It receives an already-authorized stream and
does its work; it cannot run for a peer the gate refused, because the authorization is proof it takes by
hand, not a check it could forget. Services are private by default: your node's gate admits your devices
and your delegates, and turns everyone else away. `--public` is the one deliberate opt-out, and some
services (a shell) refuse it outright.

You **serve** names on your node; a peer **calls** one:

```sh
serve ssh=sshd: fetch=fetch:        # publish a shell and an HTTP fetch, both gated
```

The friendly verbs (`ssh`, `fetch`, `ping`, `speed`) are shortcuts: each bakes in one service name, so
`swoosh ssh alice/desk` is just `call` reaching the shell service on Alice's desk.

### Services we have

| service | what it is |
| --- | --- |
| a keyless shell (`sshd:`) | A real SSH session with no ssh keys to manage. Membership is the login. Never public. |
| HTTP fetch (`fetch:`) | The node fetches a URL for you and streams it back, TLS terminated at the node, scoped to the URL asked for. |
| link diagnostics (`diag:`) | Round-trip time and throughput to a peer, addressed by key. |
| a file or a pipe (`file:` / `fifo:`) | A path's raw bytes, streamed to the peer: a static file, or a live feed written into a named pipe. |
| a raw forward | Bind any local `host:port` or Unix socket and reach it as a local port on another machine. |

### Coming

- **`call`**: invoke any named service on a peer, the long form the friendly verbs are shortcuts for.
- **A lifecycle service**: ask a running node its status, or tell it to shut down, over the same reach +
  auth machine as everything else.

## The family, by layer

Grouped by what each one does.

**Reach**
| | |
| --- | --- |
| [bifrost](https://github.com/theia-hq/bifrost) | Address a peer by key over any transport. |
| [quirk](https://github.com/theia-hq/quirk) | Our own QUIC over UDP, from scratch. |

**Auth**
| | |
| --- | --- |
| [nauthy](https://github.com/theia-hq/nauthy) | The gate: an identity you own admits your devices and your delegates, offline. |

**Services + the CLI**
| | |
| --- | --- |
| [swoosh](https://github.com/theia-hq/swoosh) | The one CLI: reach a key, measure the link, ssh in, fetch through a peer, serve and share services. |
| [tightbeam](https://github.com/theia-hq/tightbeam) | The forward primitive and the service registry a node serves its services from. |
| [sshh](https://github.com/theia-hq/sshh) | The keyless SSH server: hand it an authorized stream, it runs the shell. |
| [iris](https://github.com/theia-hq/iris) | Send files to a key, verified end to end. |

**Ready-to-run nodes**
| | |
| --- | --- |
| [swoosh-action](https://github.com/theia-hq/swoosh-action) | Turn a GitHub Actions runner into a node you reach by key: ssh in, fetch through it, run diagnostics against it, across GitHub's NAT. |
| [qat](https://github.com/theia-hq/qat) | A template for an on-demand box: dormant until you dial it, one running machine while you're in, gone when you leave. |

_Planned: relay and NAT-traversal helpers, and a self-owned storage line. Not started._

## Try it

Grab a `swoosh` binary from the [latest release](https://github.com/theia-hq/swoosh/releases):

```sh
swoosh serve                 # on one machine: prints its key, stays reachable behind your gate
swoosh ping <their-key>      # from another, anywhere: reach it by key, across NAT
swoosh speed <their-key>     # measure throughput to it over the real internet path
```

Swap `--transport quirk` for the default and the same key reaches the same peer over a QUIC stack we
wrote ourselves. Reach is a seam. Full walkthrough:
[swoosh/DEMO.md](https://github.com/theia-hq/swoosh/blob/main/DEMO.md).

## Where we're going

The next canvas is running code at a key: a function you `call` by public key, woken by the reach,
metered while it runs, back to nothing when you leave. Same foundation, same gate, a new kind of service
behind it. Named but not started: presence, self-owned storage, region-picked egress over your own trust
graph, and pubkey-native names you can type into any app.

---

> A scratchpad, and experimental. Wire protocols, CLIs, and identity formats will change; not for
> production yet.
