# AGENTS.md

## Repository purpose

This repository publishes reusable Agent Skills for both Claude Code and Codex. Each skill lives in `skills/<skill-name>/` with a `SKILL.md` entry point and optional `scripts/`, `references/`, `assets/`, and client-specific metadata.

## Best practices

### Start with a portable core

- Use the open [Agent Skills specification](https://agentskills.io/specification) as the shared baseline. Keep `name` and `description` valid under the specification, make `name` match the skill directory, and describe both what the skill does and when it applies.
- Keep the workflow in `SKILL.md` useful in both Claude Code and Codex. Prefer agent-neutral language, capabilities, inputs, and outputs over client-specific command names or tool names.
- Keep each skill focused on one job. Put detailed background, examples, and long checklists in `references/`, and load them only when the workflow needs them.
- Treat client-specific fields and files as adapters around the shared workflow, not as the only place where essential behavior is defined. If one client ignores an extension, the skill should still work or clearly state the remaining limitation.
- Treat `allowed-tools` as client-dependent permission metadata, not a portable tool restriction. In Claude Code it pre-approves the listed tools but does not prevent other tools from being used under normal permission rules; Codex does not document support for it, so assume Codex may ignore it and use each client's permission or deny policy for restrictions.

### Support client extensions in pairs

- Check the current [Claude Code skill documentation](https://code.claude.com/docs/en/skills) and [Codex skill documentation](https://developers.openai.com/codex/skills) before adding client-specific behavior.
- When both clients offer the same behavior through different configuration, include both forms. Claude Code extensions usually live in `SKILL.md` frontmatter; Codex-specific invocation policy, UI metadata, and dependencies live in `agents/openai.yaml`.
- When one client has no equivalent feature, do not describe that behavior as portable. Prefer the shared default, accept a documented degraded behavior, or split the client-specific packaging when the difference is essential.
- When documentation needs explicit invocation syntax, show both forms: `/skill-name` for Claude Code and `$skill-name` for Codex.

### Invocation control example

| Intent | Claude Code | Codex | Shared-skill guidance |
| --- | --- | --- | --- |
| User and model may invoke | Default; users type `/skill-name` | Default; users mention `$skill-name` | Prefer this portable default. |
| Only the user should invoke | Set `disable-model-invocation: true` in `SKILL.md` | Set `policy.allow_implicit_invocation: false` in `agents/openai.yaml` | Include both settings when explicit-only behavior is required. |
| Only the model should invoke | Set `user-invocable: false` in `SKILL.md`; this hides the skill from the `/` menu | No documented equivalent for hiding explicit invocation while retaining implicit invocation | Do not rely on this behavior in a shared skill. Make explicit invocation harmless or provide client-specific packaging. |

For an explicit-only shared skill, pair the Claude Code frontmatter with the Codex metadata:

```yaml
# SKILL.md
---
name: deploy
description: Deploys the application after required checks. Use only when the user explicitly requests a deployment.
disable-model-invocation: true
---
```

```yaml
# agents/openai.yaml
policy:
  allow_implicit_invocation: false
```

## Working conventions

- Work directly on `main`. Commit completed changes and push `main` to `origin`; do not create a feature branch or pull request unless Yunfei asks.
- Keep every repository-facing document in English and soft-wrap it: one physical line per paragraph or list item.
- Read the complete target `SKILL.md` and its directly relevant references before changing a skill.
- Write skills for both Claude Code and Codex. Prefer agent-neutral language and shared operations; when an instruction must differ, identify the agent and provide an equivalent path for the other one.
- When adding, removing, or materially changing a published skill, update the relevant `README.md` documentation. Keep a skills list or install command in sync when the README includes one.

## Validation

- Check that every referenced local file exists and that the README matches the directory names under `skills/`.
- Validate YAML frontmatter and the portable Agent Skills fields.
- When client-specific behavior changes, test it in current Claude Code and Codex versions when practical. If one side cannot be tested, name the exact unverified behavior.
- Confirm that client-specific configuration has an equivalent for the other client where one exists, and that unsupported differences are documented instead of implied to be portable.
