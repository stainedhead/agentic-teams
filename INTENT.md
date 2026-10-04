# Intent

## Purpose
Provide a **single, discoverable root** for the set of repositories that together let autonomous
SDLC agents work as real teammates: a containerized agent runtime, a credential daemon that gives
each agent its own short-lived, attributable identity, and a small family of agent-safe CLIs for
the enterprise systems those agents need (ServiceNow, Outlook, Teams).

This repository is documentation only. It explains what each repository is for, how they depend on
one another, and where each one lives. It contains no source code and does not vendor, submodule or
otherwise track the repositories it describes.

## The set it ties together
| Repository | What it is for |
|---|---|
| `agentic-team-w-paperclip` | Baseline container images and docs for an agent swarm: Hermes, OMP and OpenCode harnesses, plus Paperclip as the orchestration and collaboration plane. |
| `agent-team-ready-container` | Planned development toolchain base for the harness images and other agent runtimes. Its repository is a documentation scaffold; no image exists yet. |
| `agent-okta-d` | A credential daemon beside each agent host. Okta OIDC is the identity root. It serves short-lived credentials to stock tools (`aws`, `git`, `gh`) and to the custom CLIs, and the agent never reads a long-lived secret. |
| `agent-cli-core` | Shared Go library the three CLIs build from: daemon-token auth, client-side policy, output envelope with untrusted-content marking, audit log and HTTP client. It wraps the daemon's `pkg/client`. |
| `snow-cli` | `snow`: task-shaped ServiceNow CLI (work items, CMDB lookups) for agents and humans. |
| `outlook-cli` | `outlook`: read, triage and send mail as the agent's own Entra user, with inbound content marked untrusted and outbound controlled. |
| `teams-cli` | `teams`: post and read Teams messages as the agent's own Entra user, using polling and no hosted relay. |

## Goals
- **Make the set discoverable as a whole.** A person or an agent arriving here can learn what every
  repository does, how they fit together, and where to go next, without opening each one first.
- **Keep each repository independent.** Every repository has its own history, releases, issues and
  CI. This root links to them and never becomes a monorepo or a build aggregator.
- **Hold the cross-repo picture in one place.** The pieces that span repositories are the identity
  chain (Okta → daemon → tools), the shared CLI core, and how the runtime image carries the daemon
  and CLIs. Those belong here, not repeated in each repository.
- **Track where each repository stands.** Some repositories exist and some are still PRDs. The
  README shows which is which, so links and expectations stay honest.

## Non-goals
- **Source code, builds or releases.** These belong to the individual repositories.
- **Requirements for an individual component.** Each component's PRD and specs live in its own
  repository. This root only summarizes them and links to them.
- **Submodules, subtrees or vendored copies** of the sub-repositories. Developers clone the ones they
  need side by side under this directory, and `.gitignore` keeps them untracked.
- **Deploying or operating** the agent fleet.

## Scope boundary in one line
> This repository is the map of the agentic-teams project, not any part of the territory.

## How this file is used
INTENT.md records *why* this root repository exists and what it is for. Update it when the set of
repositories or the purpose of the set changes, not when a sub-repository's implementation changes.
