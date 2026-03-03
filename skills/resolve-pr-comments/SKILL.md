---
name: resolve-pr-comments
description: "Review and resolve GitHub PR review comments. Use when the user wants to address, fix, or resolve unresolved review comments on a pull request. TRIGGER when: user says \"resolve PR comments\", \"fix PR comments\", \"address review comments\", \"handle PR feedback\", or provides a PR URL with instructions about unresolved comments. Do NOT use for: creating PRs, reviewing code yourself, or general PR operations that don't involve resolving existing review comments."
---

# Resolve PR Comments

Review unresolved comments on a GitHub pull request, fix the code where comments make sense, reply with reasoning, and resolve the threads.

## Workflow

```
- [ ] Step 1: Fetch unresolved review threads
- [ ] Step 2: Understand the codebase and each comment
- [ ] Step 3: For each comment, fix code or reply with disagreement
- [ ] Step 4: Reply to each thread with reasoning
- [ ] Step 5: Resolve all threads
```

## Step 1: Fetch Unresolved Threads

Extract owner, repo, and PR number from the URL. Fetch all unresolved review threads:

```bash
gh api graphql -f query='
query {
  repository(owner: "OWNER", name: "REPO") {
    pullRequest(number: NUMBER) {
      reviewThreads(first: 50) {
        nodes {
          id
          isResolved
          comments(first: 10) {
            nodes {
              id
              body
              path
              line
              originalLine
            }
          }
        }
      }
    }
  }
}'
```

Filter to threads where `isResolved == false`.

## Step 2: Understand the Codebase and Each Comment

Before acting on comments, build sufficient understanding of the codebase:
1. Read the PR diff to understand the full scope of changes
2. Read surrounding code, related files, and existing patterns — not just the commented line
3. For each unresolved thread, read the comment body to understand what's being requested
4. Determine if the comment makes sense in the context of the codebase — does the suggestion improve correctness, consistency, or clarity?

## Step 3: Act on Each Comment

**If the comment makes sense:** Fix the code using Edit tool. Apply the suggested change or an equivalent improvement.

**If the comment does not make sense:** Prepare a reply explaining why you disagree, with specific reasoning.

## Step 4: Reply to Each Thread

Reply to each review comment with a brief explanation of what you did:

```bash
gh api repos/OWNER/REPO/pulls/NUMBER/comments -f body="REPLY" -f in_reply_to=COMMENT_ID
```

- If fixed: explain what was changed and why
- If disagreed: explain the reasoning clearly and respectfully

## Step 5: Resolve All Threads

After replying, resolve each thread:

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: { threadId: "THREAD_NODE_ID" }) {
    thread { isResolved }
  }
}'
```

## Important

- Always read the actual code before deciding whether a comment makes sense
- Do not resolve threads without replying first — always leave a record of what was done and why
