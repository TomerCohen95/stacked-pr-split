---
name: stacked-pr-split
description: Take a large change or PR and split it into a stack of small PRs that build on each other with GitHub `gh stack`. Plans the layers, waits for approval, builds the stack, and adds a short Before/After note on each PR for reviewers. Use when a diff is too big to review, or when the user says "split this PR", "break this into smaller PRs", "stack my PRs", or "make this reviewable". Falls back to a clean commit series when stacks are not available.
---

# Stacked PR Split

Turn one big change into a stack of small PRs. Each PR does one thing, builds on the one below it,
and explains itself: what the code looked like before, what it looks like after, and where it sits
in the whole flow.

A stack is a chain of PRs in the same repo. The bottom PR targets the trunk (usually `main`); each
PR above targets the branch below it. Reviewers see only that layer's diff.

## Ground rules

- Never push, open PRs, or rewrite branches until the user approves the split plan.
- Create a backup branch before touching history. Never lose work.
- Never force-push a branch other people use without asking first.
- Never rewrite or squash commits authored by someone else.
- Use only commands and flags documented for `gh stack`. If unsure, use plain `git` and `gh pr`.

## 1. Preflight

Run these and stop with a clear message if any fails:

```bash
gh --version                    # docs ask for a recent gh (quickstart: 2.90.0+)
git --version
gh auth status
gh extension list | grep -q gh-stack || echo "missing: gh extension install github/gh-stack"
git status --porcelain          # must be empty; ask the user to commit or stash
gh repo view --json nameWithOwner,defaultBranchRef
```

Then determine:

- **Source branch**: the branch (or PR) holding the big change. For an open PR: `gh pr view <n> --json headRefName,baseRefName,isCrossRepository`.
- **Trunk**: the PR's base, or the repo default branch.
- **Same repo**: stacks need all branches in one repository. If `isCrossRepository` is true (fork), use the fallback section.
- **Shared branch?** If others have pushed to the source branch, note it; you will leave it untouched and build the stack on new branches.

Create the backup:

```bash
git branch backup/<source-branch>-$(date +%Y%m%d-%H%M)
```

## 2. Analyze and plan the split

```bash
git fetch origin
git diff --stat origin/<trunk>...<source-branch>
git log --oneline origin/<trunk>..<source-branch>
git diff origin/<trunk>...<source-branch>
```

Read the full diff. Group changes by intent, not by file. Order layers bottom-up:

1. **Foundations**: types, schemas, migrations, interfaces, config, new dependencies.
2. **Core logic**: the behavior change itself, behind the new interfaces.
3. **Wiring / integration**: connect the logic to callers, routes, jobs, DI, feature flags.
4. **Surface**: UI, CLI, docs, extra tests, cleanup of dead code.

Rules for layers:

- One logical change per layer. If you need "and" to describe it, consider splitting it.
- Pure refactors, renames, moves, and formatting go in their own layer, usually at the bottom, so
  reviewers can skim them.
- Each layer should build and pass tests on its own when feasible. If a layer can't (for example,
  an interface with no caller yet), say so in the plan.
- Size guideline, not a rule: aim for roughly 100-400 changed lines per layer, excluding generated
  files and lockfiles. A 3-6 layer stack is typical. Don't split below the point where a layer
  makes sense alone.
- Tests ride with the code they test, unless they are a large separate suite.

## 3. Show the plan and wait

Present the plan and **stop until the user approves or edits it**:

```text
Stack plan: <short title>   trunk: main   source: feature/big-change
Backup: backup/feature-big-change-20261024-1430

  #  Branch                 Change                                   Files  ~Lines  Builds alone
  1  stack/1-rename-models  Rename Order -> PurchaseOrder (no logic)  14     +210/-210  yes
  2  stack/2-schema         Add refund columns + types                4      +120      yes
  3  stack/3-refund-logic   Refund calculation + unit tests           3      +260      yes
  4  stack/4-api            Expose POST /refunds, wire service        5      +180      yes
  5  stack/5-ui             Refund button + e2e test                  6      +240      yes

Notes: layer 2 adds unused types until layer 3 (expected).
Open as draft PRs? (default: yes)
```

Ask whether the original big PR, if any, should stay open, be closed later by the user, or be
converted to the top of the stack. Do not close it yourself.

## 4. Build the stack

Start from the trunk so each layer contains only its own change:

```bash
git switch <trunk> && git pull --ff-only
gh stack init --base <trunk> stack/1-rename-models
```

For each layer, bring in exactly its changes, then commit. Pick whichever method fits:

- **Whole files**: `git checkout <source-branch> -- path/a path/b`
- **Specific hunks**: `git checkout -p <source-branch> -- path/x` (pick hunks interactively)
- **Existing clean commits**: `git cherry-pick <sha>`

