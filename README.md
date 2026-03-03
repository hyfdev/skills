# skills

A collection of skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Quick Installation

### Install All

Install all skills globally (non-interactive):

```bash
npx skills add hyf0/skills -g --all
```

### Selective Installation

For interactive installation where you can pick specific skills and agents:

```bash
npx skills add hyf0/skills -g
```

### Individual Skills

| Skill | Command | Description |
|-------|---------|-------------|
| `code` | `npx skills add hyf0/skills -g --all --skill code` | Open folders in VS Code |
| `create-skill` | `npx skills add hyf0/skills -g --all --skill create-skill` | Scaffold new agent skills with best practices |
| `optimize-skill` | `npx skills add hyf0/skills -g --all --skill optimize-skill` | Improve existing skills for conciseness and discoverability |
| `dev-announce` | `npx skills add hyf0/skills -g --all --skill dev-announce` | Generate data-driven feature announcement tweets |
| `resolve-pr-comments` | `npx skills add hyf0/skills -g --all --skill resolve-pr-comments` | Review, fix, and resolve GitHub PR review comments |

## Available Skills

### `/code`

Open the current working directory or a specified folder in Visual Studio Code.

**Usage:**
- `/code` - Opens the current working directory
- `/code /path/to/folder` - Opens the specified folder

### `/create-skill`

Creates a new agent skill following best practices for structure, description writing, and progressive disclosure. Use when you want to author a new SKILL.md, scaffold a skill directory, or build a reusable skill.

**Usage:**
- `/create-skill` - Starts the guided skill creation workflow

### `/optimize-skill`

Analyzes and optimizes an existing agent skill for conciseness, discoverability, and adherence to best practices. Use when a skill needs improvement, is too verbose, has poor activation rates, or fails to follow progressive disclosure patterns.

**Usage:**
- `/optimize-skill` - Starts the skill optimization workflow on a target skill

> Both `create-skill` and `optimize-skill` are based on real-world skill authoring experience, the [official skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices), [The Complete Guide to Building Skills for Claude (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf), and [Anthropic's skill-creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

### `/dev-announce`

Generates developer tool feature announcement tweets in an understated, data-driven style — proof-driven claims, no hype words, let the numbers speak. Outputs a main tweet, 1-3 thread replies, and actionable visual/image descriptions for each.

**Usage:**
- `/dev-announce Rolldown now supports CSS code splitting` - Generates a tweet thread for the feature
- `/dev-announce` - Prompts you for feature details before generating

### `/resolve-pr-comments`

Review and resolve GitHub PR review comments. Fetches unresolved threads, understands the codebase context, fixes code or replies with disagreement reasoning, then resolves all threads.

**Usage:**
- `/resolve-pr-comments https://github.com/owner/repo/pull/123` - Resolve comments on a specific PR

## License

MIT
