# pxworld wrappers for Node.js

This application has no program of its own. It installs three commands —
`pxnode`, `pxnpm` and `pxnpx` — that run the `node`, `npm` and `npx` already
on this machine's PATH through the proxy vibe manages, by re-entering vibe as

```text
vibe tool px exec --env NODE_USE_ENV_PROXY=1 --env NODE_USE_SYSTEM_CA=1 \
  --proxy-env npm_config_proxy --proxy-env npm_config_https_proxy -- node …
```

The route comes from `[proxy]` in the user's configuration or the site
profile. Nothing about Node.js is installed or configured here;
`vibe app list -g` prints this installation with the command and payload
directories, and `vibe app launchers -g org.vibevm.pxworld/pxnode` prints
what each command runs.
