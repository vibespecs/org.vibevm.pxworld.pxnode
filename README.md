# pxworld wrappers for Node.js

The **pxworld** family is a set of thin wrapper packages that run a tool
already on PATH through the proxy vibe manages. A corporate network that
needs px needs it for `npm`, `pip`, `cargo` and `curl` too, and every one of
those tools has its own idea of how a proxy is configured; vibe already knows
how to start px here, so instead of teaching each tool the route, a pxworld
package installs one prefixed command per tool whose launcher re-enters vibe
as `vibe tool px exec -- <tool>`. vibe starts exactly the px its own
operations start — the same `[proxy]` resolution, the same companion, the same
lifetime — hands the child the loopback address in every variable that tool
reads, runs it, stops px when it exits and answers with its exit code. Nothing
about the tool is installed or configured, and nothing is written to the
user's shell profile. This package, `pxnode`, is the first member: Node.js.

```text
vibe install -g org.vibevm.pxworld/pxnode
pxnode --version
pxnpm install
pxnpx prettier --check .
```

| Command | Runs | Instead of |
| --- | --- | --- |
| `pxnode` | the `node` on PATH, through px | `node` |
| `pxnpm` | the `npm` on PATH, through px | `npm` |
| `pxnpx` | the `npx` on PATH, through px | `npx` |

The names are deliberately prefixed. `vibe tool px exec -- node` resolves
`node` on PATH the way the operating system does, and vibe's own command
directory is *ahead* of the system directories on PATH — so a package that
claimed the public command `node` for a launcher that runs `node` would find
its own proxy and re-enter itself forever. `pxnode` names something that was
not there before, and the tool it wraps keeps its own name untouched: `node`
still means the machine's Node.js.

## The limitation

**The tool itself must already be on PATH.** This package ships no payload
binaries at all — no Node.js, no npm. `pxnode` is a wrapper and nothing else:
without a `node` on PATH it fails with one line naming the command. Install a
Node.js first, by whatever means the machine uses, for example

```text
vibe install -g org.vibevm.node/node
```

## What each command actually runs

```text
vibe tool px exec \
  --env NODE_USE_ENV_PROXY=1 \
  --env NODE_USE_SYSTEM_CA=1 \
  --proxy-env npm_config_proxy \
  --proxy-env npm_config_https_proxy \
  -- node <your arguments>
```

`vibe tool px exec` gives the child `HTTPS_PROXY`, `https_proxy`,
`HTTP_PROXY` and `http_proxy` pointing at px's loopback address, and
`NO_PROXY` / `no_proxy` set to `localhost,127.0.0.1,::1` plus the configured
`[proxy] no_proxy`. The four extra variables above are what Node needs beyond
that:

| Variable | Why |
| --- | --- |
| `NODE_USE_ENV_PROXY=1` | Node's `fetch` reads `HTTP(S)_PROXY` only when this is set; without it a `fetch` inside `node` would go direct. |
| `NODE_USE_SYSTEM_CA=1` | a corporate px terminates TLS with the machine's own root, which Node trusts only when told to use the system store. |
| `--proxy-env npm_config_proxy` | npm reads its own variable names, and the loopback port is not known until px starts, so the name is declared and vibe fills in the address. |
| `--proxy-env npm_config_https_proxy` | the same, for HTTPS. |

`--env` is applied last, so a user who exports one of these himself is not
overridden by surprise — and a `--env` on a hand-written
`vibe tool px exec` line still wins over the proxy variables.

When `[proxy] mode = "explicit"`, vibe lends the configured proxy URL as
written and starts no px — one hop, not two. When `[proxy]` configures no
proxy at all, the run is refused, naming `--allow-direct`, because the whole
point of asking for px is that the child cannot reach the network by itself.

## Targets

| Installer option | `--platform` / `--arch` |
| --- | --- |
| `linux-x64` | `linux` / `x86_64` |
| `linux-arm64` | `linux` / `aarch64` |
| `macos-x64` | `macos` / `x86_64` |
| `macos-arm64` | `macos` / `aarch64` |
| `windows-x64` | `windows` / `x86_64` |
| `windows-arm64` | `windows` / `aarch64` |

The package is platform-independent — it contains no program — but
`platform` and `arch` are required fields of an installer option and
selection matches them exactly against the host, so a wrapper that works
everywhere still declares one row per pair vibe can detect. The six rows
differ only in their ids and names. There is no musl row: nothing is
downloaded, so a musl host is served by the `linux` rows it is detected as.

Every option declares exactly one step — a `copy` of this package's own
`payload/README.md` into the application root. An installer option must
declare at least one installation step, and for a package with no payload
this is the smallest honest one: it downloads nothing, and it leaves a note
in the payload directory `vibe app list -g` prints, saying what the
installation is.

## Closed networks

Nothing to arrange. There are no archives to mirror and no digests to place
in a drop: the whole package is its manifest, its note and its documents. The
administrator mirrors the package into the corporate registry and
`vibe install -g org.vibevm.pxworld/pxnode` works offline.

```text
vibe registry mirror vibespecs --into <corporate registry> --package org.vibevm.pxworld/pxnode
vibe install -g org.vibevm.pxworld/pxnode --offline
```

## Maintaining

`vibe.toml` is **rendered, never edited**. Its renderer is a package of its
own — `org.vibevm.pxworld/vibevm-package-generator`, one generator for the
whole px family — so this repository ships no tooling:

```text
vibe install -g org.vibevm.pxworld/vibevm-package-generator
vibevm-generate-package-pxworld pxnode --version 1.0.0 > vibe.toml
```

It fetches and reads nothing: the six targets, the wrapped commands and the
variables each one is given are tables in the generator. The rendered header
records the generator version, and re-rendering at the same version is
byte-identical — check that before every publish.

A new member of the pxworld family is a new name in that one generator: the
public command names, the wrapped command names, and whichever
`--env` / `--proxy-env` that tool needs.

## License

This package (its manifest and documents) is UPL-1.0, see
`LICENSE.md`. It contains no third-party code. Node.js, npm and npx are not
distributed here and keep their own licenses wherever the machine installed
them.
