# You are your key.

No account for you or the people you share with. Give someone a link to one service on your machine, for as long
as you choose, and revoke it without touching anyone else.

Theia is [swoosh](https://github.com/theia-hq/swoosh), the command-line program that does this, and the Rust libraries
it is built from. swoosh reaches your machines by their public keys, wherever they are, across home routers and NATs.

A machine serves what you name: a shell, a folder that takes files, a local port, or a website it fetches for you.
Your own machines get in. Anyone else needs a link from you, unless you open the service to everyone.

```sh
curl -fsSL https://raw.githubusercontent.com/theia-hq/swoosh/main/scripts/install.sh | sh
```

Prebuilt for x86_64 and aarch64 Linux and Apple Silicon macOS. The script checks the download's SHA-256 checksum
before it installs. The binaries are also on the [releases page](https://github.com/theia-hq/swoosh/releases).

Experimental: the commands, the wire protocol and the key formats change between releases. Not for production use
yet.

**Docs:** [Getting started](https://github.com/theia-hq/swoosh/blob/v0.14.1/docs/getting-started.md) ·
[Share one service](https://github.com/theia-hq/swoosh/blob/v0.14.1/docs/capabilities.md) ·
[Keys](https://github.com/theia-hq/swoosh/blob/v0.14.1/docs/keys.md) ·
[All docs](https://github.com/theia-hq/swoosh/blob/v0.14.1/docs/README.md)

## One program, built from libraries that each do one job

Each library below has its own repository and works without swoosh.

- **[bifrost](https://github.com/theia-hq/bifrost)**: Opens a byte-stream to a machine identified by its public key,
  wherever it is on the internet, across NATs, without knowing its address. In swoosh, it makes every
  connection between machines, over [iroh](https://iroh.computer) by default.
- **[nauthy](https://github.com/theia-hq/nauthy)**: Capability tokens you can revoke with no server, rooted at one
  ed25519 key you hold. In swoosh, it decides who gets in.
- **[tightbeam](https://github.com/theia-hq/tightbeam)**: A node addressed by its public key serves named services
  behind one gate. In swoosh, it serves each service you name.
- **[services](https://github.com/theia-hq/services)**: Four services you can embed: fetch, measure, sshh and transfer.
  In swoosh, they are the website fetch, `ping` and `speed`, the shell, and the folder that takes files.
- **[keystore](https://github.com/theia-hq/keystore)**: Stores one ed25519 secret key in one file, plain or encrypted.
  In swoosh, it keeps each key on disk.
- **[quirk](https://github.com/theia-hq/quirk)**: A QUIC-style transport over UDP, written from scratch to learn how
  one works. In swoosh, it is a transport you can choose instead of iroh.

MIT OR Apache-2.0.
