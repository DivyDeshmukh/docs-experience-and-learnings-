# Git Merge Strategies: Fast-Forward and Squash
## With Practical Flow: Feature → Dev → Main

This document explains **fast-forward merges** and **squash merges**, how they work internally, when to use them, and how they fit into a clean, industry-standard Git workflow using **feature**, **dev**, and **main** branches.

---

## 1. Fast-Forward Merge

### Definition

A **fast-forward merge** happens when the target branch has **no new commits** since the feature branch was created.  
In this case, Git does not create a merge commit. It simply moves the branch pointer forward.

---

### How Git Decides Fast-Forward

Git checks whether the target branch is an **ancestor** of the source branch.

If yes:
- No divergence
- No merge commit needed
- Pointer just moves forward

---

### Example

Initial state:

```

dev:     A — B — C
feature:           D — E

````

Command:

```bash
git checkout dev
git merge feature
````

Result (fast-forward):

```
dev: A — B — C — D — E
```

There is **no merge commit**.

---

### Why No Merge Commit Is Created

Because Git already sees a straight line of history.
There is nothing to “merge” — dev was simply behind.

---

### Characteristics of Fast-Forward

* No merge commit
* Fully linear history
* All feature commits are preserved
* Clean and readable history

---

### When Fast-Forward Is Possible

Fast-forward merge is possible only when:

* The target branch has not moved since the feature branch was created
* There is no divergence between branches

If dev has even one new commit after branching, fast-forward is not possible.

---

## 2. Squash Merge

### Definition

A **squash merge** combines **all commits from a feature branch** into **one single commit** before adding it to the target branch.

The individual commits from the feature branch do **not** appear in the target branch history.

---

### Example

Feature branch commits:

```
feature:
D: add UI
E: fix spacing
F: refactor component
```

Squash merge into dev:

```
dev:
A — B — C — S
```

Where `S` is a single commit that contains all changes from D, E, and F.

---

### How Squash Works Internally

* Git takes the **final diff** between feature and target
* Applies that diff as **one new commit**
* Original commits remain only in the feature branch

---

### Manual Squash Command

```bash
git checkout dev
git merge --squash feature
git commit -m "feat: implement generate tests UI"
```

---

### Squash via Pull Request

Most teams use:

* “Squash and merge” option in GitHub / GitLab PRs

This:

* Squashes all feature commits
* Creates exactly one commit on dev or main

---

### Characteristics of Squash Merge

* No merge commit
* Only one commit added to target branch
* Feature branch commit history is not preserved
* Very clean history

---

## 3. Fast-Forward vs Squash Comparison

| Aspect                | Fast-Forward            | Squash               |
| --------------------- | ----------------------- | -------------------- |
| Merge commit created  | No                      | No                   |
| Preserves all commits | Yes                     | No                   |
| History linear        | Yes                     | Yes                  |
| Best for              | Clean, reviewed commits | Noisy or WIP commits |
| Typical usage         | After rebase            | PR merge strategy    |

---

## 4. Recommended Workflow (Feature → Dev → Main)

### Step 1: Create Feature Branch

```bash
git checkout dev
git checkout -b feature/generate-tests-ui
```

---

### Step 2: Work and Commit on Feature

```bash
git commit -m "add UI layout"
git commit -m "fix scrolling issue"
git commit -m "final polish"
```

At this point, feature history may be noisy. That is acceptable.

---

### Step 3: Rebase Feature onto Dev (Important)

```bash
git checkout feature/generate-tests-ui
git rebase dev
```

What this does:

* Places feature commits **on top of dev**
* Removes divergence
* Enables fast-forward or clean squash later

Resulting mental model:

```
dev:     A — B — C
feature:           D — E — F
```

---

### Step 4: Push Feature (Force Push Required)

```bash
git push --force-with-lease
```

This is required because rebase rewrites commit history.

---

### Step 5: Raise PR (Feature → Dev)

At this stage you have two clean options:

#### Option A: Squash and Merge (Recommended)

* Use “Squash and merge” in PR
* Dev gets exactly **one clean commit**
* No merge commit
* No noisy history

Result:

```
dev: A — B — C — S
```

Where `S` is the squashed feature commit.

---

#### Option B: Fast-Forward Merge (If Allowed)

If your repo allows fast-forward merges and:

* Dev has not changed after rebase

Then merging will fast-forward dev:

```
dev: A — B — C — D — E — F
```

---

### Step 6: Dev → Main

For production, teams usually do one of the following:

#### Preferred: Squash or Fast-Forward

