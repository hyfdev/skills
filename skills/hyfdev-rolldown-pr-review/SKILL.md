---
name: hyfdev-rolldown-pr-review
description: Reviewer-side review of rolldown/rolldown pull requests and PR stacks that ends in an evidence-backed Approve or Request changes decision. Use whenever the user asks to review, adversarially review, re-review, approve, request changes on, or decide whether to merge a Rolldown PR or stack. Do not use for implementing fixes or for addressing review comments received on the user's own PR.
---

# Rolldown PR Review

Perform a reviewer-side review that gives the maintainer enough evidence to trust the verdict. Rolldown-specific commands, CI facts, and review policies live in [references/rolldown-repo.md](references/rolldown-repo.md); read it at the start of every review — its policy and CI sections shape findings even when you run no commands.

## Safety and authorization

- Keep the reviewed branch read-only. Create repros in an isolated worktree or temporary directory and clean them up.
- Treat PR code as untrusted. Inspect changed install, build, test, and automation scripts before executing them; do not expose credentials or run commands with unneeded external side effects.
- Every GitHub mutation — submitting a review, inline comments, replies, approvals, thread resolution — follows the approval rules of the environment this skill runs in. If no such rules are in effect, show the exact intended output and get the user's confirmation first. If the user asks for a read-only review, prepare the exact GitHub output without posting.
- Follow the user's stated review scope; do not turn a correctness review into an unrelated optimization search.
- Mirror the user's language in conversation. Write GitHub output in English.

## Verdict rule

This is the single authoritative verdict rule; every other section defers to it.

- The review is complete only when the target head is stable, material checks have finished, critical behavior is verified, and the required independent adversarial review ran — or the user explicitly accepted reduced coverage after you named exactly what is missing. An existing approval from another reviewer does not substitute for any of these.
- On a complete review: any Blocker remaining means Request changes; no Blocker means Approve, even if Important or Nit findings remain.
- If the platform does not permit the intended verdict, report the verdict and the exact limitation; never substitute a different review state.

## Checklist

Track these steps internally. Use brief progress updates during a long review, but do not paste the checklist into GitHub output or the final handoff unless the user asks for it.

```
Review progress:
- [ ] 1. Pin the target (head SHA, base, merge-base, three-dot diff)
- [ ] 2. Model the problem before reading the patch
- [ ] 3. Review the implementation with evidence
- [ ] 4. Run independent adversarial review (1 subagent simple / 2 complex)
- [ ] 5. Verify findings; classify Blocker / Important / Nit
- [ ] 6. Draft inline comments
- [ ] 7. Re-pin the head and publish per the Verdict rule
- [ ] 8. Re-review author responses and new heads
```

## 1. Pin the target

- Resolve the repository, PR number, base branch, merge base, exact head SHA, and commit list. Scope the diff with paths and stats only for now (`gh pr diff <number> --name-only`); the patch body is read in step 3, after your model exists.
- For an already-merged squash PR, a live `git diff <base>...<head>` is empty or misleading because the squash commit sits on the base branch; use GitHub's recorded comparison (`gh pr diff`) or diff the squash commit against its parent.
- Read the repository instructions and contribution rules, the originating issue or specification, linked discussions, and the relevant surrounding code.
- Record existing review threads and CI state, but keep their conclusions out of the subagent prompts in step 4.
- Establish which versions define relevant external behavior — the repository-pinned dependency, the claimed upstream version, the deployed implementation — and verify claims against the version actually in scope, not only a stale pinned copy.
- Reviewing a stack (a chain of dependent PRs)? Read [references/stacked-prs.md](references/stacked-prs.md) first.

## 2. Model the problem before the patch

- Before inspecting the implementation, write down the observable success conditions, the invariants that must remain true, and the solution shape you would choose from scratch.
- Keep it proportional: a narrow fix needs a few sentences; a cross-stage semantic change needs a fuller model.
- If no authoritative contract exists, name that uncertainty instead of inventing one.
- Only then read the implementation and compare it with your model. A different design preference is not a finding; report only concrete consequences supported by evidence.

## 3. Review the implementation

Now read the full patch (`gh pr diff <number>`, or `git show` on the squash commit for a merged PR). Cover three axes explicitly:

