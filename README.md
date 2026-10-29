# homebrew-claude-use

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white)](https://github.com/ExaDev/homebrew-agent-shim)

Homebrew tap for [`claude-use`](https://github.com/ExaDev/claude-use), a profile manager and launcher for Claude Code.

## Install

```bash
brew install ExaDev/claude-use/claude-use
```

or, tapped separately first:

```bash
brew tap ExaDev/claude-use
brew install claude-use
```

Both install `claude-use` alone, the same as the project's own `curl | sh` installer, `npm install -g claude-use`, and its Scoop bucket (Windows) — see [the main repo's Install section](https://github.com/ExaDev/claude-use#install) for the full comparison across all four distribution channels. None of them install a `claude` command by default; run `claude-use shim enable` afterward if you'd also like the shorter `claude @<name>` form (`claude-use shim disable` undoes it).

## What this is

`Formula/claude-use.rb` fetches the precompiled, self-contained Node SEA binary from the matching [claude-use GitHub Release](https://github.com/ExaDev/claude-use/releases) for the running platform, rather than building from source — the whole point of the SEA binary is that installing it needs no Node.js toolchain, and building from source here would defeat that. macOS arm64 and both Linux architectures are Node core's own verified single-executable-application targets.

**macOS x64 (Intel) is the one exception, and doesn't install a SEA binary at all.** Node core doesn't test or support `--build-sea` on that target, and the resulting binary genuinely crashes there (segfaults on every invocation — see claude-use's own README, "Build (Node SEA)" section, for the confirmed root cause and upstream issue links: [nodejs/node#62893](https://github.com/nodejs/node/issues/62893), [nodejs/node#59553](https://github.com/nodejs/node/issues/59553)). Since there's no fix available for the binary itself, the formula gives this one architecture a genuinely different install path: `depends_on "node"` plus `system "npm", "install", *std_npm_args"` against the npm registry tarball, Homebrew's own documented pattern for wrapping an npm package (see [Language-Specific Formulae](https://docs.brew.sh/Language-Specific-Formulae)) — the same plain-bundle code the npm channel (`npm install -g claude-use`) already publishes and verifies, not the broken SEA binary. `brew install` on Intel Macs therefore pulls in Homebrew's own `node` as a dependency and gives you a genuinely working `claude-use`.

## Keeping the formula in sync with releases

The formula's `version` and every platform's `sha256` are regenerated from scratch and pushed here automatically by the `update-tap` job in `claude-use`'s own `ci.yml`, immediately after each GitHub Release is published — never by hand. That job authenticates by checking this repository out directly with an SSH deploy key (`actions/checkout`'s `ssh-key` input), stored as the `HOMEBREW_TAP_DEPLOY_KEY` secret in the `ExaDev/claude-use` repository — the default `GITHUB_TOKEN` a workflow gets is scoped to the repo it runs in, so it cannot push here on its own. The corresponding public key is added to this repository's own Deploy keys (Settings → Deploy keys) with write access. Until both halves of that key pair exist, the `sha256` values above stay at their placeholder zeros and `brew install` will fail its checksum check, rather than silently installing something unverified.
