# Reviewing PR stacks

A stack is a chain of dependent PRs where each layer's base branch is the previous layer's head branch. `rolldown/rolldown` has no stacked-PR tooling configured (Graphite is used only as a merge queue), so stacks are the exception there — confirm the dependency order from the PR base branches or the author before assuming one exists.

- Scope first: review the layers the user asked for. When the stated target is a single layer inside a stack, review only that layer and use the rest of the stack as context — dependency order, and whether a later layer alters or already fixes the behavior under review — then name the unreviewed layers in the user-facing report instead of silently expanding scope.
- When the whole stack is in scope: identify the dependency order, review each layer against its own base (three-dot diff of layer vs its base), and separately review the cumulative result of the whole stack against the trunk.
- Attribute every finding to the earliest layer that introduces it, and post it on that layer's PR.
- After a restack or rebase, compare the old and new layer with `git range-diff <old-base>..<old-head> <new-base>..<new-head>` before carrying any earlier conclusion forward; re-verify rather than assume.
- The Verdict rule applies per layer: a Blocker in layer N blocks layer N and every layer stacked on it, but does not by itself block earlier layers.
- For the independent adversarial review, a stack falls under the skill's complex classification, and each subagent gets the whole stack's material with the layer boundaries stated.