- **Contract:** missing or partial requirements, wrong behavior, scope creep.
- **Repository standards:** documented rule violations that tooling does not already enforce (see the policy section of references/rolldown-repo.md).
- **Correctness and regression:** behavior, compatibility, state transitions, ordering, error paths, generated output, tests, interactions with surrounding code.

Use evidence proportionally:

- Run focused reproductions and tests before broad suites.
- Treat green CI as supporting evidence, not proof; separate real failures from advisory or flaky checks using the CI notes in references/rolldown-repo.md.
- Regenerate changed snapshots and generated files instead of accepting large generated diffs by inspection.
- Stop before destructive, credentialed, Docker-based, or externally mutating validation unless authorized (see Safety and authorization).

Classify provenance before posting:

- A regression introduced by the PR is in scope.
- A pre-existing problem is in scope only when the PR claims to fix it or materially worsens it; state that provenance clearly.
- Keep unrelated pre-existing problems out of the PR review.

## 4. Run independent adversarial review

Decide depth from risk and uncertainty, not line count:

- **Simple** — narrow single-subsystem change with a clear contract and direct tests: one fresh context-isolated subagent.
- **Complex** — stack, cross-subsystem change, public contract, protocol or format, concurrency, lifecycle, state machine, generated-output migration, external compatibility, or uncertain design: two fresh subagents in parallel.
- Briefly tell the user the classification and the reviewer count.

Isolation rules:

- Give each subagent only objective materials: repository and worktree paths, PR URL, exact base and head, problem source, relevant contract, repository instructions, and in-scope commands.
- Never include the primary reviewer's findings, suspected bugs, intended verdict, or other reviewers' conclusions; ask it not to read existing review comments until it has produced its own findings.
- The adversarial task is pure review: reconstruct the problem, inspect the full PR, report concrete findings with evidence. A standards-only, test-running, implementation, or comment-drafting subagent does not count as the required adversarial reviewer. Neither does your own second pass or any simulated reviewer sharing your context.

Use a prompt shaped like:

```text
Independently review <PR> at <head> against <base>. First reconstruct the intended behavior from <issue/spec> and the baseline, then inspect the full diff and relevant code. Do not read existing review comments or any primary-review analysis before forming your findings. Report only concrete correctness, compatibility, regression, or missing-test findings with file/line, impact, and evidence. Do not modify code or GitHub state.
```

Verify every subagent finding yourself — reproduce it, or establish strong static or contractual proof — before it may appear in the review. Never publish raw subagent output.

Client note: dispatch each reviewer with your client's isolation mechanism (Claude Code: the Agent tool; Codex: subagent threads). Treat client-capability claims, including this one, as testable — attempt the isolated dispatch before concluding it is unavailable. A reviewer slot that fails or is filtered before producing analysis does not count; replace it. Only after attempting isolation and failing: do not simulate independent reviewers inside your own context — apply the reduced-coverage clause of the Verdict rule and name the missing coverage.

## 5. Classify findings

- **Blocker** — must be fixed before merge; requires a runnable reproduction or strong static or contractual proof, and must not be an explicitly accepted project tradeoff.
- **Important (non-blocking)** — a real issue that may be fixed now or tracked as follow-up.
- **Nit (optional)** — low-impact documentation, naming, cleanup, or style.

Unverified suspicions never become GitHub findings; mention unverified coverage in the user-facing handoff only when it is material to the verdict or next step.

## 6. Draft inline comments

- Put each finding on the causal changed line; use a file-level comment or a short review-body item only when no meaningful changed line exists.
- Combine symptoms sharing one root cause into one comment; skip anything formatter, lint, or CI already reports; reply to an existing thread instead of opening a duplicate.
- Before drafting any GitHub prose, read [references/github-writing.md](references/github-writing.md).
- Lead with the code behavior or consequence the author needs to understand. Include only the conditions, impact, and proof needed to make that point credible; do not turn every finding into a fixed checklist of fields.
- Keep the required outcome separate from a possible implementation. A fix direction is an example that may help, not an exhaustive menu or an instruction to choose one of the reviewer's options.
- Use declarative sentences in every finding. State behavior that must hold directly, and phrase optional advice as a statement about its benefit. Do not carry either one in a question-shaped request such as `Could we …?` or `Can you …?`, or a polite command such as `Please …`; these forms often read as requirements while hiding whether the point is required or optional.
- Let the review event carry the verdict. Lead with the code behavior rather than a severity prefix such as `Important (contract; non-blocking):`. If the finding's effect on the verdict could still be misunderstood, add a complete sentence such as `This blocks merge.` or `This does not block this approval.` after the consequence; do not turn it into a label at the start of the finding.
- Use short paragraphs with one purpose each: the observation first, the minimum proof or context next, and an optional direction last. Use bullets when several independent conditions or reproductions would otherwise form a dense paragraph.
- Keep reviewer-process metadata out of author-facing prose unless it changes the verdict or tells the author what to do. Exact SHAs, merge bases, reviewer counts, command inventories, routine skipped checks, and irrelevant flaky CI belong in the internal record or user handoff.
- Mention a validation boundary only when the missing coverage could change the finding or the author's next step. Do not append ritual disclaimers that the author must validate the work independently.