* Dev is already clean
* Main receives only meaningful commits

```
main: A — B — C — S
```

#### Acceptable: One Merge Commit

Some teams keep one merge commit from dev to main for release tracking.

---

## 5. Why This Keeps History Clean

* Rebase removes divergence
* Squash removes noise
* Fast-forward avoids merge commits
* Dev and main remain readable and linear

You avoid commit history like:

```
Merge PR #12
Merge PR #13
Merge PR #14
```

And instead get:

```
feat: generate tests UI
feat: flaky tests dashboard
chore: infra docs
```

---

## 6. Key Takeaways

* Fast-forward moves branch pointer when no divergence exists
* Squash combines many commits into one clean commit
* Rebase before merging enables both strategies
* Squash is best for feature → dev
* Dev → main should remain extremely clean

---

## 7. Simple Mental Model

* Rebase = rearrange commits
* Fast-forward = move pointer
* Squash = compress commits
* Merge commit = record divergence

Use rebase + squash to avoid merge commits in most cases.

Continuing and **closing the notes cleanly** with a few **practical decision rules, anti-patterns, and a final recommended setup** so you can directly apply this without confusion.

---

## 8. When NOT to Use Fast-Forward or Squash

### Avoid Fast-Forward When:
- You want an explicit record of a release or integration point
- Multiple features are being merged together intentionally
- The branch represents a long-lived integration branch (like `dev` in some teams)

In such cases, a **merge commit is acceptable**.

---

### Avoid Squash When:
- You want to preserve granular commit history for debugging
- Commits are already clean, meaningful, and atomic
- You are merging a long-lived branch with semantic commits

Example where squash is NOT ideal:
```

feat: add auth middleware
fix: refresh token bug
test: add auth tests

```

These commits may be worth preserving.

---

## 9. Common Anti-Patterns to Avoid

### ❌ Rebasing Shared Branches
Never rebase:
- `main`
- `dev` (if shared with others)
- Any branch others have already pulled

Rebase is safe only on:
- Local branches
- Personal feature branches

---

### ❌ Mixing Merge + Rebase Randomly
Do NOT:
- Rebase sometimes
- Merge sometimes
- Squash sometimes

Pick a **consistent strategy**.

Inconsistency leads to confusing history.

---

### ❌ Large Feature Branches Without Rebase
If a feature branch lives too long:
- Conflicts increase
- Rebasing becomes painful
- PRs become hard to review

Rebase frequently.

---

## 10. Recommended Industry-Grade Workflow (Your Case)

Given:
- You are currently the sole contributor
- You want clean history
- You have `feature → dev → main`

### Best Setup for You

#### Feature Branch
- Free to commit anything
- No need to keep it clean
- Rebase frequently onto dev

```

git checkout feature/x
git rebase dev
git push --force-with-lease

```
---

#### Feature → Dev
- Use **Squash and Merge**
- One feature = one commit on dev
- No merge commits

Dev stays linear and clean.

---

#### Dev → Main
- Use **Fast-Forward** or **Squash**
- Prefer **fast-forward** if dev is already clean
- Main history shows only release-worthy commits

---

## 11. Example Final History (Ideal)

### Main Branch
```

feat: generate tests UI
feat: flaky tests dashboard
chore: infra docs

```

### Dev Branch
```

feat: generate tests UI
feat: flaky tests dashboard
chore: infra docs

```

### Feature Branch (temporary)
```

wip: layout
fix: scroll
refactor: textarea

```

Feature branch history disappears after squash.

---

## 12. Why Merge Commits “Disappear”

Merge commits disappear because:

- Rebase removes branch divergence
- Squash collapses commits
- Fast-forward avoids merge nodes

Git only creates merge commits when **two histories diverge**.

No divergence → no merge commit.

---

## 13. Decision Table (Quick Reference)

| Situation | Recommended Action |
|---------|--------------------|
| Feature behind dev | `git rebase dev` |
| Noisy feature commits | Squash merge |
| Clean feature commits | Fast-forward |
| Dev → Main | Fast-forward or squash |
| Shared branch | Never rebase |

---

## 14. One-Line Summary

- Rebase = align
- Fast-forward = move pointer
- Squash = compress
- Merge commit = record divergence

Use **rebase + squash** for clean, professional Git history.

---

## 15. Final Advice

Your current approach is **correct and safe**.

If you:
- Rebase feature branches
- Squash into dev
- Fast-forward or squash into main

You are following **industry best practices** used in:
- Big tech
- High-scale startups
- Open-source projects

No need to over-optimize further.

---

End of notes.
```
