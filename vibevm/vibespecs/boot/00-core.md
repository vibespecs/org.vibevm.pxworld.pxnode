# pxworld wrappers for Node.js — application package

This package declares how Vibe installs the pxworld wrappers for Node.js as a
portable user application on six targets (Linux glibc x86_64/aarch64, macOS
x86_64/aarch64, Windows x86_64/aarch64). It ships no payload binaries at all
and downloads nothing: each installer option's single `copy` step writes this
package's own `payload/README.md` into the application root, which is the
smallest honest step an option can declare, because an option with neither
`installation` nor `[[steps]]` is refused.

It installs three public commands — `pxnode`, `pxnpm` and `pxnpx` — whose
launchers are all `executable = "command:vibe"` and re-enter vibe as
`vibe tool px exec … -- node|npm|npx`, so the tool already on PATH runs
through the px vibe manages. That spelling needs Vibe 1.0.12, which is why
`min_vibe_version` is `1.0.12`; the proxy appends the user's own arguments
after the declared ones, which is why each launcher's `args` end at `--` and
the wrapped command name.

The prefixed names are load-bearing, not cosmetic. `vibe tool px exec`
resolves the wrapped name on PATH exactly as the operating system does, with
no exclusion of vibe's own command directory, and that directory sits ahead
of the system directories — so a public command named `node` whose launcher
ran `node` would find its own proxy and re-enter itself without end. Keep the
public command names `pxnode`, `pxnpm` and `pxnpx` stable, and never claim a
public command whose name equals the command its own launcher runs.

Before changing the package, read `vibe.toml` and `README.md`. **`vibe.toml`
is rendered, never edited**, and its renderer is not in this repository: it is
`org.vibevm.pxworld/vibevm-package-generator`, one generator for the whole px
family.

```text
vibe install -g org.vibevm.pxworld/vibevm-package-generator
vibevm-generate-package-pxworld pxnode --version 1.0.0 > vibe.toml
```

Its tables for this package are the whole package: `PXNODE_TARGETS` (one row
per platform/architecture pair vibe can detect — both fields are required and
matched exactly, which is why a platform-independent wrapper still needs six
rows), `PXNODE_WRAPPERS` (public name, wrapped command, description) and the
`PXNODE_LITERAL_ENV` / `PXNODE_PROXY_ENV` lists (the variables beyond the
proxy variables `vibe tool px exec` sets itself). After a change, render,
re-render and compare, run `vibe check`, and install and run at least one
command against a real px.

A further member of the pxworld family — the same wrapper idea for `cargo`,
`curl` or another tool — is a further name in that one generator, beside
`pxnode` and `pxpip`, published as a group-mate under `org.vibevm.pxworld`.
