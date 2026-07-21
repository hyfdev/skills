# AGENTS.md

## Purpose

This repository is Yunfei He's public collection of reusable coding-agent skills. Treat `main` as a published catalog: every checked-in skill should be usable, documented, and ready for other people to install.

## Scope

- Target Claude Code and Codex only.
- Publish plain Agent Skills. Do not add Claude Code or Codex plugin manifests, marketplaces, plugin installation paths, or plugin release machinery.
- Do not add compatibility files, installation instructions, or support claims for other coding agents.
- Client-specific metadata is allowed when it is necessary for the same skill to behave correctly in Claude Code and Codex.

## Sources of truth

- Use the [Agent Skills specification](https://agentskills.io/specification) for the portable `SKILL.md` format.
- Use the current [Claude Code skill documentation](https://code.claude.com/docs/en/skills) and [Codex skill documentation](https://developers.openai.com/codex/skills) for client-specific behavior.
- Re-check the official documentation before adding or changing a client-specific field. Do not rely on remembered behavior.
- `AGENTS.md` is canonical for repository instructions. Keep `CLAUDE.md` as its symlink so both agents read the same rules.

## Repository structure

- Published skills live under `skills/`. A skill is the directory that directly contains its `SKILL.md`; that directory name must match the skill's `name`.
- A skill may include `scripts/` for deterministic or repeated operations, `references/` for material loaded on demand, and `assets/` for templates, images, icons, data, and other static resources used by the skill or client metadata.
- A skill may include `agents/openai.yaml` for Codex-specific metadata such as display information or invocation policy. This is skill metadata, not a plugin manifest.
- `README.md` is the public catalog and installation guide. It should let a visitor understand what each skill does, choose one, and install it for Claude Code or Codex.
- Do not add empty skill directories, placeholder skills, or unfinished skills to `main`.
- Do not add evals, staged test scenarios, or answer keys to a skill, even when a skill-authoring methodology demands them. Full scenario runs are too costly to re-run, so such files rot instead of preventing regressions. Verify a skill change with the cheapest sufficient evidence — schema checks, fact checks against the target repository's current state, a capped live probe of discovery and early behavior — record that evidence in the pull request, and treat real usage as the ongoing test: when a skill misbehaves in real work, fix it then.

## Writing skills

- Read the complete target `SKILL.md` and every directly relevant supporting file before changing a skill.
- Treat a skill's `name` as its public installation identifier. Public standalone skill names start with the stable `hyfdev-` publisher prefix, followed by a short, natural task phrase; include a product or domain only when the workflow depends on it, as in `hyfdev-rolldown-pr-review`.
- Do not put client names or subjective quality and method claims such as `trust`, `smart`, or `advanced` in a skill name. Add a role or version only when it distinguishes workflows that must coexist.
- Keep client UI metadata natural and concise: `interface.display_name` omits the publisher prefix and names the task directly.
- Treat a rename as an installation migration: update the directory, frontmatter, README, client metadata, installed copies, and lock state together. Do not leave a second alias with the same workflow.
- Keep each skill focused on one job. Split unrelated workflows instead of accumulating them in one entry point.
- Write a concise `description` that states what the skill does and when it should be used. Front-load the terms most likely to match a real request.
- Write the body as direct instructions with clear inputs, outputs, decision points, and completion criteria.
- Prefer agent-neutral language in the shared workflow. Name Claude Code or Codex only when their behavior actually differs.
- Keep `SKILL.md` concise and use progressive disclosure. Move branch-specific detail, long examples, and reference material into linked files under `references/`.
- Use scripts only when deterministic behavior, repeated parsing, or external tooling makes prose instructions insufficient. Test every added or changed script.
- Keep each rule or fact in one authoritative place. Do not duplicate the full workflow across `SKILL.md`, supporting files, and README prose.

## Claude Code and Codex compatibility

- The core workflow of every published skill must work in both Claude Code and Codex.
- When the clients express the same required behavior differently, include both configurations and keep them synchronized.
- For a skill that only the user may invoke, set `disable-model-invocation: true` in `SKILL.md` for Claude Code and set `policy.allow_implicit_invocation: false` in `agents/openai.yaml` for Codex.
- Do not depend on `user-invocable: false` for shared behavior because Codex has no documented equivalent that hides explicit invocation while preserving implicit invocation.
- When no equivalent exists, keep the portable behavior safe and useful, then state the exact client-specific limitation. Do not silently give the two clients different workflows.
- Do not treat client-specific tool permission fields as portable security boundaries. Verify enforcement in each client and use the client's own permission or deny controls when a restriction must be guaranteed.

## Documentation

- List every published skill in `README.md` with a one-line human-facing summary linked to its `SKILL.md`.
- Give every listed skill its own copyable, non-interactive command that installs it globally for both Claude Code and Codex: `npx skills@latest add hyfdev/skills --skill <skill-name> -g -a claude-code -a universal -y`.
- Update README whenever a skill is added, removed, renamed, materially changed, or installed differently.
- Show only Claude Code and Codex installation and invocation paths.
- Keep repository-facing writing in English and soft-wrap it: one physical line per paragraph or list item.

## Validation

Before committing:

- Validate every skill against the Agent Skills specification with `skills-ref validate <skill-directory>` when available or an equivalent schema check. At minimum, verify valid YAML, a matching 1-64 character lowercase alphanumeric-and-hyphen `name` with no leading, trailing, or consecutive hyphen, and a non-empty `description` of at most 1,024 characters that explains both purpose and usage.
- Check that every referenced local file exists and every relative path resolves from the skill directory.
- Check that README matches the published skills under `skills/`.
- Run every relevant script or validation command supplied by the changed skill.
- When client-specific metadata or invocation behavior changes, test discovery and invocation in current Claude Code and Codex versions when practical. If one side cannot be tested, name the exact unverified behavior.
- Check that the change adds no plugin machinery and no support surface for coding agents outside Claude Code and Codex.
- Run `git diff --check`.
