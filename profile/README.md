# You are your key.

Run a service on your machine and it answers to you and only you, verified against your key alone. No
account. No control plane. Nothing in the middle. You cannot be deplatformed from a thing that has no
platform.

## The one idea

A network is made of three things, and today you rent all three. Reachability, from Tailscale or
Cloudflare or ngrok. Identity, an account someone else can suspend. The roster of who belongs, a control
plane you cannot hold or audit. Here, all three are things you **run**: your address is a key you hold,
your services answer to that key, and the list of who is allowed is served by your own machine and signed
by your own key. Nothing to phone home to, because there is nothing in the middle.

Even Tailscale conceded the split. They shipped [`tailcat`](https://tailscale.com/blog/tailcat), a build
of their networking with the control plane stripped out, so you reach a peer with no coordinator. But the moment you
want a gate back (who is allowed, who you are), they hand you nothing, and you are renting or self-hosting
a control plane again. Theia keeps the gate **and** the roster in the same key that does the reach.

Against the tools you already use:

- **vs Tailscale (rented) or headscale (self-hosted).** It reaches your peers, but who you are and who is
  allowed both resolve against a control plane: a server, a database of who belongs, state to run,
  secure, and back up. Headscale is the honest move, self-host that plane instead of renting it, but
  either way you are operating one. Here there is nothing to host. Admission is a signature your own key
  already vouches for, checked offline, with no coordinator in the path.
- **vs iroh.** It reaches an ed25519 key over QUIC beautifully, then stops: no gate, no notion of "is this
  peer allowed," no roster. Theia is that same reach plus the missing half. (bifrost-iroh *is* iroh
  underneath; this is the layer iroh chose not to be.)
- **vs libp2p.** libp2p is a toolkit for building a swarm: a DHT, pubsub, multiaddrs, transport
  negotiation, a protocol zoo, most of which you run to discover peers you do not know. Here you already
  know the peer, it is a key, so all of that disappears: addressing is the key, membership is a signature,
  and two peers talk with no distributed system to stand up first.

## Nothing to take away

Your identity is not a row in someone's database. It is a key you hold. There is no account to register,
no dashboard to open, no coordinator in the path. A box reachable only by your key can do its work and
then be **gone**: `serve --for 30m` reaps itself when the timer runs out, and there was never anything to
deregister because there was never an account.

> **No company can deplatform what has no account.**

## It is real, and it runs today

Reach a machine by its key, no login, across NAT:

```sh
swoosh serve                 # on one machine: prints its key, stays reachable behind your gate
swoosh ping <their-key>      # from anywhere else: reach it by key
```

No account was created. No dashboard was opened. No tunnel URL was registered. You reached a machine by
its key. And now you can just use it:

```sh
swoosh ssh <their-key>          # a real shell on it, keyless: no ssh keys to manage, membership is the login
swoosh speed <their-key>        # measure throughput to it over the real internet path
swoosh fetch --via <their-key> <url>   # pull a URL through the box, out its network
```

No account, no port forward, no tunnel URL. You address the box by key and do real work on it.

**The transport is swappable by design.** A peer is addressed by key, and the transport under that key is
an interface, not a lock-in: one trait, a shared conformance suite, a runtime `--transport` flag. iroh is
the default here, the real, encrypted, relay-backed transport the demos run on. To prove that is a genuine
interface and not one implementation wearing an abstraction, we also wrote our own QUIC-shaped transport
from scratch, **quirk**, and it already passes the same conformance suite iroh does:

```sh
swoosh ping <their-key> --transport quirk
```

quirk is early. We built it to understand QUIC from the inside, so it has no Noise handshake yet and is
not a production transport. But it proves the point that matters: two independently written transports run
under the same key, so `--transport` is a real choice, not a diagram.

## The shape, at three radii

The gate, the reach, and the `serve` verb never change. Only who is trusted widens. It is one shipped
shape at three radii, not three unbuilt features.

**One box (today).** A machine that answers only to your key. Run a named service and it is reachable by
your key alone, over any transport, across NAT, with no account and no control plane in the trust path.

**Your fleet (next).** You already ssh into box1, box2, box3 by key today, each one gated to the same key.
What is designed, not yet built, is the convenience: a fresh machine learns your other boxes from a signed
member list one of your own nodes serves, so you add a machine and it knows the rest, nothing copied by
hand.

**A network you own together (the north star).** Fleets that join into a network people run together,
members hosting exits and relays for each other, two fleets interlinking by vouching for each other's
members. This is the private overlay you would otherwise rent from a Tailscale or a Mullvad, except it is
member-hosted and the authority stays yours. Owning it never means owning the
metal: rent a machine from anyone and move it whenever you like; what you never rent is the say over who
gets in. And it reaches over any transport there is, the same key over iroh or quirk today, any medium
tomorrow. The exit you reach in another region is a friend's machine, admitted against keys you hold, not
a vendor's.

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
| [swoosh](https://github.com/theia-hq/swoosh) | The one CLI: reach a key, measure the link, ssh in, fetch through a peer, push files, serve and share services. A keyless shell (`swoosh ssh <key>`) is the marquee: a real SSH session with no ssh keys to manage, membership is the login, never public. |
| [tightbeam](https://github.com/theia-hq/tightbeam) | The forward primitive and the registry a node serves its services from. |

**Stand up a node**

| | |
| --- | --- |
| [swoosh-action](https://github.com/theia-hq/swoosh-action) | Turn a GitHub Actions runner into a node you reach by key: ssh in, fetch through it, run diagnostics against it, across GitHub's NAT. |
| [qat](https://github.com/theia-hq/qat) | An example you can stand up yourself: an on-demand box, dormant until you dial it, one running machine while you're in, gone when you leave. |

## Try it

Grab a `swoosh` binary from the [latest release](https://github.com/theia-hq/swoosh/releases):

```sh
swoosh serve                       # prints its key, stays reachable behind your gate
swoosh ssh <their-key>             # a real shell on it, no ssh keys, membership is the login
swoosh ping <their-key> --transport quirk   # same key, same peer, over a QUIC stack we wrote
```

Want a box to try it against? [qat](https://github.com/theia-hq/qat) is a template: install swoosh,
`swoosh mint qat`, set the authkey, then summon and dial. A few minutes, no standing VM, and you have an
on-demand box you ssh into by key.

Full walkthrough: [swoosh/DEMO.md](https://github.com/theia-hq/swoosh/blob/main/DEMO.md).

---

A grant is a bearer token: whoever holds an unexpired, un-revoked one gets that one service, so mint them
scoped and short and per person, and expiry and revocation are yours as backstops. Wire protocols, CLIs,
and identity formats will change; not for production yet.