## 7. Publish

- Re-pin immediately before submitting: exact head SHA, unresolved threads, CI, and pending mutating automation (rolldown's autofix.ci bot pushes commits to PR branches; see references/rolldown-repo.md).
- The GitHub review state and `commit_id` already record the verdict and reviewed head. Write the body for the author: explain why the change is safe or summarize the blocking consequence. Do not repeat `Approve`, the full SHA, or the review process as an audit log.
- A clean approval may need only one natural sentence, or no body beyond the signature line when it adds no useful context. An approval with inline notes should briefly explain why the PR is safe overall and mention a note only when it helps the author read the review. A Request changes body should summarize the blockers by consequence without duplicating every inline comment.
- Include residual uncertainty only when it could change the verdict or gives the author a concrete next step. Routine local coverage omitted because equivalent CI passed is not material by itself.
- Before publishing, make one prose-only pass: remove sentences whose only purpose is to show reviewer effort, split paragraphs that make more than one point, and confirm that every optional implementation idea still reads as optional.
- Sign everything the review publishes with the model that produced it; this signature is the standing exception to keeping reviewer-process metadata out of author-facing prose, and the prose-only pass above never removes it. End the review body with a signature line naming the exact model your client reports, such as `— AI review by claude-opus-4-1`; when an approval would otherwise have no body, the signature line alone is the body. Append the same line to any comment posted outside a review, such as a file-level comment or a thread reply, since those have no review body to carry it. Inline comments inside a review need no individual signature.
- `gh pr review` cannot attach inline comments. When authorized, submit the whole review as one call:

```bash
gh api repos/<owner>/<repo>/pulls/<number>/reviews --input review.json
```

```json
{
  "commit_id": "<reviewed head SHA>",
  "event": "REQUEST_CHANGES",
  "body": "<review summary>",
  "comments": [
    { "path": "crates/rolldown/src/x.rs", "line": 42, "side": "RIGHT", "body": "This can emit an importer before its dependency, so the generated bundle throws during startup. …" },
    { "path": "crates/rolldown/src/y.rs", "start_line": 10, "start_side": "RIGHT", "line": 14, "side": "RIGHT", "body": "…" }
  ]
}
```

- `event` is `APPROVE`, `REQUEST_CHANGES`, or `COMMENT`; `body` is required for `REQUEST_CHANGES` and `COMMENT`. The comments array is line-level only; for a true file-level comment, post it separately with `gh api repos/<owner>/<repo>/pulls/<number>/comments -f subject_type=file -f path=<path> -f commit_id=<head SHA> -f body=<text>`, or fold it into the review body.
- Read the submitted review back and confirm its state, head, body, and comment placement.

## 8. Re-review author responses and new heads

- Treat the author's response as evidence, not a debate to win; withdraw, narrow, correct, or downgrade your own finding when the evidence requires it.
- Diff the new head against the previously reviewed head (`git range-diff` plus the three-dot diff), rerun the relevant proof, inspect unresolved and outdated threads, and review the whole PR at the latest head.
- State whether the new head materially changes the reviewed behavior, and run a fresh proportional adversarial pass when it does.
- Reply to and resolve satisfied threads when authorized; do not leave stale findings open.
- Decide per the Verdict rule.

## User-facing handoff

Lead with the verdict, the concrete findings, and any GitHub action taken. Then give enough evidence for the user to trust the decision, including the from-scratch comparison when it explains why the implementation is sound. Mention independent reviewer count and unavailable coverage after the conclusion, and only when useful. Adapt the structure to the review; do not force a seven-part report, paste the internal checklist, or narrate every command.
