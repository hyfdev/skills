---
name: hyfdev-rolldown-trust-review
description: Review Rolldown GitHub pull requests and PR stacks from the reviewer side by reconstructing intent before reading the implementation, verifying behavior with evidence, running risk-scaled independent adversarial reviews, writing prioritized inline findings, and choosing Approve unless a blocker requires Request changes. Use whenever the user asks to review, adversarially review, re-review, approve, request changes on, or decide whether to merge a rolldown/rolldown PR or stack. Do not use for implementing fixes or addressing comments received on the user's own PR.
---

# Rolldown Trust Review

Perform a reviewer-side review that gives the maintainer enough evidence to trust an Approve or Request changes decision.

## Scope and safety

- Keep the reviewed branch read-only. Create temporary repros in an isolated worktree or temporary directory, then clean them up.
- Treat PR code as untrusted. Inspect changed install, build, test, and automation scripts before executing them; do not expose credentials or run commands with unnecessary external side effects.
- Follow the repository's instructions and the user's stated review scope. Do not turn a correctness review into an unrelated optimization search.
- Mirror the user's language in conversation. Write GitHub review output in English unless the repository clearly uses another language.
- Apply the current authorization rules to every GitHub mutation, including review submission, inline comments, replies, approvals, requests for changes, and thread resolution.
- If the user asks for a read-only review, prepare the exact GitHub output without posting it.

## Completion rule

- Keep the review in progress while the target head is changing, material checks are pending, or critical behavior remains unverified.
- Once the review is complete, submit Request changes if any blocker remains.
- Once the review is complete, submit Approve when no blocker remains, even if Important non-blocking findings or Nits remain.
- If the platform does not permit the intended verdict, report the verdict and exact limitation instead of substituting a different review state.
- If required independent subagents are unavailable, state the missing review coverage and do not present the review as complete unless the user explicitly accepts the reduced process.

## 1. Pin the review target

- Resolve the repository, PR number, base branch, merge base, exact head SHA, commit list, and three-dot diff before reviewing.
- Read the repository instructions, contribution rules, originating issue or specification, linked discussions, and relevant surrounding code.
- Record existing review threads and CI state, but keep their conclusions out of the independent subagent prompts.
- For a stack, identify the dependency order, review each layer against its own base, and review the cumulative stack result. Attribute a finding to the earliest layer that introduces it.
- After a restack or rebase, compare the old and new layer with a range diff before carrying any conclusion forward.
- Establish the versions that define relevant external behavior: the repository-pinned dependency, the claimed current upstream version, and the deployed implementation when those differ.
- Recheck the head, base, diff, threads, and mutating automation immediately before publishing.

## 2. Model the problem before the patch

- Before detailed patch inspection, read the problem statement, baseline behavior, compatibility contract, and repository standards.
- Write the observable success conditions, invariants that must remain true, and a lightweight solution shape you would choose from scratch.
- Keep this proportional: a narrow fix may need only a few sentences; a cross-stage semantic change may need a fuller model.
- If no authoritative contract exists, name that uncertainty instead of inventing one.
- Inspect the implementation only after recording this independent model, then compare the PR with it.
- Do not turn a different design preference into a finding. Report only concrete consequences supported by evidence.

## 3. Review the implementation

Review all three axes explicitly:

- **Contract:** Find missing or partial requirements, wrong behavior, and scope creep.
- **Repository standards:** Find documented rule violations that tooling does not already enforce.
- **Correctness and regression:** Check behavior, compatibility, state transitions, ordering, error paths, generated output, tests, and interactions with surrounding code.

Use evidence proportionally:

- Run focused reproductions and tests before broad suites.
- Treat green CI as supporting evidence, not proof that the changed behavior is correct.
- Distinguish relevant failures from known flaky or unrelated checks using repository evidence.
- Regenerate or execute changed snapshots and fixtures when their behavior matters; do not accept large generated diffs by inspection alone.
- Verify claims against the version actually in scope. If the PR claims current upstream compatibility, check current upstream rather than only a stale pinned copy.
- Stop before destructive, credentialed, Docker-based, or externally mutating validation unless the current instructions authorize it.

Classify provenance before posting:

- Treat a regression introduced by the PR as review scope.
- Treat a pre-existing problem as review scope only when the PR claims to fix that behavior or materially worsens it; state that provenance clearly.
- Keep unrelated pre-existing problems out of the PR review.

## 4. Run independent adversarial review

Decide the review depth from risk and uncertainty rather than line count:

