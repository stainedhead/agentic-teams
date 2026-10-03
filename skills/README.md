# Skills

One place for AI agents to find and adopt instructions for using the tools in this set. Each file is a
**skill document** for one repository, named `<repo-name>.md`. The tool repositories do not keep their
own copy; they are required to keep the one here current (SKILL-1..7 in each PRD).

## Index

| Skill | Teaches an agent | Status of the tool |
|---|---|---|
| [snow-cli.md](snow-cli.md) | `snow`: read and update ServiceNow work items, look up CMDB items | Planned: PRD and scaffold, no release |
| [outlook-cli.md](outlook-cli.md) | `outlook`: read, triage and send mail as its own mailbox | Planned: PRD and scaffold, no release |
| [teams-cli.md](teams-cli.md) | `teams`: post and read Teams messages as its own user | Planned: PRD and scaffold, no release |
| [agent-cli-core.md](agent-cli-core.md) | The conventions all three CLIs share: output envelope, exit codes, untrusted content, policy, retries | Planned: library, no release |
| [agent-okta-d.md](agent-okta-d.md) | What to know, and never do, on a host running the credential daemon | Planned: PRD and scaffold, no release |

`agentic-team-w-paperclip` has no skill. It is the runtime an agent lives in, not something an agent
calls; its docs are for the people who deploy it.

## Adopting a skill

1. Install only what the host really has. Check `command -v snow outlook teams agent-okta-d`; a tool
   that is not installed is not usable, and its skill says so.
2. Give the agent the skill for each tool it will use, plus [agent-cli-core.md](agent-cli-core.md) for
   any of the three CLIs and [agent-okta-d.md](agent-okta-d.md) on any host where the daemon runs.
3. Put the files where your harness reads skills or rules (a skills folder, an `AGENTS.md` reference, a
   Hermes profile), or fetch them at start-up:

```bash
gh api repos/stainedhead/agentic-teams/contents/skills/snow-cli.md --jq .content | base64 -d
# or, without gh:
curl -fsSL https://raw.githubusercontent.com/stainedhead/agentic-teams/main/skills/snow-cli.md
```

Pin the revision your environment was tested with rather than following `main` blindly, and re-read the
skill when the tool's version changes.

## Be honest about availability

Every tool here is still a draft PRD with no code and no release. Each skill carries a banner saying so,
and describes the *planned* command surface. An agent must confirm a tool is installed and check its
version before relying on a skill, and must report a missing tool instead of building or reimplementing
it. A banner is removed only after a release exists and the skill's examples have been run against it.

## Format

Plain Markdown with `name` and `description` frontmatter. The skill format that Hermes, or the CLI
harness we provide, expects is not defined yet (unconfirmed), so the format may be adapted without
changing the content.

## Keeping them current

The source of truth for what a command does is its repository. For `snow`, `outlook` and `teams` the
skill is generated from the command tree and published in each release, then copied here. For
`agent-okta-d` and `agent-cli-core` it is written by hand. The copy here is updated by a **manual pull request**
against this repository. Opening it is part of each tool's release checklist, and a release is not
complete until it is merged. Automating it is deferred, because the release workflow's own token cannot
write to another repository (unconfirmed).
