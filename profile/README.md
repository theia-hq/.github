# You are your key.

Run a service on a machine you own, and reach it from anywhere by the public key it prints. No
account, no control plane, nothing in the middle.

Today, reaching machines means compromising in one of three ways: run a control plane (Tailscale,
headscale), rent a tunnel (ngrok, Cloudflare Tunnel), or open a port and hope. Identity is an account a
vendor can suspend, and access is a bearer secret someone can lose. You are renting the front door to
your own machine.

The proposition is against the compromise itself: any two people can run a service between them, with
no account, no middle, and no rented permission.

## It ends with you

Your identity is a key you hold, not a row in someone's database. There is no account to register and
no dashboard to open, so nothing to suspend, migrate, or delete. `swoosh serve --expires 30m` reaps
itself when the timer runs out; there was never an account to deregister.

> **No company can deplatform what has no account.**

Every service sits behind a gate. Your own machines and the people you admit get in with a signature
from your key. Everyone else is refused:

> bf01hwttmgsklixr via quirk: reached, but refused (not admitted: not a member of this node's family, and no capability for this service)
<!-- captured from swoosh/docs/demo.md (scripts/demo.sh) -->

## Get it

```sh
curl -fsSL https://raw.githubusercontent.com/theia-hq/swoosh/main/scripts/install.sh | sh
```

## What you get in 60 seconds

Reach a machine by key across NAT with no account. Hand out a grant that expires on its own and can be
cut at any time. The commands below ran against the released binary:

<!-- Captured 2026-09-11 from swoosh v0.8.0 (aarch64-macos release); long keys truncated with `…`; the serve banner is trimmed to the readiness key and the served services, and the ping path line is omitted. -->

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

Enroll a second machine (`swoosh mint` on the first, `swoosh adopt` on the second), then reach the first
by the key it printed:

```console
$ swoosh ping bf013m24wpob5axo
  4 sent, 4 received, 0% loss
  rtt min/avg/max/mdev = 2.139/179.306/266.663/88.583 ms

$ swoosh ssh bf013m24wpob5axo -- echo hello
hello
```

Grant one person one service for 14 days, then revoke it:

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

First reach, the whole model, every verb: [getting started](https://github.com/theia-hq/swoosh/blob/main/docs/getting-started.md),
[keys](https://github.com/theia-hq/swoosh/blob/main/docs/keys.md),
[commands](https://github.com/theia-hq/swoosh/blob/main/docs/reference/commands.md).

## Compared to what you already use

- **vs Tailscale (rented) or headscale (self-hosted).** Reach works in both, but who you are and who is
  allowed resolve against a control plane you operate: a server, a database of who belongs, state to run,
  secure, and back up. Here admission is a signature your own key already vouches for, checked offline,
  with no coordinator in the path.
- **vs iroh.** It reaches an ed25519 key over QUIC, then stops: no gate, no roster. This is that reach
  plus the missing half. (bifrost-iroh is iroh underneath; this is the layer iroh chose not to be.)
- **vs libp2p.** libp2p is a toolkit for a swarm: a DHT, pubsub, multiaddrs, transport negotiation, most
  of it for discovering peers you do not know. You already know the peer, it is a key, so none of that is
  needed: addressing is the key, admission is a signature.

## The family

Grouped by what each one does.

**Reach** (address a peer by its ed25519 key over any transport)

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
| [swoosh](https://github.com/theia-hq/swoosh) | The one CLI: reach a key, measure the link, ssh in, fetch through a peer, push files, serve and share services. `swoosh ssh <key>` opens a shell with no ssh keys to manage: membership is the login, and the shell is never public. |
| [tightbeam](https://github.com/theia-hq/tightbeam) | Reach a service by key, pass its gate, get a raw stream. The registry a node serves its services from. |

**Stand up a node**

| | |
| --- | --- |
| [swoosh-action](https://github.com/theia-hq/swoosh-action) | Turn a GitHub Actions runner into a node you reach by key: ssh in, fetch through it, run diagnostics against it, across GitHub's NAT. |
| [qat](https://github.com/theia-hq/qat) | An example you can stand up yourself: an on-demand box, dormant until you dial it, one running machine while you're in, gone when you leave. |

## Try it

Grab a binary from the [latest release](https://github.com/theia-hq/swoosh/releases), or run the install
one-liner above.

Need a box to try it against? [qat](https://github.com/theia-hq/qat) is a template for an on-demand box:
dormant until you dial it, gone when you leave.

Full walkthrough with captured output: [swoosh/docs/demo.md](https://github.com/theia-hq/swoosh/blob/main/docs/demo.md).

## Prior art

Even Tailscale conceded the split. They shipped [`tailcat`](https://tailscale.com/blog/tailcat), their
data plane with the control plane stripped out, so you reach a peer with no coordinator. The moment you
want a gate back, who is allowed and who you are, they hand you nothing and you are operating a control
plane again. Theia keeps the gate and the roster in the same key that does the reach.

The cryptography is not ours: identity is a plain ed25519 signature, admission a capability token
(biscuit), both older than this project. Nothing here asks you to trust a new cipher.

---

A grant is a bearer token: whoever holds an unexpired, un-revoked one gets that one service, so mint
them scoped, short, and per person. A revoke is node-local and does not cut a session already open.
quirk has no Noise handshake yet, so its identity is nominal, not proven crypto. Wire protocols, CLIs,
and identity formats will change; not for production yet.
