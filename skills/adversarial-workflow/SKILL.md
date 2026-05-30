---
name: adversarial-workflow
description: "Generates a ready-to-run dynamic-workflow prompt in the adversarial fan-out style: split a task into independent units, do the work on each in isolation, have multiple agents refute that work, then serially build, test, and commit. Use when the user wants to run a large or repetitive coding task as a workflow with adversarial review (bug sweeps from a report, file-by-file ports or migrations, endpoint or auth audits, refactors across many files), says 'adversarial workflow' or 'workflow to fix/port/audit all X', or does not want to hand-type the long workflow prompt. Do NOT use for single-file edits, quick one-off tasks, or generating a non-workflow prompt."
metadata:
  author: yunfeihe
  version: "1.0.0"
  category: workflow
  tags: [dynamic-workflows, adversarial-review, prompt-generation, orchestration]
argument-hint: "[task description]"
---

# adversarial-workflow

Turn a one-line task into the full dynamic-workflow prompt that fans the task out across many subagents, has them adversarially review each other's work, and only touches git or the build once at the end. The point is to save the user from hand-typing a long, carefully structured prompt: they describe the task in a sentence, this produces the prompt.

The deliverable is a natural-language **workflow prompt** (the kind a workflow runtime compiles into an orchestration script), not a script. The user can paste it with the word `workflow` in their message, or ask to launch it directly.

-> See [pattern](reference/pattern.md) for the canonical example, the rationale behind each rule, and worked examples. Read it first if you are unsure how to fill the template.

## Workflow

Copy this checklist and check off each step:

```
- [ ] Step 1: Identify the unit of work and its source
- [ ] Step 2: Fill the template
- [ ] Step 3: Set the knobs (reviewers, tokens, commit, /loop)
- [ ] Step 4: Output the prompt and offer to run it
```

### Step 1: Identify the unit of work and its source

Every adversarial workflow fans out over a **source** that splits into independent **units**. Extract both from the task. Infer them when obvious; ask the user only when you genuinely cannot.

| Task phrased as... | Unit | Source |
|---|---|---|
| "fix all the bugs in REPORT.md" | one bug / finding | ./REPORT.md |
| "port these .zig files to Rust" | one .zig file | the directory of .zig files |
| "audit every endpoint for missing auth" | one endpoint / route | src/routes/ |
| "refactor X across the codebase" | one file / call site | the files that use X |

The unit must be **independent**: two units worked on in parallel must not need each other's results. If they do, this is not a fan-out — say so and stop, rather than producing a prompt that will have agents step on each other.

### Step 2: Fill the template

```
Write a workflow where for each [UNIT] in [SOURCE]:

Step 1: [Do the work on that unit]. Do not use any git or build commands to avoid stepping on another Claude running in the same branch.
Step 2: Use [N] adversarial review agents to refute the [work product]. Uncover every flaw. Do not use any git or build commands to avoid stepping on another Claude running in the same branch.

After all are complete:

Step 3: Apply all the [work products], run the build and get the relevant tests to pass. Once passing, commit and make a PR.

Start by splitting up [SOURCE] into individual [UNIT] items and pass these to the workflow.
```

Preserve these three rules when filling it in — they are what make the pattern work, not boilerplate:

- **Ban git and build commands inside the per-unit steps.** Parallel agents share one branch; a slow or stateful command from one corrupts what another sees.
- **Keep the barrier** (`After all are complete:`). Build, test, and commit happen once, serially, only after every unit is done and reviewed.
- **Step 2 refutes, it does not confirm.** The reviewers' job is to break the work and uncover every flaw, not to bless it.

### Step 3: Set the knobs

Adjust from the user's phrasing; otherwise use the default.

| Knob | Default | Change when |
|---|---|---|
| Number of adversarial reviewers | 2 | User wants more scrutiny on risky work (3-4) |
| Token budget | none | User flags the task as big or hard — add a line like "Use a ballpark of N tokens." More for harder tasks, less to save tokens |
| Final step | build, test, commit, PR | User wants to inspect first — stop after applying changes, before commit |
| /loop | off | Extra-large job — suggest pairing the workflow with /loop so it keeps running across the whole job |

### Step 4: Output the prompt and offer to run it

Output the finished prompt in a single copy-paste code block. Then offer the two ways to run it:

- Paste it back with the word `workflow` in the message, so the runtime writes and runs the orchestration script.
- Ask to launch it directly here, if running in a session that can execute workflows.

Generating the prompt is the default deliverable. Do not launch the workflow unless the user asks.
