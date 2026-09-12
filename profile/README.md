# You are your key.

Any two people can run services between their own machines. The way in is a signature from your key,
checked offline. No account, no control plane, no one's permission.

Today, reaching machines usually means compromising in one of three ways: run a control plane
(Tailscale, headscale), rent a tunnel (ngrok, Cloudflare Tunnel), or open a port and hope. Identity is
an account a vendor can suspend, and access is a bearer secret someone can lose. You are renting the
front door to your own machine.

## It ends with you

Your identity is a key you hold, not a row in someone's database. There is no account to register and
no dashboard to open, so nothing to suspend, migrate, or delete.

> **No company can deplatform what has no account.**

Services sit behind a gate by default. Your own machines and the people you admit get in with a
signature from your key. Everyone else is refused:

> bf01hcq6balrlxwa via iroh: reached, but refused (not admitted: not a member of this node's family, and no capability for this service)
<!-- captured from swoosh/docs/capabilities.md on main -->

Opening one to anyone takes a deliberate `--public`; the keyless shell service (`sshd:`) is refused.

## The family

[bifrost](https://github.com/theia-hq/bifrost) is how a peer is reached. It addresses the peer by its
ed25519 key and opens a stream, and the transport underneath is swappable. Today that is iroh, QUIC with
NAT traversal and a fallback to public relays, and [quirk](https://github.com/theia-hq/quirk), our own
QUIC-style transport written from scratch. Most readers will never touch quirk; we wrote it to understand
how this layer works. One limit today: both ends have to be online and findable for NAT traversal to
connect them. Transports that work without both ends online are where this goes next.

[nauthy](https://github.com/theia-hq/nauthy) decides who gets in: capability tokens rooted in your
key, checked offline against the key that dialed. The trust is in the math, and the math is standard
and not ours: ed25519 keys and biscuit tokens. The only operator is you.

[tightbeam](https://github.com/theia-hq/tightbeam) gives the node a set of named services; each sits
behind its own gate, and an admitted peer gets a raw stream to the one it asked for and nothing else.
It ships no services of its own, so you define the set.

[swoosh](https://github.com/theia-hq/swoosh) is the one CLI and one install: serve a service, reach a
key, measure the link, ssh in, send files, forward ports, fetch through a peer, share access.

Every layer here is a crate you can build on: reach, the gate, and the service runtime stand alone, and
the service engines are not tied to swoosh. [swoosh](https://github.com/theia-hq/swoosh) is the one we
built to show them working together.

**Ready-made nodes**

| | |
| --- | --- |
| [swoosh-action](https://github.com/theia-hq/swoosh-action) | Turn a GitHub Actions runner into a node you reach by key: serve a keyless shell and link diagnostics behind the family gate, with HTTP fetch opt-in, across GitHub's NAT, no port forward, nothing session-identifying in the logs. |
| [qat](https://github.com/theia-hq/qat) | A template for an on-demand machine you `swoosh ssh` into: summoned on demand, up for the minutes you set, gone on `swoosh stop` or at the timer. Across GitHub's NAT, by membership, no ssh keys and no standing VM. |

## Get it

Install (prebuilt for x86_64 and aarch64 Linux and Apple Silicon macOS; pre-1.0, not for production yet):

```sh
curl -fsSL https://raw.githubusercontent.com/theia-hq/swoosh/main/scripts/install.sh | sh
```

Grab a binary from the [latest release](https://github.com/theia-hq/swoosh/releases) instead if you
prefer. The install script gives the current release; the doc links below point at that release's docs.
Use [qat](https://github.com/theia-hq/qat) when you need a machine to try it against.

## Serve, reach, grant, revoke

Reach a machine by key across NAT with no account. Hand out a grant that expires on its own and can be
cut at any time. The commands below ran against the released binary:

<!-- Captured 2026-09-11 from swoosh v0.8.0 (aarch64-macos release); long keys truncated with `…`; the serve banner is trimmed to the readiness key and the operator-served rows; the reach section, the control row, and the stop line are omitted. The ping path line is omitted. -->

Serve on the machine you want to reach:

```console
$ swoosh serve ssh=sshd: ping=ping:
swoosh ready

    bf013m24wpob5axozjmmzv2w3nikjto6nllmwpfiuij7qemsrd3m4hgq

serving
  family-gated   your devices + peers you've granted
    ping          round-trip probe
    ssh -> sshd   a shell on this machine
```

Enroll a second machine (`swoosh mint` on the first, `swoosh adopt` on the second; [getting
started](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/getting-started.md)), then reach the first
by the key it printed:

```console
$ swoosh ping bf013m24wpob5axozjmmzv2w3nikjto6nllmwpfiuij7qemsrd3m4hgq
  4 sent, 4 received, 0% loss
  rtt min/avg/max/mdev = 2.139/179.306/266.663/88.583 ms

$ swoosh ssh bf013m24wpob5axozjmmzv2w3nikjto6nllmwpfiuij7qemsrd3m4hgq -- echo hello
hello
```

`signet` is the local name you gave a key; `fleet` is the set of devices that key vouches for. A
`sheer:` link is the grant you hand out. Grant one person one service for 14 days, then revoke it:

```console
$ swoosh contact signet contractor bf014hag2b3nqbyt…
recorded contractor's signet -> bf014hag2b3nqbyt

$ swoosh grant issue ssh --for fleet:contractor --expires 14d
issued a fleet-bound grant for `ssh` to fleet signet bf014hag2b3nqbyt…
  every device that signet vouches for can use it (theft-resistant); expires in 14d
  revoke: swoosh grant revoke bf014hag2b3nqbyt…
sheer:bf013m24wpob5axo…

$ swoosh grant revoke bf014hag2b3nqbyt…
revoked 1 grant(s) to bf014hag2b3nqbyt… (…/revoked)
```

First reach, the whole model, every verb: [getting started](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/getting-started.md),
[keys](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/keys.md),
[commands](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/reference/commands.md). Full walkthrough
with captured output: [swoosh/docs/demo.md](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/demo.md).

## What the model makes possible

Two keys are enough to run a service between two people. You run it on a machine you own, and the other
side reaches it by key with no account on either end. Anything that speaks over a TCP port or a Unix
socket can sit behind the gate: a shell, a file drop, a database port, a service you wrote.

Access is a grant rooted at a key, not a second identity to manage. It expires on its own and revokes
without re-keying anyone. A grant names one service, so an admitted person reaches that one service and
nothing else. Who belongs is a signature checked against the key the peer already dialed with. Who is
cut off is a denial your own machine writes.

The same node runs on a laptop, a runner, or an on-demand machine.

## Compared to what you already use

- **vs Tailscale (rented) or headscale (self-hosted).** Both give you reach and identity, with a control
  plane you rent or operate: a server, a database of who belongs, state to run, secure, and back up.
  This gives you that reach and identity without the control plane, and then past it: a grant here is
  scoped to one service, expiring, revocable, delegable, and checked offline against the key that
  dialed.
- **vs ngrok and Cloudflare Tunnel.** Both give HTTP ingress, TLS, domains, browser access, and a free
  tier. The edge sits in the vendor's trust path, and the service is theirs to cut. Here the service runs
  on your machine behind your key; the endpoint is a key, not a rented hostname.
- **vs iroh.** It reaches an ed25519 key over QUIC, then stops: no gate, no roster. This is that reach
  plus the missing half. (bifrost-iroh is iroh underneath; this is the layer iroh chose not to be.)
- **vs libp2p.** libp2p is a toolkit for a swarm: a DHT, pubsub, multiaddrs, transport negotiation, most
  of it for discovering peers you do not know. You already know the peer, it is a key, so none of that is
  needed: addressing is the key.

## What Tailscale shipped next

Tailscale open-sourced [`tailcat`](https://tailscale.com/blog/tailcat) on 2026-08-31, after swoosh
v0.1.0 shipped on 2026-08-27. It is their data plane with the control plane stripped out. It concedes
reach with no coordinator. It does not concede the gate or the roster. Admission is possession of the
address, with an optional flat static nodekey allow-list. There is no expiry, no delegation, and no
per-service capability. They hand you no revocable membership.

---

**A bare grant is a bearer token.** Whoever holds an unexpired, un-revoked one gets that one service,
so keep it short-lived. A bound grant (`--for`) is theft-resistant: a stolen copy only works from a
device it is bound to, or a device the bound signet vouches for. A revoke is node-local: the next dial
is refused, and a session already open is not cut.

**The relay fallback is iroh's.** When a direct path fails, iroh's public relays forward encrypted
bytes and cannot read them or admit anyone. They can see who talks to whom and can drop traffic.
Pointing swoosh at your own relay is not wired up yet.

**Early software.** quirk has no Noise handshake yet, so its identity is nominal, not proven crypto,
and it is direct-only (LAN or an address you pass with `--peer`). The crates are consumed as git
dependencies today; none of them is published to crates.io yet. Wire protocols, CLIs, and identity
formats will change; not for production yet.
