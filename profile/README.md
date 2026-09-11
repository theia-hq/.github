# You are your key.

Run a service on a machine you own, and reach it from anywhere by the public key it prints. No
account, no control plane, no one's permission.

Today, reaching machines means compromising in one of three ways: run a control plane (Tailscale,
headscale), rent a tunnel (ngrok, Cloudflare Tunnel), or open a port and hope. Identity is an account a
vendor can suspend, and access is a bearer secret someone can lose. You are renting the front door to
your own machine.

The proposition is against the compromise itself: any two people can run a service between them, with
no account, no control plane, and no rented permission.

## It ends with you

Your identity is a key you hold, not a row in someone's database. There is no account to register and
no dashboard to open, so nothing to suspend, migrate, or delete.

> **No company can deplatform what has no account.**

Services sit behind a gate by default. Your own machines and the people you admit get in with a
signature from your key. Everyone else is refused:

> bf01hcq6balrlxwa via iroh: reached, but refused (not admitted: not a member of this node's family, and no capability for this service)
<!-- captured from swoosh/docs/capabilities.md -->

Opening one to anyone takes a deliberate `--public`; a shell can never be public.

## Get it

Install (pre-1.0, not for production yet):

```sh
curl -fsSL https://raw.githubusercontent.com/theia-hq/swoosh/main/scripts/install.sh | sh
```

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

`signet` is the local name you gave a key; `fleet` is the set of devices that key vouches for. Grant one
person one service for 14 days, then revoke it:

```console
$ swoosh contact signet contractor bf014hag2b3nqbyt…
recorded contractor's signet -> bf014hag2b3nqbyt

$ swoosh grant issue ssh --for fleet:contractor --expires 14d
issued a fleet-bound grant for `ssh` to fleet signet bf014hag2b3nqbyt…
  every device that signet vouches for can use it (theft-resistant); expires in 14d
  revoke: swoosh grant revoke bf014hag2b3nqbyt…
sheer:bf013m24wpob5axo…

$ swoosh grant revoke bf014hag2b3nqbyt
revoked 1 grant(s) to bf014hag2b3nqbyt… (…/revoked)
```

First reach, the whole model, every verb: [getting started](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/getting-started.md),
[keys](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/keys.md),
[commands](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/reference/commands.md).

## Compared to what you already use

- **vs Tailscale (rented) or headscale (self-hosted).** Reach works in both, but who you are and who is
  allowed resolve against a control plane, rented or self-hosted: a server, a database of who belongs,
  state to run, secure, and back up. Here admission is a signature your own key already vouches for,
  checked offline, with no coordinator in the trust path.
- **vs ngrok and Cloudflare Tunnel.** Both give HTTP ingress, TLS, domains, browser access, and a free
  tier. The edge sits in the vendor's trust path, and the service is theirs to cut. Here the service runs
  on your machine behind your key; the endpoint is a key, not a rented hostname.
- **vs iroh.** It reaches an ed25519 key over QUIC, then stops: no gate, no roster. This is that reach
  plus the missing half. (bifrost-iroh is iroh underneath; this is the layer iroh chose not to be.)
- **vs libp2p.** libp2p is a toolkit for a swarm: a DHT, pubsub, multiaddrs, transport negotiation, most
  of it for discovering peers you do not know. You already know the peer, it is a key, so none of that is
  needed: addressing is the key.

## The family

Three responsibilities compose. Reach is [bifrost](https://github.com/theia-hq/bifrost), with
[quirk](https://github.com/theia-hq/quirk) as our own QUIC backend: it opens a connection to an ed25519
key over any transport. The gate is [nauthy](https://github.com/theia-hq/nauthy): it decides offline,
against that same key, whether a peer may use one service. The service runtime is
[tightbeam](https://github.com/theia-hq/tightbeam): it hands an admitted peer a raw stream to a named
service. [swoosh](https://github.com/theia-hq/swoosh) is where they come together, one CLI and one
install.

**The stack**

| | |
| --- | --- |
| [bifrost](https://github.com/theia-hq/bifrost) | Reach. Address a peer by its ed25519 key and open a byte stream, wherever it is, across NATs, without knowing its address. Backends: iroh (QUIC with NAT hole-punching and public relays), our own quirk, and an in-process one for tests. It gives the connection and nothing more. |
| [quirk](https://github.com/theia-hq/quirk) | Our own QUIC over UDP, written from scratch: connections, reliable streams, and datagrams by hand. One of bifrost's backends, and it passes the same conformance suite as iroh. |
| [nauthy](https://github.com/theia-hq/nauthy) | The gate. Capability tokens rooted at one key you hold. Mint a grant (bearer or bound): a bearer one can be narrowed and passed on; either can be revoked. Every grant carries an expiry and a revocation id, checked offline against the key the peer already dialed with. No server, no PKI, no allowlist to sync. |
| [tightbeam](https://github.com/theia-hq/tightbeam) | The service runtime. A machine exposes local services under its key, each behind a gate; an admitted peer gets a raw bidirectional stream to one named service. Anything that speaks over a TCP port or Unix socket rides it unchanged. It ships no services of its own; you embed it and supply them. |
| [swoosh](https://github.com/theia-hq/swoosh) | The assembly. One install, one binary: serve services, reach a key, measure the link, ssh in (a keyless shell, membership is the login), send files, forward ports, fetch through a peer, share access. It wires bifrost, nauthy, and tightbeam together. |

**Ready-made nodes**

| | |
| --- | --- |
| [swoosh-action](https://github.com/theia-hq/swoosh-action) | Turn a GitHub Actions runner into a node you reach by key: serve a keyless shell and link diagnostics behind the family gate, with HTTP fetch opt-in, across GitHub's NAT, no port forward, nothing session-identifying in the logs. |
| [qat](https://github.com/theia-hq/qat) | A template for an on-demand machine you `swoosh ssh` into: summoned on demand, up for the minutes you set, gone on `swoosh stop` or at the timer. Across GitHub's NAT, by membership, no ssh keys and no standing VM. |

## What the model makes possible

Two keys are enough to run a service between two people. You run it on a machine you own, and the other
side reaches it by key with no account on either end. Anything that speaks over a TCP port or a Unix
socket can sit behind the gate: a shell, a file drop, a database port, a service you wrote.

Access is a grant rooted at a key, not a second identity to manage. It expires on its own and revokes
without re-keying anyone. A grant names one service, so an admitted person reaches that one service and
nothing else. Who belongs is a signature checked against the key the peer already dialed with. Who is
cut off is a denial your own machine writes.

The same node runs on a laptop, a runner, or an on-demand machine. Tailscale and headscale put identity
and permission in a control plane, rented or self-hosted. Here both live in the key, so no company can
suspend them.

## Try it

Grab a binary from the [latest release](https://github.com/theia-hq/swoosh/releases), or run the install
one-liner above. The install script gives the current release; the links above pin its docs.

Use [qat](https://github.com/theia-hq/qat) when you need a machine to try it against.

Full walkthrough with captured output: [swoosh/docs/demo.md](https://github.com/theia-hq/swoosh/blob/v0.8.0/docs/demo.md).

## Prior art

Tailscale shipped [`tailcat`](https://tailscale.com/blog/tailcat), their data plane with the control
plane stripped out: it reaches a peer with no coordinator, but membership is a flat static nodekey list,
with no names, expiry, revocation, or scoped services. They hand you no revocable membership.

The cryptography is not ours: identity is a plain ed25519 key; the credential is a signature. Theia roots
the gate and the roster in a key you own, not a separate control plane.

---

A bare grant is a bearer token: whoever holds an unexpired, un-revoked one gets that one service, so
issue those scoped and short. A bound one (`--for`) is theft-resistant: a stolen copy only works from a
device it is bound to, or a device the bound signet vouches for. A revoke is node-local and does not cut
a session already open. Reaching across the internet falls back to iroh's public relays when a direct
path fails; they forward encrypted bytes and cannot read them or admit anyone, but they can see who talks
to whom and can drop traffic, and you can self-host them. quirk has no Noise handshake yet, so its
identity is nominal, not proven crypto. Wire protocols, CLIs, and identity formats will change; not for
production yet.
