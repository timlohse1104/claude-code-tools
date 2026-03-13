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

### Commands (`commands/`)

| Command | Description |
|---|---|
| `commit-push.md` | Full commit + push workflow with quality checks, CHANGELOG update, and branch creation |

## Usage in Claude Code

Commands are available via `/` slash shortcuts in Claude Code, e.g. `/commit-push`.
