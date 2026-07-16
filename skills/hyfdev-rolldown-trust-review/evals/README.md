# Evals

Scenario-based checks that this skill actually changes reviewer behavior. Run one scenario per fresh agent session with only this skill installed, paste the `query`, and grade the transcript against each `expected_behavior` item; a scenario passes when every item passes. The `setup` field tells the grader how to pick a suitable PR — record which PR was used so results are comparable.

## Scenarios

- [01-simple-pr-blocker.json](01-simple-pr-blocker.json) — simple PR with one known defect: target pinning, problem-first modeling, single-subagent isolation, Blocker leads to Request changes, no unauthorized posting.
- [02-complex-cross-subsystem.json](02-complex-cross-subsystem.json) — cross-subsystem PR with generated output: complex classification with two subagents, snapshot regeneration instead of inspection, the no-semver policy, provenance discipline.
- [03-re-review-new-head.json](03-re-review-new-head.json) — author response plus autofix.ci commit: range-diff re-review, withdrawing satisfied findings, re-pinning the head, Approve when no blocker remains.

## Run record

| Date | Client / model | What ran | Result | Notes |
| --- | --- | --- | --- | --- |
| 2026-07-16 | Claude Code / claude-fable-5 | Dry-run navigation test: steps 1–2 plus classification of the scenario-01 flow against real PR rolldown/rolldown#10313, read-only, no subagents dispatched | pass | Agent read references/rolldown-repo.md unprompted, correctly skipped references/stacked-prs.md, copied the checklist, modeled the problem before reading the patch body, classified simple with one isolated subagent prompt free of primary conclusions. It surfaced four usability gaps — patch-body exposure during step 1, the merged-squash-PR empty-diff pitfall, the missing `gh pr diff` command, and an existing-approval shortcut around the adversarial pass — all fixed in this version. |

## Unverified coverage

- Scenarios 01–03 have not been executed end to end; builds, tests, subagent dispatch, and the publish step have not been exercised under this skill version.
- Codex is untested: skill discovery, `$hyfdev-rolldown-trust-review` invocation, and behavior without an isolated-subagent mechanism have not been checked in a live Codex session (the `agents/openai.yaml` fields and invocation syntax were verified against the Codex skills documentation on 2026-07-16).
- The publish recipe (`gh api …/pulls/<number>/reviews` and the file-level comment fallback) was verified against the GitHub REST documentation on 2026-07-16, not exercised against a live PR.
