---
name: agent-okta-d
description: Consult this when a command fails with an auth error, `reauth_required`, a 503 or 403 from the credential layer, or a "daemon unreachable" message, or when you wonder how your credentials work or whether you may touch them on a host running agent-okta-d.
---

# agent-okta-d (credential daemon beside the agent host)

> **Status: built, merged on main, tagged v0.1.0, verified against fakes only.** Everything (Okta, AWS, GitHub, ServiceNow, Graph) was tested against fakes; nothing has run against a real tenant, AWS account or credential, so vendor behavior is UNVERIFIED (`docs/assumptions.md` in the repo). Deferred per `docs/deferred.md`: real AWS SDK adapters (KMS signer, Secrets Manager store, STS client), so the `kms` signer, `aws-secretsmanager` store, the `atlassian` provider and the AWS `doctor` probe do not work (a config needing them exits 78); Keychain and TPM signers (fail-closed stubs); real-tenant M0 spikes; Okta roadmap evaluation; Entra Agent User spike; P2 items (metrics, hot reload, memory hygiene); release workflows. Only the `file` signer and `file-encrypted` store work in this build. Do not assume a daemon is running. Check what the host provides (`command -v agent-okta-d`, then `agent-okta-d version`) and report a missing tool instead of building or improvising a substitute.

## What it is and why

`agent-okta-d` is a credential daemon that runs beside you on the agent host. It holds a key you can never read, authenticates to Okta, and turns short-lived tokens into the forms each tool already understands:

- Stock tools take them through their native mechanisms: `aws` (web-identity token file), `git` (credential helper), `gh` (`GH_TOKEN`).
- The `snow`, `outlook` and `teams` CLIs obtain their tokens from the daemon themselves, through `agent-cli-core` wrapping the daemon's `pkg/client` library.
- Atlassian uses a secret handed to the Rovo MCP client (the `atlassian` provider cannot start in v0.1.0, see status).

You never handle OIDC, never read a long-lived secret and never see a signing key. The daemon runs as a different OS user than you. That separation is the main control, so protect it. Each agent is its own identity (one Okta application per agent), so everything you do is attributable to you alone.

## What you must NEVER do

- Read, or ask anyone to show you, the daemon's keys, tokens, secret store, token files' source material or config.
- Try to run as the daemon's OS user, escalate to it, or change file permissions to reach its files.
- Connect to the daemon's unix socket yourself to fetch tokens, or run `token` to read a credential for your own use outside the tools above. `token` prints the real secret (not a redacted value). Never log or paste its output.
- Copy, print, log, paste or send any credential, even to debug.
- Work around a failed credential: no borrowed tokens, no other accounts, no alternate login paths, no retry loops.
- Run operator commands: `run`, `doctor`, `revoke`, `enroll ...`, and `configure ...` are for the operator, not for you. Enrolling or re-enrolling accounts is a human step. Never restart the daemon after exit 77.

## Commands an agent may run

Client commands reach the daemon over its unix socket. Pass `--socket PATH` or set `AGENT_OKTA_D_SOCKET` (the client default is `/run/agentd/agentd.sock` on Linux and `/var/run/agentd/agentd.sock` on macOS and can differ from the daemon's per-agent socket). Do not go looking for the socket yourself: use what the operator configured.

- `agent-okta-d status [--json]`: daemon and per-provider state, expiry and last error, no secrets. Exits 0 only when everything is valid; non-zero when the daemon is not valid, revoked (77), or any provider is `degraded` or `reauth_required`. Safe to run to report facts. Implemented against fakes only.
- `agent-okta-d env github`: prints export lines (`GH_TOKEN`, or `GH_ENTERPRISE_TOKEN` plus `GH_HOST`) for stock `gh`. `env aws` also exists and takes `--config`.
- `agent-okta-d credential-helper github get|store|erase`: the git credential helper protocol. Git invokes it; you normally do not.
- `agent-okta-d token <provider> [--format raw|json] [--refresh]`: prints the real secret. Plumbing for tools and humans; run it only if your operator tells you to, and never log the output.
- `agent-okta-d configure git|gh|aws`: native tool setup, normally an operator step (`configure git --apply` runs `git config --system`).

Operator commands, not for agents: `run`, `doctor`, `revoke`, `enroll github|msgraph|okta`. A permission-denied result is an answer: report it and stop. Do not invent other commands. Provider states you may see are `degraded`, `reauth_required` and `revoked`. `version` prints the version.

### Exit codes (all client commands, including `token`)

| Code | Meaning | Do |
|---|---|---|
| 0 | ok | |
| 1 | failure (transient, provider, policy, degraded, daemon not valid) | read the message; retry only if `degraded` |
| 2 | usage error | fix flags or arguments |
| 3 | daemon unreachable | report; do not loop |
| 77 | revoked | stop; do not retry or restart the daemon |
| 78 | configuration error | report the message to the operator |

## Failure modes and what to do

| You see | Meaning | Do |
|---|---|---|
| Exit 3, "the agent-okta-d daemon is unreachable (is it running, and is this user in ipc.allow_gids?)" | Daemon not running, wrong socket path, or you are not in an allowed group | Do not retry in a loop or look for the socket. Report the message and exit code. |
| `<provider> needs re-enrollment` / `reauth_required` (HTTP 401; `github` or `msgraph`, including an expired PAT or refresh credential) | Stored user credential expired or was invalidated. `token` fails for that provider and `status` exits non-zero until a human runs `enroll`. The daemon keeps serving other providers | Stop work that needs that provider. Report the provider name. Never retry or work around it. |
| `<provider> is degraded, retry in Ns` (HTTP 503 with `Retry-After`) | Okta or network trouble; the daemon retries with backoff | Wait at least the retry interval, retry a small bounded number of times, then report. |
| Exit 77, state `revoked` (HTTP 403 code `revoked`) | The daemon was revoked (operator `revoke`, SIGUSR1) or Okta definitively rejected the client (`invalid_client`, `unauthorized_client`). It stops serving and removes credential files. A rejection for one provider's client revokes the whole daemon | Stop and report. Do not retry or restart the daemon. |
| HTTP 403 `unauthorized` | Your OS identity is not allowed on the socket (not in `ipc.allow_gids`) | Report; do not try to change permissions. |
| HTTP 404 `not_configured` | Provider unknown or disabled | Report. |
| Exit 78 | Configuration error, including a config that needs an unavailable backend (kms, aws-secretsmanager, atlassian) | Report to the operator. |
| GitHub 403/429 with `Retry-After` | Server-side rate limiting | Honor `Retry-After`; do not hammer. |

Auth errors from the target system itself (for example an AWS or ServiceNow 401/403) are also authorization or credential state. Report them. Do not try other credentials.

## Kill switch awareness

An operator can cut you off deliberately: disable the Okta app, the agent's user account, the ServiceNow user, AWS sessions and so on (PRD section 13). `agent-okta-d revoke` is the operator-side switch for the daemon itself; it does not disable anything in Okta. Tokens already issued may keep working until they expire, and then stop. If access disappears, assume revocation may be intentional. Do not try to regain it. Stop, summarize what you were doing, and tell a human.

## Links

- Repo: https://github.com/stainedhead/agent-okta-d (v0.1.0; `INTENT.md`, `user-docs/usage.md`, `user-docs/troubleshooting.md`, `docs/deferred.md`; the PRD is under `specs/archive/261003-agent-okta-d/`)
- Sibling skills in this folder: `snow-cli.md`, `outlook-cli.md`, `teams-cli.md`, `agent-cli-core.md`
