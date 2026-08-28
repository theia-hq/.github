# theia-hq

**Reach, name, and run code at a public key, over any transport, across any NAT.**
You address _who_, not _where_: identity is an ed25519 public key (a `NodeId`), and everything the
overlay does is built on that one primitive.
One machine, one key, one address for everything it does, no matter which transport is carrying it.

## The family

| project | what it is |
| --- | --- |
| [bifrost](https://github.com/theia-hq/bifrost) | The reach substrate. `Transport` / `Session` / `Discovery` / `Node` traits and a transport-agnostic conformance suite. Backends: iroh (real), mem (tests), quirk (ours). |
| [quirk](https://github.com/theia-hq/quirk) | Our own QUIC over UDP, written from scratch to learn the internals (not built on quinn). Passes the same conformance suite as iroh. Noise is phase 1. |
| [iris](https://github.com/theia-hq/iris) | A verified file courier: send files to a public key, verified end to end. |
| [tightbeam](https://github.com/theia-hq/tightbeam) | Private peer-to-peer tunnels (ssh -L / cloudflared shaped, but pubkey-addressed and p2p): `expose` / `connect` / `approve`. |
| [swoosh](https://github.com/theia-hq/swoosh) | The unified CLI front door. One binary, one identity, every p2p operation as a verb. |
| [proxy](https://github.com/theia-hq/proxy) | A standalone SOCKS5 server; tunnel it via tightbeam for an overlay exit. Archived. |
| derpie / maggie / elco / nimbus | Named and planned: relay, NAT traversal, and the drive pieces. Not started. |

## The marquee proof

`swoosh` runs the identical command over iroh and over our own QUIC (quirk) from the same identity: the
same key yields the same NodeId over both transports, so you swap the entire transport stack underneath
an unchanged command and reach the same peer at the same address. Reach is a seam.
See [swoosh/DEMO.md](https://github.com/theia-hq/swoosh/blob/main/DEMO.md).

> Experimental. Wire protocols, CLIs, and identity formats will change; not ready for production use.
