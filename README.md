# homebrew-agent-shim

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white)](https://github.com/ExaDev/homebrew-agent-shim)

Homebrew tap for [`agent-shim`](https://github.com/ExaDev/agent-shim), a profile manager and launcher for Claude Code. The project was called `claude-use` until it was renamed.

## Install

```bash
brew install ExaDev/agent-shim/agent-shim
```

or, tapped separately first:

```bash
brew tap ExaDev/agent-shim
brew install agent-shim
```

Homebrew maps `ExaDev/agent-shim` to this repository because the one-argument form of `brew tap` expects a GitHub repository named `homebrew-<repository>` ([Taps](https://docs.brew.sh/Taps), [archived](https://web.archive.org/web/20261017041211/https://docs.brew.sh/Taps)). The formula is `Formula/agent-shim.rb`.

Both install `agent-shim` alone, the same as the project's own `curl | sh` installer, `npm install -g agent-shim` and its Scoop bucket (Windows). See [the main repository's installation guide](https://github.com/ExaDev/agent-shim/blob/main/docs/installation.md) for every channel. None of them install a `claude` command by default; run `agent-shim shim enable` afterwards if you would also like the shorter `claude @<name>` form (`agent-shim shim disable` undoes it).

A `claude-use` command is installed beside `agent-shim`, so scripts written against the old name keep working.

## Upgrading

```bash
brew upgrade agent-shim
```

`agent-shim update` refuses to replace a Homebrew-managed installation and names this command instead. An installed `claude-use` formula moves to `agent-shim` on `brew upgrade`, because `formula_renames.json` maps the old name to the new one.

## What this is

`Formula/agent-shim.rb` fetches the precompiled, self-contained Node SEA binary from the matching [agent-shim GitHub Release](https://github.com/ExaDev/agent-shim/releases) for the running platform, rather than building from source. The point of the SEA binary is that installing it needs no Node.js toolchain, and building from source here would defeat that. macOS arm64 and both Linux architectures are Node core's own verified single-executable-application targets.

**macOS x64 (Intel) is the one exception, and does not install a SEA binary at all.** Node core does not test or support `--build-sea` on that target, and the resulting binary crashes there on every invocation (see the main repository's [release process](https://github.com/ExaDev/agent-shim/blob/main/docs/release-process.md#build-node-sea) for the root cause and upstream issue links: [nodejs/node#62893](https://github.com/nodejs/node/issues/62893), [nodejs/node#59553](https://github.com/nodejs/node/issues/59553)). Since the binary itself cannot be fixed, the formula gives this one architecture a different install path: `depends_on "node"` plus `system "npm", "install", *std_npm_args` against the npm registry tarball, Homebrew's documented pattern for wrapping an npm package (see [Language-Specific Formulae](https://docs.brew.sh/Language-Specific-Formulae)). It is the same plain bundle the npm channel publishes and verifies, not the broken SEA binary, so `brew install` on Intel Macs pulls in Homebrew's own `node` as a dependency and gives you a working `agent-shim`.

## Keeping the formula in sync with releases

The formula's `version` and every platform's `sha256` are regenerated from scratch and pushed here automatically by the `update-tap` job in `agent-shim`'s own [`ci.yml`](https://github.com/ExaDev/agent-shim/blob/main/.github/workflows/ci.yml), immediately after each GitHub Release is published, never by hand. The same workflow then installs the tapped formula on macOS (arm64 and x64) and Linux to verify it. That job authenticates by checking this repository out directly with an SSH deploy key (`actions/checkout`'s `ssh-key` input), stored as the `HOMEBREW_TAP_DEPLOY_KEY` secret in the `ExaDev/agent-shim` repository. The default `GITHUB_TOKEN` a workflow gets is scoped to the repository it runs in, so it cannot push here on its own. The corresponding public key is added to this repository's own Deploy keys (Settings, Deploy keys) with write access.
