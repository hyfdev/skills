# AGENTS.md

## Repository purpose

This repository publishes reusable agent skills. Each skill lives in `skills/<skill-name>/` and has a `SKILL.md` entry point. Supporting material belongs in that skill's `reference/` directory.

## Working conventions

- Work directly on `main`. Commit completed changes and push `main` to `origin`; do not create a feature branch or pull request unless Yunfei asks.
- Keep every repository-facing document in English and soft-wrap it: one physical line per paragraph or list item.
- Read the complete target `SKILL.md` and its directly relevant references before changing a skill.
- Keep `SKILL.md` focused on the workflow. Put detailed examples, background, and long checklists in `reference/`, and link to them from the entry point when needed.
- When adding, removing, or materially changing a published skill, update the skills list and install command in `README.md`.
- Before committing, check that every referenced local file exists, that all skill frontmatter remains valid, and that the README matches the directory names under `skills/`.