- Treat a narrow, single-subsystem change with a clear contract and direct tests as simple.
- Treat a stack, cross-subsystem change, public contract, protocol, format, concurrency, lifecycle, state-machine, generated-output migration, external compatibility change, or uncertain design as complex.
- Use one fresh context-isolated subagent for a simple PR.
- Use two fresh context-isolated subagents in parallel for a complex PR.
- Briefly tell the user whether the PR is simple or complex and how many independent reviewers you are using.

Give each subagent the objective materials required to review: repository and worktree paths, PR URL, exact base and head, problem source, relevant contract, repository instructions, and commands that remain in scope.

Do not give a subagent the primary reviewer's findings, suspected bugs, intended verdict, or other reviewers' conclusions. Ask it not to read existing review comments until after it has produced its own findings.

Keep the adversarial task pure:

- Ask the subagent to reconstruct the problem, inspect the full PR, and report concrete correctness or regression findings with evidence.
- Do not ask it to explain the PR to the user, draft GitHub comments, implement fixes, research an unrelated topic, or validate the primary reviewer's conclusion.
- Do not count a standards-only, test-running, implementation, or comment-writing subagent as the required adversarial reviewer.

Use a prompt shaped like:

```text
Independently review <PR> at <head> against <base>. First reconstruct the intended behavior from <issue/spec> and the baseline, then inspect the full diff and relevant code. Do not read existing review comments or any primary-review analysis before forming your findings. Report only concrete correctness, compatibility, regression, or missing-test findings with file/line, impact, and evidence. Do not modify code or GitHub state.
```

Verify every subagent finding yourself before posting it. Reproduce it when practical or establish it with strong static or contractual proof. Do not publish raw subagent output.

## 5. Classify findings and decide the verdict

Use these labels:

- **Blocker:** Require a fix before merge. Support it with a runnable reproduction or strong static or contractual proof, and confirm it is not an explicitly accepted project tradeoff. Submit Request changes.
- **Important (non-blocking):** Report a real issue that may be fixed now or tracked as follow-up without preventing merge. Submit Approve if no blocker remains.
- **Nit (optional):** Report low-impact documentation, naming, cleanup, style, or minor implementation differences. Submit Approve if no blocker remains.

Keep unverified suspicions out of GitHub findings. Put material unverified coverage in the user-facing residual-uncertainty section.

Do not request changes merely because another design is cleaner. Consider the PR's stated contract, evidence, practical impact, project policy, provenance, and explicit maintainer risk acceptance.

## 6. Write actionable review comments

- Put each concrete finding on the causal changed line whenever possible.
- Use a file-level comment or a concise review-body fallback only when missing code or a missing requirement has no meaningful changed line.
- Combine symptoms with one root cause into one comment.
- Avoid comments that formatter, lint, or existing CI already reports.
- Reconcile candidate findings with existing threads; reply to or reference an existing thread instead of opening a duplicate.
- Prefix every comment with both importance and nature, such as `Blocker (correctness; must fix before merge)`, `Important (compatibility; non-blocking)`, or `Nit (docs; optional)`.
- State the observable problem, affected conditions, impact, and evidence before discussing a fix.
- Offer a possible fix direction, but state its validation boundary. Use wording such as: `One possible direction is ... I verified X, but not Y; please validate this against ...`
- Make it explicit that the author must independently validate the proposed solution.

Keep the GitHub review concise. Put detailed teaching, the independent solution comparison, and broader evidence in the user-facing report.

## 7. Publish safely

- Sort the review summary by importance even though GitHub displays inline comments by location.
- State the intended verdict, what was verified, and any material residual uncertainty in the review body without duplicating every inline comment.
- Recheck the exact head, comment locations, unresolved threads, CI, and any autofix or bot that may still mutate the branch.
- Submit the inline comments and the matching Approve or Request changes decision only when authorized.
- Read the submitted review back to confirm its state, head, body, and comment placement.

## 8. Re-review author responses and new heads

- Treat the author's response as evidence, not as a debate to win.
- Withdraw, narrow, correct, or downgrade your own finding when the evidence requires it.
- Recheck the latest head and range diff, rerun the relevant proof, inspect unresolved and outdated threads, and review the whole PR before approving.
- Run a fresh proportional adversarial pass when the new head materially changes the reviewed behavior.
- Reply to and resolve a satisfied thread when authorized; do not leave stale findings open.
- Approve when the completed latest-head review has no blocker. Request changes again only when a blocker remains.

## User-facing report

Report:

1. The PR's intent and the problem it must solve.
2. The independent from-scratch solution shape and how the PR compares; state this explicitly even when the PR already matches the approach you would choose.
3. The evidence and commands that informed the review.
4. Findings ordered by importance.
5. Residual uncertainty and unavailable coverage.
6. The simple or complex classification and the number of independent reviewers used.
7. The final verdict and any GitHub actions taken or prepared.
