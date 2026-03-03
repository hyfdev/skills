## How to Use

Score each item as:
- **PASS** — Meets the criterion
- **FAIL** — Does not meet the criterion (requires fix)
- **N/A** — Not applicable to this skill

---

## Frontmatter

- [ ] `---` YAML delimiters present on both sides
- [ ] `name` is kebab-case
- [ ] `name` matches the folder name exactly
- [ ] `description` exists
- [ ] `description` is under 1024 characters
- [ ] `description` is in third person ("Creates...", "Analyzes...")
- [ ] `description` follows "What + When" formula (states what it does AND when to use it)
- [ ] `description` includes key trigger terms users would actually say
- [ ] `description` contains no XML angle brackets (`<` or `>`)
- [ ] `description` contains no agent-specific terms
- [ ] `license` field is present
- [ ] `allowed-tools` is present if and only if the skill runs commands
- [ ] `metadata` is present (at minimum: author, version)
- [ ] `user-invocable: false` is set if the skill is background knowledge not meant for direct user invocation

## Body Content

- [ ] Under 500 lines (~5,000 words max)
- [ ] Does not teach general knowledge the agent already has (standard libraries, basic programming, well-known tool usage)
- [ ] No time-sensitive information (version numbers, URLs, release dates that will rot)
- [ ] Consistent terminology — one term per concept throughout
- [ ] Imperative voice ("Run the command", not "You should run the command")
- [ ] Concrete examples where they aid understanding
- [ ] No magic numbers without explanation ("voodoo constants")
- [ ] Critical instructions are near the top, not buried deep in the file
- [ ] No ambiguous language — specific instructions over vague guidance

## Structure

- [ ] Progressive disclosure applied when content is a knowledge base or only some content is needed per invocation
- [ ] Reference files are one level deep only (`reference/file.md`)
- [ ] All links from SKILL.md to reference files resolve to existing files
- [ ] No orphan reference files (every file is linked from SKILL.md)
- [ ] Headers use H3 (`###`) for categories within sections
- [ ] Workflow steps are numbered
- [ ] If the skill has a verification checklist, it appears near the top — not appended at the bottom
- [ ] No README.md inside the skill folder

## Workflow Quality

- [ ] Validation step exists (output is checked for correctness)
- [ ] Feedback loops for critical operations (report intent, get confirmation)
- [ ] Error handling is explicit (not "use best judgment")
- [ ] Dependencies between steps are clear
- [ ] Task list checklist included for workflows with 3+ steps (copyable `- [ ]` items)
- [ ] Scripts/commands solve problems directly (do not punt decision-making to the agent)

## Reference Files

- [ ] Each file is self-contained on a single topic
- [ ] Incorrect/correct pairs for examples where applicable
- [ ] Task checklists for multi-step reference content

## Triggering Quality

- [ ] Triggers on the obvious/direct request for this skill's functionality
- [ ] Triggers on paraphrased requests (different wording, same intent)
- [ ] Does NOT trigger on unrelated topics
- [ ] Negative triggers present if the skill's scope overlaps with other skills ("Do NOT use for X")
