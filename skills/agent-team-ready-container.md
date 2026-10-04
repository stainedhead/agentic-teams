---
name: agent-team-ready-container
description: Use inside a released agent-team-ready-container image to discover its development tools and supported ways to add project dependencies or task-specific tooling.
---

# Development container tools

> **Status: CI-verified candidate, not released. Applies to no published image version yet.** The [first-image PR](https://github.com/stainedhead/agent-team-ready-container/pull/1) built on `linux/amd64` and `linux/arm64`; its [CI run](https://github.com/stainedhead/agent-team-ready-container/actions/runs/37215901006) passed toolchain, browser and runtime installation tests. The image is not published, and the harness and external-browser isolation gates remain open. Use this inventory only when you know you are running that candidate or a later verified release.

The image is a development base, without a harness entrypoint. Check the running environment before relying on a tool:

```bash
id -u                         # expected 1000 in this image
cat /etc/os-release            # expected Debian 13
command -v git gh aws bun uv playwright chromium
```

## Tools in the CI candidate

| Work | Preinstalled tools exercised by CI |
|---|---|
| C++23 | GCC, Clang, make, CMake, Ninja, vcpkg, common headers |
| Rust | rustup, Rust 1.99.0, Cargo |
| Go | Go 1.27.1 |
| Java | Temurin JDK 25, Maven, Gradle 9.1.0 |
| Python | Debian Python 3, venv, pip, pipx, uv |
| C# | .NET SDK 10 |
| JavaScript and TypeScript | Node 24.21.0, npm, pnpm 10.18.3, Bun 1.4.2, TypeScript 7.0.2 |
| CLI and build utilities | git, gh, AWS CLI v2, curl, wget, jq, yq, ripgrep, fd, SSH client, archive and diagnostic tools |
| Browser | Playwright 1.63.0 and its matching Chromium download; Debian `chromium` CLI |

The exact Debian package versions can change between builds. Check `--version` for the tool you use. This is a Chrome for Testing Chromium build for Playwright plus Debian Chromium; it is not a claim that branded Google Chrome is installed. A project's own Playwright dependency may need its own matching browser download.

## Add tools during a task

Install project packages in the task workspace with the project's chosen manager. The CI candidate passed these representative commands as UID 1000 on both architectures:

```bash
bun add --dev is-number@7.0.0
uv pip install --python .venv/bin/python packaging
cargo add regex@1
sudo -n apt-get update && sudo -n apt-get install -y --no-install-recommends cowsay
```

Create `.venv` first with `python3 -m venv .venv`. npm, pnpm, Go modules, Maven, Gradle, NuGet and vcpkg are also available for their respective projects. Project dependencies and user caches are writable under `/workspace` and `/home/vscode`. `sudo apt-get` changes only the disposable container; document any OS package needed again in the project's Dockerfile or setup instructions. Container root is powerful within the container, so the deployment must not mount the host Docker socket, host credentials or unrelated host paths.

## Browser use

Playwright's matching Chromium launched and loaded a local page in both CI architectures. Use it for application tests. External-site research must run in a separate browser container without the agent workspace, credential socket or cloud metadata access. The isolation and browser sandbox have not yet been verified on deployment targets; do not use the credential-bearing task container to visit untrusted sites on the strength of this skill.

If a tool is missing, identify the actual image and version, then use the appropriate package manager or report the missing prerequisite. Do not assume this candidate skill applies to a different host or an older Paperclip image.
