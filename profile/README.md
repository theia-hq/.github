# You are your key.

Reach a machine by its public key. Grant one person one service, and cut them off without touching
anyone else. No account, no server you rent, no one's permission.

```sh
curl -fsSL https://raw.githubusercontent.com/theia-hq/swoosh/main/scripts/install.sh | sh
```

Prebuilt for x86_64 and aarch64 Linux and Apple Silicon macOS; pre-1.0. A binary from the
[latest release](https://github.com/theia-hq/swoosh/releases) works too. The doc links below point at that
release's docs. [qat](https://github.com/theia-hq/qat) gives you a machine to try it against.

## Serve, reach, grant, revoke

The commands below ran against the released binary:

<!-- Captured 2026-09-15 from swoosh v0.9.0 (aarch64-macos release); long keys truncated with `…`. -->

Serve on the machine you want to reach:

```console
$ swoosh serve ssh=sshd: ping=ping:
swoosh ready

    bf016hqovu7t2eigosog42dttwpr6ypzqdsnr7t5t5ih3xytmi26w56q

serving
  family-gated   your devices + peers you've granted
    ping          round-trip probe
    ssh -> sshd   a shell on this machine
```

Enroll a second machine (`swoosh invite add <label>` on the first, `swoosh adopt` on the second; see
[Getting started](https://github.com/theia-hq/swoosh/blob/v0.9.0/docs/getting-started.md)), then reach
the first by the key it printed, across NAT:

```console
$ swoosh ping bf016hqovu7t2eigosog42dttwpr6ypzqdsnr7t5t5ih3xytmi26w56q
  4 sent, 4 received, 0% loss
  rtt min/avg/max/mdev = 0.437/0.788/1.582/0.397 ms

$ swoosh ssh bf016hqovu7t2eigosog42dttwpr6ypzqdsnr7t5t5ih3xytmi26w56q -- echo hello
hello
```

Anyone else who reaches the node is refused:

> bf01hcq6balrlxwa… via iroh: reached, but refused (not admitted: no member badge or capability for this service was accepted)
<!-- observed on the shipped v0.9.0 binary; the same wording is in swoosh/docs/demo.md (stranger refused over quirk+noise) -->

Now let alice in. `signet` is a person's identity key, recorded under a local name; `fleet` is the set
of devices that key vouches for. A `sheer:` link is the grant you hand out. Grant her one service for 14
days:

```console
$ swoosh contact signet alice bf01o6vqymgz727gazsni37uoify447gropuhsuduzd6lbn4q5iscxfq
recorded alice's signet -> bf01o6vqymgz727g

$ swoosh grant issue ssh --for fleet:alice --expires 14d
issued a fleet-bound grant for `ssh` to fleet signet bf01o6vqymgz727g…
  every device that signet vouches for can use it (theft-resistant); expires in 14d
  revoke: swoosh grant revoke bf01o6vqymgz727gazsni37uoify447gropuhsuduzd6lbn4q5iscxfq
sheer:bf016hqovu7t2eigosog42dttwpr6ypzqdsnr7t5t5ih3xytmi26w56q.…
```

On a device her signet vouches for, alice presents the link when she reaches the node:
`swoosh ssh <node> --present sheer:… -- echo hello`. When you are done with her:

```console
$ swoosh grant revoke bf01o6vqymgz727gazsni37uoify447gropuhsuduzd6lbn4q5iscxfq
revoked 1 grant(s) to bf01o6vqymgz727g… (…/revoked)
```

The grant expired on its own if you forgot. Nobody else's access moved.

First reach, the whole model, every verb: [Getting started](https://github.com/theia-hq/swoosh/blob/v0.9.0/docs/getting-started.md),
[Keys](https://github.com/theia-hq/swoosh/blob/v0.9.0/docs/keys.md),
[Commands](https://github.com/theia-hq/swoosh/blob/v0.9.0/docs/reference/commands.md). The same commands
over two transports, a stranger refused: [Demo](https://github.com/theia-hq/swoosh/blob/v0.9.0/docs/demo.md).

## It ends with you

Today, reaching machines usually means one of three things: run a control plane (Tailscale, headscale),
rent a tunnel (ngrok, Cloudflare Tunnel), or open a port and hope. Identity is an account a vendor can
suspend, and access is a bearer secret someone can lose. You are renting the front door to your own
machine.

Here, your identity is a key you hold, not a row in someone's database. There is no account to register
and no dashboard to open, so nothing to suspend, migrate, or delete.

> **No company can deplatform what has no account.**

Services sit behind a gate by default. The node's family gets in with a signature from your key: your
devices and anyone you grant. Everyone else is refused, with the one line above. Opening a service to
anyone takes a deliberate `--public`; a file, a fifo, or stdin needs the separate `--public-unsafe`. The
keyless shell (`sshd:`) never opens.

Access is a grant rooted at your key, not a second identity to manage. It names one service, expires on
its own, and revokes without re-keying anyone. Who belongs is a signature checked against the key the
peer already dialed with. Who is cut off is a line your own machine writes.

## The pieces

Everything served is a service on a byte stream: a shell, a forwarded port, a file drop, a protocol you
write. A node is a router. Each name binds one service behind the one gate, and an admitted peer gets a
stream to the service it asked for and nothing else. The gate decides who may open the stream; what the
service does with it is the service's business.

[bifrost](https://github.com/theia-hq/bifrost) is how a peer is reached. It addresses the peer by its
ed25519 key and opens a stream, and the transport underneath is swappable. Today that is iroh: QUIC with
NAT traversal and a fallback to public relays. Beside it sits [quirk](https://github.com/theia-hq/quirk), a
transport we wrote from scratch over UDP to learn the layer. It is QUIC-shaped, not QUIC, and direct-only:
no NAT traversal, no discovery beyond the address you hand it. With its Noise handshake it proves the
peer's key and gates work over it; most people will never touch it. One limit today: both ends have to
be online and findable for NAT traversal to connect them.

[nauthy](https://github.com/theia-hq/nauthy) decides who gets in: capability tokens rooted in your key,
checked offline against the key that dialed. The trust is in the math, and the math is standard: ed25519
keys and biscuit tokens, neither of them ours. The only operator is you.

[tightbeam](https://github.com/theia-hq/tightbeam) is the router: named services, one gate in front of all
of them, and a wire that refuses before a byte flows. It ships its own built-ins (`echo:`, forwards, raw
streams); the set you serve is yours.

[services](https://github.com/theia-hq/services) are the engines a node serves: `fetch`, `measure`,
`sshh`, `transfer`. Each is a library that does one job on an admitted stream and depends on none of the
tools.

[swoosh](https://github.com/theia-hq/swoosh) is where reach, the gate, and the engines become one tool:
one CLI, one install. Serve, reach a key, measure the link, ssh in, send files, forward ports, fetch
through a peer, share access.

Every layer is a library you can build on. Two ready-made nodes:

- [swoosh-action](https://github.com/theia-hq/swoosh-action) turns a GitHub Actions runner into a node
  you reach by key.
- [qat](https://github.com/theia-hq/qat) is a template for an on-demand machine you `swoosh ssh` into:
  up when you dispatch it, gone at the timer.

## Compared to what you already use

- **vs Tailscale (rented) or headscale (self-hosted).** Both give you reach and identity, with a control
  plane you rent or operate: a server, a database of who belongs, state to secure and back up. Here there
  is no control plane. The grant goes further: scoped to one service, expiring, revocable, checked offline
  against the key that dialed. A bare grant can be handed on; a bound one cannot.
- **vs ngrok and Cloudflare Tunnel.** Both give HTTP ingress: TLS, domains, and browser access, plus a free
  tier. The edge sits in the vendor's trust path. The service is theirs to cut. Here the service runs on
  your machine behind your key. The endpoint is a key, not a rented hostname.
- **vs iroh.** It reaches an ed25519 key over QUIC, then stops: no gate, no roster. This is that reach
  plus the missing half. (bifrost-iroh is iroh underneath; this is the layer iroh chose not to be.)
- **vs libp2p.** libp2p is a toolkit for a swarm: a DHT, pubsub, multiaddrs, transport negotiation, the
  machinery for finding peers you do not know. You already know the peer, so addressing is the key.

## The same direction

Tailscale's [`tailcat`](https://tailscale.com/blog/tailcat) stripped the control plane out of their data
plane: reach with no coordinator, and it stops there. Admission is address possession; no expiry, no
delegation, no per-service capability. Here membership is a signature rooted in your key.

---

**A bare grant is a bearer token.** Whoever holds an unexpired, un-revoked one gets that one service,
so keep it short-lived. A bound grant (`--for`) is theft-resistant: a stolen copy only works from a
device it is bound to, or a device the bound signet vouches for. A revoke is node-local: the next dial
is refused, and a session already open is not cut.

**Finding a peer and the relay fallback are iroh's.** A serving node publishes where it can be reached to
n0's public discovery service, and when a direct path fails, n0's public relays forward encrypted bytes.
Neither can read traffic or admit anyone. Both can see who talks to whom, and both can go away. Pointing
swoosh at a relay and a resolver you run is not wired today.

**Early software.** Wire protocols, CLIs, and identity formats will change; not for production yet.
