# claude-code-tools

Reusable Claude Code project tools (commands, agents, skills) for use as a Git submodule.

## Setup

Clone into your project's `.claude/` directory as a submodule:

```bash
git submodule add git@github.com:timlohse1104/claude-code-tools.git .claude
git submodule update --init
```

After cloning a project that uses this submodule:

```bash
git clone --recurse-submodules <project-repo>
# or if already cloned:
git submodule update --init
```

## Local Permissions

Copy the example permissions file and customize it:

```bash
cp .claude/settings.local.json.example .claude/settings.local.json
```

`settings.local.json` is gitignored in this repo and will not be tracked.

## Contents

### Skills (`skills/`)

| Skill | Trigger | Description |
|---|---|---|
| `commit-push/` | `/commit-push` | Full commit + push workflow with quality checks, CHANGELOG update, and branch creation |
| `finish-feature/` | `/finish-feature` | Closes a merged feature branch: switches to main, updates it, deletes the local branch |
| `get-joke/` | `/get-joke` | Präsentiert 3-5 tagesaktuelle Witze fürs Daily Standup (Flachwitze, Wortwitze, IT-Witze) |

## Usage in Claude Code

Skills are available via `/` slash shortcuts in Claude Code, e.g. `/commit-push`.

Skills with `disable-model-invocation: true` must be called explicitly. Others can also be triggered automatically by Claude based on context.