```bash
git add -A
git commit -m "<layer title>"     # use the commit template in the fallback section
# build + run tests for this layer here
gh stack add stack/2-schema       # new branch on top; must be on the top branch
```

`gh stack add -Am "<message>" <branch>` stages, commits, and creates the branch in one step when
that fits the flow.

After the last layer, verify nothing was lost or added:

```bash
git diff <source-branch> stack/<last-layer>    # must be empty (or only intended differences)
gh stack view --short
```

If the diff isn't empty, fix the right layer, then `gh stack rebase` to cascade it up. To
reorder, fold, or rename layers, use `gh stack modify` (interactive; `--abort` restores the stack).

## 5. Write each PR body

Write one body file per layer, e.g. `.git/stack-bodies/1.md` (inside `.git` so it is never
committed). Keep it plain and short. Template:

```markdown
## Where this fits
Layer **N of M** in the "<stack title>" stack.
Previous: #<prev PR or "trunk (main)"> · Next: #<next PR or "none, top of stack">

<flow diagram, current layer marked>

## Before
<1-3 sentences: how the code behaves today, relative to the layer below.>

## After
<1-3 sentences: what is different once this PR is applied. Concrete, observable.>

## Why
<The reason this step exists and why it comes at this point in the order.>

## How to review
- Start with: `path/to/main_file` (the real change)
- Then: `path/to/other`
- Skim / ignore: <renames, generated files, lockfiles, snapshots>

## Risk & tests
- Risk: <low/medium/high + what could break>
- Tests: <what was added or run; how to verify locally>
```

Flow diagram, same in every PR, with the current layer marked:

```text
main
 └─ #101  1. Rename models (no logic)
     └─ #102  2. Schema + types
         └─ #103  3. Refund logic        ◀ this PR
             └─ #104  4. API wiring
                 └─ #105  5. UI + e2e
```

A mermaid `flowchart BT` with the same nodes works too, if the team prefers it.

Writing guidance:

- Before/After describe behavior, not a list of files.
- One idea per sentence. No marketing words, no "this PR aims to".
- The top PR may add a short "Whole change" paragraph summarizing the stack end to end.

## 6. Submit

Only after the user approved pushing:

```bash
gh stack submit --auto        # pushes all branches, opens PRs as drafts, links the stack
```

- `--auto` skips the editor and uses generated titles; new PRs are created as **drafts**.
- Add `--open` only if the user wants PRs ready for review.
- Plain `gh stack submit` (interactive) lets the user write titles and bodies in an editor
  instead; draft is a toggle there, default is ready for review.

`gh stack submit` has no title/body flags, so set them after the PRs exist. PR numbers are known
now, so fill in the Previous/Next refs and diagram numbers first:

```bash
gh stack view --json                                   # branch -> PR number mapping
gh pr edit <pr> --title "<layer title>" --body-file .git/stack-bodies/<N>.md
```

Repeat for every layer. Then `gh pr view <pr> --json body` on one of them to confirm.

## 7. Report

Give the user:

- The stack, bottom to top: number, title, URL, draft or ready.
- The backup branch name.
- Any layer that doesn't build alone, and anything you weren't sure how to split.
- Next steps: `gh stack sync` after the trunk moves; `gh stack rebase` then `gh stack push` after
  editing a lower layer; `gh stack merge` (or the web UI) when approved.

## Fallback: no stacked PRs available

Use this when the repo is not on github.com (GitLab, Bitbucket, Gerrit), on GitHub Enterprise
Server without stacks, the change comes from a fork, or `gh stack` exits with code 9 (stacks not
enabled for the repo).

Same plan, same order, different packaging:

**Option A: one PR, clean commit series.** One commit per layer, rebuilt on a new branch
(`<source>-split`), never by rewriting a shared branch. Ask reviewers to review commit by commit.

```text
<layer title, imperative, <= 72 chars>

Layer N of M: <stack title>

Before: <how it behaves today>
After:  <what changes>
Why:    <reason, and why at this point in the order>

Review: start with <file>; skim <renames/generated>.
Tests:  <added/run>
```

**Option B: dependent PRs/MRs.** One branch per layer, each PR targeting the branch below it
(`gh pr create --base stack/1-... --head stack/2-... --draft --body-file ...` on GitHub; the
equivalent target-branch setting elsewhere). Use the same body template. Merge bottom-up and
retarget the next PR after each merge.

## For reviewers

If you receive a PR that's too large to review well, ask for a split instead of skimming it:
"Can you split this into a stack: refactors first, then the core change, then wiring and UI?
Each part with a short before/after." Point the author at this skill. Review bottom-up, and
approve layers as they're ready; lower layers can merge while upper ones are still in review.
