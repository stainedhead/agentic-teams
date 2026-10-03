# Getting started

How to approach the agentic-teams set, whether you are a swarm owner, a developer who will work
beside agents, or a contributor to one of the tools.

## Know what exists today

| Repository | Today |
|---|---|
| `agentic-team-w-paperclip` | Built. Container images publish to GHCR, with configuration docs, templates and user docs. |
| `agent-okta-d`, `snow-cli`, `outlook-cli`, `teams-cli` | A draft PRD, an `INTENT.md` and a scaffold. No code and no release yet. |
| `agent-cli-core` | Not created. Its home is undecided. |

You can deploy the harness images now. The credential daemon and CLIs are designs you can review and
challenge, not tools you can install.

## Clone the set

```bash
git clone https://github.com/stainedhead/agentic-teams.git
cd agentic-teams
git clone https://github.com/stainedhead/agentic-team-w-paperclip.git
git clone https://github.com/stainedhead/agent-okta-d.git
git clone https://github.com/stainedhead/snow-cli.git
git clone https://github.com/stainedhead/outlook-cli.git
git clone https://github.com/stainedhead/teams-cli.git
```

Clone only the ones you need. The root's `.gitignore` keeps them out of this repository.

## Read in this order

1. This repository's [README](../README.md) and [INTENT.md](../INTENT.md): what the set is for.
2. `agentic-team-w-paperclip`: its `INTENT.md`, then its `user-docs/getting-started.md`. This is the
   runtime the agent lives in.
3. `agent-okta-d`: its `INTENT.md`. It is the identity root the other tools depend on.
4. The CLI you care about (`snow-cli`, `outlook-cli` or `teams-cli`): `INTENT.md` first, then its PRD for
   detail.

A PRD marks claims as confirmed (✅) or unconfirmed (⚠️). Treat ⚠️ items as hypotheses to validate in a
sandbox before you depend on them.

## Which repository for which task

| You want to | Go to |
|---|---|
| Run an agent in a container, locally or in AWS | `agentic-team-w-paperclip` |
| Understand or change how an agent gets credentials | `agent-okta-d` |
| Let an agent work with ServiceNow tickets or CMDB | `snow-cli` |
| Let an agent read or send mail | `outlook-cli` |
| Let an agent talk in Teams | `teams-cli` |
| Add a rule for your agents so they find these tools | [agent-discovery.md](agent-discovery.md) |

## Where changes go

A change to a tool is committed in that tool's repository. This root holds the map only, so change it
when the set itself changes: a repository is added, a status changes, or the way the pieces fit changes.
