---
name: agent-okta-d
description: Consult this when a command fails with an auth error, `reauth_required`, a 503 or 403 from the credential layer, or a "daemon unreachable" message, or when you wonder how your credentials work or whether you may touch them on a host running agent-okta-d.
---

# agent-okta-d (credential daemon beside the agent host)

> **Status: provisional and planned.** No code and no release exist yet. This describes the PLANNED behavior in `agent-okta-d-PRD.md` (draft v0.2) and may change. The skill format itself is provisional (the Hermes/harness skill format is unconfirmed). Do not assume a daemon is running. Check what the host actually provides (for example `command -v agent-okta-d`) and report what you find. Do not improvise a substitute.

## What it is and why

`agent-okta-d` is a credential daemon that runs beside you on the agent host. It holds a key you can never read, authenticates to Okta, and turns short-lived tokens into the forms each tool already understands:

- Stock tools take them through their native mechanisms: `aws` (web-identity token file), `git` (credential helper), `gh` (`GH_TOKEN`).
- The `snow`, `outlook` and `teams` CLIs obtain their tokens from the daemon themselves.
- Atlassian uses a secret handed to the Rovo MCP client.

You never handle OIDC, never read a long-lived secret and never see a signing key. The daemon runs as a different OS user than you. That separation is the main control, so protect it. Each agent is its own identity (one Okta application per agent), so everything you do is attributable to you alone.

## What you must NEVER do

- Read, or ask anyone to show you, the daemon's keys, tokens, secret store, token files' source material or config.
- Try to run as the daemon's OS user, escalate to it, or change file permissions to reach its files.
- Connect to the daemon's unix socket yourself to fetch tokens. The PRD's agent-facing path is the tools above. Section 11 lists `token`, `env` and `credential-helper` as commands, but it does not say agents may run them directly. Treat them as plumbing for tools and humans unless your operator tells you otherwise.
- Copy, print, log, paste or send any credential, even to debug.
- Work around a failed credential: no borrowed tokens, no other accounts, no alternate login paths, no retry loops.
- Enroll or re-enroll accounts (`enroll ...`). That is a human step.

## Commands an agent may run

The PRD (section 11) does not explicitly define which commands agents may run. The only ones that are read-only and carry no secrets are:

- `agent-okta-d status`: per-provider state, expiry and last error, with no secrets. Unverified that it is meant for agent use. It is reasonable to run it to report facts.
- `agent-okta-d doctor`: end-to-end self-test (Okta reachable, key usable, clock skew, each enabled provider). Same caveat.

Run these only if they exist on the host and your operator allows it. A permission-denied result is an answer: report it and stop. Do not invent other commands. Provider states you may see are `degraded`, `reauth_required` and `revoked` (see below).

## Failure modes and what to do

| You see | Meaning (per PRD) | Do |
|---|---|---|
| CLI says the daemon is unreachable (the CLIs exit with code 3 and a clear message naming the socket they tried; this is an approved decision in the core PRD) | Daemon not running, socket missing, or you are not allowed on it | Do not retry in a loop or look for the socket. Report the message and the exit code. |
| `reauth_required` for `github` or `msgraph` (also an expired GitHub token) | Stored user credential is expired, rejected or consented away. The daemon keeps serving other providers and waits for a human to run `enroll` | Stop work that needs that provider. Report it with the provider name. Never retry. |
| HTTP 503 with `Retry-After` | State `degraded`: Okta or network trouble, refresh failing transiently (FR-5) | Wait at least the `Retry-After` interval and retry a small bounded number of times. If it persists, report. |
| 403 from the daemon, or state `revoked` | The agent has been disabled. The daemon stops serving, deletes its sinks and exits 77 | Stop and report. Do not retry. |
| GitHub 403/429 with `Retry-After` | Server-side rate limiting (GH-9) | Honor `Retry-After`; do not hammer. |

Auth errors from the target system itself (for example an AWS or ServiceNow 401/403) are also authorization or credential state. Report them. Do not try other credentials.

## Kill switch awareness

An operator can cut you off deliberately: disable the Okta app, the agent's user account, the ServiceNow user, AWS sessions and so on (PRD section 13). Tokens already issued may keep working until they expire, and then stop. If access disappears, assume revocation may be intentional. Do not try to regain it. Stop, summarize what you were doing, and tell a human.

## Links

- Repo: https://github.com/stainedhead/agent-okta-d (`INTENT.md`, `agent-okta-d-PRD.md`, `user-docs/`)
- Sibling skills in this folder: `snow-cli.md`, `outlook-cli.md`, `teams-cli.md`, `agent-cli-core.md`
