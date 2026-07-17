# GitHub review writing

GitHub review prose is for the author, not a record of how much work the reviewer performed. GitHub already shows the review state and reviewed commit. Use the prose to explain the code consequence and what, if anything, the author needs to decide or change.

## Review body

A clean approval needs little or no body beyond the signature line. When a short explanation helps, talk about why the code is sound rather than listing verification mechanics.

Avoid:

> Approve: no blocking correctness findings. Verified at `f5c384e56b0365419bdfa5b4217db7b9415829a2`: the full three-dot diff, focused tests, and two independent reviewers. Residual uncertainty: the NAPI package was not rebuilt locally.

Prefer:

> The range validation rejects non-contiguous layouts before the relocation splice runs, and the regression tests cover the failure modes reported after #10311. This looks good overall.
>
> I left one note about the same-point zero-width case.
>
> — AI review by claude-opus-4-1

The first version reports reviewer activity. The second tells the author why the change is safe and where to look next. Omit the second paragraph when there is no inline note worth calling out.

## Inline findings

A short finding often reads best as three small parts: the observable problem, the minimum proof or context, and an optional direction. Omit any part that adds no value. Exact code calls or a small reproduction often provide enough evidence without a sentence announcing that the reviewer reproduced them.

Avoid:

> Important (contract; non-blocking): This no-op branch does not cover a zero-width move whose destination is the same point. One possible direction is to move the check; I verified the Rust path but not the NAPI package, so please validate the chosen behavior independently.

Prefer:

> `relocate(1, 1, 1)` still returns an error before this zero-width no-op runs.
>
> That makes zero-width moves depend on `to`: `relocate(1, 1, 0)` succeeds, while `relocate(1, 1, 1)` returns an error. The PR describes zero-width moves as no-ops, although the latter matches upstream `magic-string`.
>
> Adding a test for the same-point case would make the intended behavior clear. If all zero-width moves are meant to be no-ops, moving this check above the containment guard is one possible way to make that behavior consistent.

The required result is clear behavior covered by a test. The proposed code change is only one example. Declarative wording such as `would make` and `one possible way` leaves room for an implementation the reviewer did not anticipate. By contrast, `Could we …?`, `Can you …?`, `Please …`, and an apparent list of alternatives often read as instructions to choose from the reviewer's options even when meant politely.

## Blocking findings

When a finding blocks merge, be direct about the observable failure and the behavior that must hold. A possible fix should still remain an example rather than a prescribed implementation.

> This can emit the entry before its dependency when two entry groups share the same chunk, so the generated bundle throws during startup. The `shared-entry-order` fixture reproduces it on this head.
>
> Dependencies need to remain before their importers in the final order. Recording this edge before the sort is one possible fix, but another implementation is fine if it preserves that order and covers the fixture.

## Keep internal unless it matters to the author

- Exact reviewed SHA and merge base.
- Number of independent reviewers or agents.
- Commands and test inventories.
- Routine checks skipped locally when equivalent CI passed.
- Flaky or advisory CI unrelated to the changed area.
- Generic reminders that the author remains responsible for validation.

Surface one of these only when it changes confidence, affects the verdict, or gives the author a concrete action.

## Final pass

Read the draft once as a message to the author rather than as a review record. Remove a sentence when its only purpose is to demonstrate reviewer thoroughness. Split a paragraph when the observation, compatibility context, and possible implementation compete for attention. Check that the required behavior is direct, any implementation idea still leaves room for an approach the reviewer did not anticipate, and no finding starts with a severity label or disguises an instruction as a question.
