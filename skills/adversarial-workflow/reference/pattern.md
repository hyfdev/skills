# The adversarial fan-out workflow pattern

The dynamic-workflow pattern Jarred Sumner described as part of how Bun was rewritten in Rust in six days. Provenance (recovered thread): https://gist.github.com/hyf0-agent/c28398ee11c598366cb7675a79ec13f1

## The shape

For each unit of work:

1. **Do the work** — slow or stateful commands (git, build, cargo) are banned.
2. **Adversarial review** — independent agents try to refute the work.
3. **(After all units are done) Apply, then build, test, and commit** — once, serially.

Each step gets the same prompt with different arguments, and each runs in its own context window or fork. Moving the plan into a workflow splits the work more deterministically than ad-hoc subagents — it is closer to a bespoke build system for the task than a chat.

## Canonical example (verbatim)

The original bug-fixing prompt:

```
Write a workflow where for each finding in ./REPORT.md.

Step 1: Fix the bug. Do not use any git or build commands to avoid stepping on another Claude running in the same branch.
Step 2: Use 2 adversarial review agents to refute the bugfix. Uncover every flaw. Do not use any git or build commands to avoid stepping on another Claude running in the same branch.

After all are complete:

Step 3: Apply all the bugfixes, run the build and get the relevant tests to pass. Once passing, commit and make a PR.

Start by splitting up the ./REPORT.md into individual files with bugs and pass these files to the workflow.
```

## Blank template

```
Write a workflow where for each [UNIT] in [SOURCE]:

Step 1: [Do the work on that unit]. Do not use any git or build commands to avoid stepping on another Claude running in the same branch.
Step 2: Use [N] adversarial review agents to refute the [work product]. Uncover every flaw. Do not use any git or build commands to avoid stepping on another Claude running in the same branch.

After all are complete:

Step 3: Apply all the [work products], run the build and get the relevant tests to pass. Once passing, commit and make a PR.

Start by splitting up [SOURCE] into individual [UNIT] items and pass these to the workflow.
```

## Why each rule exists

- **Ban git/build inside the per-unit steps.** Many agents run in parallel on the same branch. A `git commit` or `cargo build` from one corrupts what another sees. Defer every stateful or slow command to the final serial step.
- **One barrier, then serial consolidation.** `After all are complete:` separates the parallel fan-out from the single pass that applies results, builds, runs tests, commits, and opens a PR. That pass is the only place global state changes, so it cannot collide with anything.
- **Adversarial, not confirmatory, review.** Reviewers are told to refute the work and uncover every flaw. A reviewer asked "is this right?" tends to rubber-stamp; one asked "prove this is wrong" finds the real problems. Use two or more independent reviewers and let the majority decide.
- **Same prompt, different arguments, isolated context.** Each unit is one focused task in its own context window, so an agent is not distracted by unrelated work, and the run aggregates far more reasoning and search than a single context could hold.
- **Token budget is a feature, not a cost to minimize.** The spend buys the split context windows and the adversarial passes, which is part of what makes the pattern reliable. Tell the workflow a ballpark to use — more for hard tasks, less for easy ones.
- **Pair with /loop for very large jobs.** Individual workflows can run for hours; /loop keeps one going across an entire large project.

## Worked examples

### Bug sweep
Task: "fix everything in REPORT.md"
- Unit: one finding. Source: ./REPORT.md.
- Step 1 does "Fix the bug"; Step 2 reviews "the bugfix"; the tail splits REPORT.md into one file per bug.

### Port / migration
Task: "rewrite the .zig files to Rust following ./PORTING"
- Unit: one .zig file. Source: the directory of .zig files.
- Step 1 does "Rewrite this .zig file to .rs following the patterns in ./PORTING and ./LIFETIMES.tsv"; Step 2 reviews "the port".
- A follow-up workflow can then be "get the crates to compile" once every file is ported.

### Audit
Task: "audit every endpoint under src/routes for missing auth"
- Unit: one route. Source: src/routes/.
- Step 1 does "Check this route for missing auth and add it"; Step 2 reviews "the auth fix".
- For a report-only audit, drop the commit from Step 3 and have the final pass collect findings into one report instead.
