# AGENTS.md

Rules and context for agents (and contributors) working in this directory.

## What this directory is
This is the **root repository for the agentic-teams set**. It is documentation only: it exists so the
independent repositories below can be discovered, understood and used as one set. It contains no
source code, no build and no tests. See [INTENT.md](INTENT.md) for why it exists.

Each sub-repository is a **separate git repository** that lives in a sub-directory of this one when
cloned. They are **not** tracked here: `.gitignore` ignores every top-level directory except an
explicit allowlist (`.github/`, `docs/`). Do not add submodules, subtrees or copies.

## Sub-repositories
Sub-directory names match the repository names. All five remotes are public. Clone each one you need next to this file:
`git clone https://github.com/stainedhead/<name>.git`.

| Directory | Language / kind | Purpose | Remote | Status |
|---|---|---|---|---|
| `agentic-team-w-paperclip/` | Dockerfiles, shell, CI, docs | Baseline container images (Hermes, OMP, OpenCode CLI, optional Paperclip) and the docs and templates a swarm owner configures them with | `github.com/stainedhead/agentic-team-w-paperclip` | Exists |
| `agent-okta-d/` | Go | Credential daemon: Okta OIDC identity root, serves short-lived credentials to stock tools and the CLIs | `github.com/stainedhead/agent-okta-d` | Exists: PRD and scaffold, no code yet |
| `snow-cli/` | Go | `snow`: ServiceNow CLI (work items, CMDB lookups); defines the shared CLI core | `github.com/stainedhead/snow-cli` | Exists: PRD and scaffold, no code yet |
| `outlook-cli/` | Go | `outlook`: mail as the agent's own Entra user via Graph | `github.com/stainedhead/outlook-cli` | Exists: PRD and scaffold, no code yet |
| `teams-cli/` | Go | `teams`: Teams messaging as the agent's own Entra user via Graph | `github.com/stainedhead/teams-cli` | Exists: PRD and scaffold, no code yet |
| not yet located | Go module | `agent-cli-core`: shared auth/policy/output/audit/httpx packages that `snow`, `outlook` and `teams` are built from (defined in `snow-cli-PRD.md` §5) | none yet | Planned; where it lives (own repo or inside `snow-cli`) is an open question |

When a "Planned" repository is created, update this table and the README table in the same change.

## How they relate
```
agent-okta-d ──serves tokens──► snow · outlook · teams   (via the daemon's unix socket)
agent-okta-d ──feeds creds────► aws · git · gh            (stock tools, unmodified)
snow · outlook · teams ──built from──► agent-cli-core
agentic-team-w-paperclip images ──run on the agent host──► the daemon and the CLIs
```
- The agent process and the daemon run as different OS users; the agent never sees a long-lived secret.
- Authorization is always enforced server side (IAM, ServiceNow ACLs, Exchange/Teams policy). CLI
  policy is a guardrail, not the control.
- Cross-repo design decisions are recorded in the PRDs until the owning repository exists, then in
  that repository.

## Rules for working here
- **Commit sub-repository changes in the sub-repository**, never in this root. A change to
  `snow-cli/` is committed and pushed from inside `snow-cli/`.
- **Only root-level material is committed here:** `README.md`, `INTENT.md`, `AGENTS.md`, `CLAUDE.md`,
  `.gitignore`, `.github/` and anything under `docs/`. If you add a new top-level directory that
  belongs to this repo, add it to the allowlist in `.gitignore`.
- **Before committing, check nothing from a sub-repo is staged:** `git ls-files -s` must show no mode
  `160000` entries and no sub-repository paths.
- **Do not duplicate sub-repository documentation.** Summarize in one place and link to the
  sub-repository. Give each fact one home.
- **Keep links honest.** Only link to a remote that exists. For planned repositories write the name
  and status, not a URL.
- Never put credentials, tokens or tenant identifiers in this repository.

## Current state
Five repositories exist: `agentic-team-w-paperclip` (built) and four Go repositories that hold a PRD
and a scaffold but no code yet. The shared `agent-cli-core` module is not created and its location is
undecided. The root repository contains documentation only.
