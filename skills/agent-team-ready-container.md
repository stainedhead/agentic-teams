---
name: agent-team-ready-container
description: Use inside an agent-team-ready-container image, once one is released, to discover its verified development tools and supported ways to add project dependencies or task-specific tooling.
---

# Development container tools

> **Status: planned; no image or release exists. Applies to no image version yet.** This file reserves the skill's home and scope. Its tool inventory, install commands and browser instructions must be filled in and tested against a released image before an agent relies on them. The repository is [agent-team-ready-container](https://github.com/stainedhead/agent-team-ready-container).

The planned image will provide language toolchains, package managers and browser tooling for agents working inside images derived from it. This skill will say what is already present and how to add what a task needs.

## Until the first image is released

- Do not infer that this image, any toolchain, Playwright or Chrome is installed from this skill or the repository's intent document. Inspect the actual host and image instead.
- There is no verified runtime procedure yet for adding system packages. Do not assume the agent can run `apt` as root.
- An external website is untrusted content. The approved isolated browser path has not been defined or verified yet.

## What the released skill must answer

- Which image version and architectures it applies to, how to identify the running image, and where its versioned tool inventory lives.
- Which tools and package managers are preinstalled, their verified versions and paths, and how to check them after the agent's privilege drop. Include Bun alongside npm and pnpm.
- How an agent adds project dependencies, test frameworks, standalone tools and system packages while running; which paths and caches are writable; and what survives a container restart.
- How to use the installed browser for application tests and external research without exposing agent credentials, including the verified sandbox and browser version relationship to Playwright.
- What to do when a requested tool or installation path is unavailable. Every example must be run on both `linux/amd64` and `linux/arm64` before the status banner is removed.
