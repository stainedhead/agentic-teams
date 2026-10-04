# AGENTS.md

Rules and context for agents (and contributors) working in this directory.

## What this directory is
This is the **root repository for the agentic-teams set**. It is documentation only: it exists so the
independent repositories below can be discovered, understood and used as one set. It contains no
source code, no build and no tests. See [INTENT.md](INTENT.md) for why it exists.

Each sub-repository is a **separate git repository** that lives in a sub-directory of this one when
cloned. They are **not** tracked here: `.gitignore` ignores every top-level directory except an
explicit allowlist (`.github/`, `docs/`, `user-docs/`, `skills/`). Do not add submodules, subtrees or copies.

## Sub-repositories
Sub-directory names match the repository names. All seven remotes are public. Clone each one you need next to this file:
`git clone https://github.com/stainedhead/<name>.git`.

| Directory | Language / kind | Purpose | Remote | Status |
|---|---|---|---|---|
| `agentic-team-w-paperclip/` | Dockerfiles, shell, CI, docs | Baseline container images (Hermes, OMP, OpenCode CLI, optional Paperclip) and the docs and templates a swarm owner configures them with | `github.com/stainedhead/agentic-team-w-paperclip` | Exists |
| `agent-team-ready-container/` | Dockerfile, CI, docs | Development toolchain base for the harness images and other agent runtimes | `github.com/stainedhead/agent-team-ready-container` | First image and PRD merged on main; amd64/arm64 CI passed; not published |
| `agent-okta-d/` | Go | Credential daemon: Okta OIDC identity root, serves short-lived credentials to stock tools and the CLIs | `github.com/stainedhead/agent-okta-d` | Built, merged on main; v0.1.0 tagged |
| `snow-cli/` | Go | `snow`: ServiceNow CLI (work items, CMDB lookups); defines the shared CLI core | `github.com/stainedhead/snow-cli` | Built, merged on main; not released |
| `outlook-cli/` | Go | `outlook`: mail as the agent's own Entra user via Graph | `github.com/stainedhead/outlook-cli` | Built, merged on main; not released |
| `teams-cli/` | Go | `teams`: Teams messaging as the agent's own Entra user via Graph | `github.com/stainedhead/teams-cli` | Built, merged on main; not released |
| `agent-cli-core/` | Go library | Shared code `snow`, `outlook` and `teams` build from: daemon-token auth (wraps `agent-okta-d`'s `pkg/client`), policy, output envelope, audit, HTTP client | `github.com/stainedhead/agent-cli-core` | Built, merged on main; tagged to v0.2.1 |

When a repository is added or changes status, update this table and the README table in the same change.

## How they relate
```
agent-okta-d ──pkg/client──► agent-cli-core ──auth──► snow · outlook · teams   (daemon's unix socket)
agent-okta-d ──feeds creds────► aws · git · gh            (stock tools, unmodified)
snow · outlook · teams ──built from──► agent-cli-core
agent-team-ready-container ──intended base image──► agentic-team-w-paperclip images
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
  `.gitignore`, `.github/` and anything under `docs/`, `user-docs/` or `skills/`. If you add a new top-level directory that
  belongs to this repo, add it to the allowlist in `.gitignore`.
- **Before committing, check nothing from a sub-repo is staged:** `git ls-files -s` must show no mode
  `160000` entries and no sub-repository paths.
- **Do not duplicate sub-repository documentation.** Summarize in one place and link to the
  sub-repository. Give each fact one home.
- **Keep links honest.** Only link to a remote that exists. For planned repositories write the name
  and status, not a URL.
- **`user-docs/` rule.** `user-docs/` holds only files that help someone adopt, configure and use the set
  as a whole (orientation, install order, agent discovery). It is not for design, requirements or process
  material, and each tool's own usage docs live in that tool's repository. Do not restate those here.
- **Keep the discovery rule working.** The sample rule in `README.md` and `user-docs/agent-discovery.md`
  depends on the table above. When a repository is added, split or changes status, update the table, the
  README table and, if the repository mapping changed, the sample rule in the same change.
- **`skills/` rule.** `skills/` holds the agent skill documents for agent-facing tools and runtime capabilities, named
  `<repo-name>.md` (plus the shared `agent-cli-core.md`). It is the only home for them: the tool
  repositories require it (SKILL-1..7 in each PRD) and do not keep a copy. Each skill must carry an honest
  status banner until a release exists and its examples have run against it, describe only commands the
  tool really has, and name the version it applies to. Update the skill in the same change cycle as any change
  to a tool's commands, exit codes or forbidden actions. For now skills arrive here by manual pull request
  from each tool's maintainers; review them against that tool's code and `--help`.
- Never put credentials, tokens or tenant identifiers in this repository.

## Current state
Seven repositories exist: `agent-team-ready-container` (CI-tested image source, not published), `agentic-team-w-paperclip` (built) and five Go repositories (`agent-okta-d`,
`agent-cli-core`, `snow-cli`, `outlook-cli`, `teams-cli`) that are built and merged on `main` but verified only against fakes; only `agent-okta-d` (`v0.1.0`) and
`agent-cli-core` (to `v0.2.1`) are tagged.
The dependency order is `agent-okta-d` (`pkg/client`) -> `agent-cli-core` -> the three CLIs. The root
repository contains documentation only.
