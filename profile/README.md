# Reach a public key, prove you're allowed, run a service at it.

Give a machine another machine's public key and it opens a private, authenticated byte-stream to that
exact key, over any transport, across any NAT. You address _who_, not _where_. On top of that one reach,
a node runs named services (a shell, a file, an HTTP fetch, link diagnostics), each gated so only people
you allow can use it. That is the whole idea, and every tool here is one instance of it.

## The foundation: two primitives

Everything stands on two things. Nothing above them works without both, and together they are the entire
trust story: no server, no account, no coordinator in the path.

**Reach.** Give it an ed25519 public key and it opens a mutually authenticated, encrypted,
NAT-traversing byte-stream to exactly that key. The transport underneath is swappable and the same key
reaches the same peer whichever one you pick.

| | |
| --- | --- |
| [bifrost](https://github.com/theia-hq/bifrost) | Address a peer by its public key over any transport. Backends: iroh (internet, relay-backed), our own quirk, and an in-process one for tests. |
| [quirk](https://github.com/theia-hq/quirk) | Our own QUIC over UDP, written from scratch to own the internals. It passes the same conformance suite as iroh. |

**Auth.** Reaching a peer is not permission to use it. The gate verifies offline against your own
identity, with nothing to phone home to, and rules yes or no for a specific service.

| | |
| --- | --- |
| [nauthy](https://github.com/theia-hq/nauthy) | The gate at the door. An identity you own admits your own devices and anyone you delegate to, through signed tokens that are expiring, revocable, and can only be narrowed, never widened. No server, no PKI, no allowlist to sync. |

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
