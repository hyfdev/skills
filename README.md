# skills

A collection of skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Installation

Install all skills globally (non-interactive):

```bash
npx skills add hyf0/skills -g --all
```

Interactive (pick specific skills and agents):

```bash
npx skills add hyf0/skills -g
```

## Skills

### `/code`

Open folders in VS Code.

```bash
npx skills add hyf0/skills -g --all --skill code
```

- `/code` - Opens the current working directory
- `/code /path/to/folder` - Opens the specified folder

### `/create-skill`

Scaffold new agent skills with best practices for structure, description writing, and progressive disclosure.

```bash
npx skills add hyf0/skills -g --all --skill create-skill
```

- `/create-skill` - Starts the guided skill creation workflow

### `/optimize-skill`

Improve existing skills for conciseness, discoverability, and adherence to best practices.

```bash
npx skills add hyf0/skills -g --all --skill optimize-skill
```

- `/optimize-skill` - Starts the skill optimization workflow on a target skill

> Both `create-skill` and `optimize-skill` are based on real-world skill authoring experience, the [official skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices), [The Complete Guide to Building Skills for Claude (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf), and [Anthropic's skill-creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

### `/dev-announce`

Generate data-driven feature announcement tweets — proof-driven claims, no hype words, let the numbers speak.

```bash
npx skills add hyf0/skills -g --all --skill dev-announce
```

- `/dev-announce Rolldown now supports CSS code splitting` - Generates a tweet thread for the feature
- `/dev-announce` - Prompts you for feature details before generating

### `/resolve-pr-comments`

Review, fix, and resolve GitHub PR review comments. Fetches unresolved threads, understands the codebase context, fixes code or replies with disagreement reasoning, then resolves all threads.

```bash
npx skills add hyf0/skills -g --all --skill resolve-pr-comments
```

- `/resolve-pr-comments https://github.com/owner/repo/pull/123` - Resolve comments on a specific PR

## License

MIT
