# Git Rebase Notes (Feature vs Dev vs Main)

## What Is Rebase?

Rebase is a Git operation that:
- Takes the **unique commits from your current branch**
- Reapplies them on top of another branch (the "source branch")
- Produces a **linear history** without merge commits

In other words:

> Rebase removes the "branching diversion" between two branches
> by replaying your unique commits as if they came *after* the destination branch.

This is especially useful when you want:
- Clean history
- No automatic merge messages
- A linear flow of commits

---

## What Rebase *Does*

Given:

```

dev: A — B — C
feat:     D — E — F

```

If you run:

```

git checkout feat
git rebase dev

```

Git does:
1. Move `feat` pointer to the tip of `dev`: `C`
2. Replay commits D, E, F on top of `C`

So the new history becomes:

```

A — B — C — D' — E' — F'

```

Where `D'`, `E'`, `F'` are *new copies* of your commits.

🧠 The original `D`, `E`, `F` still exist in reflog but are replaced in the branch.

---

## Why This Removes Merge Commits

**Merge commits represent a branch join.**

Example merge history:

```

A — B — C — M
\    /
D — E — F (feature)

```

The “M” merge commit is created because:
- Git needed to join a branch
- There was a point of divergence

After rebase:

```

A — B — C — D' — E' — F'

```

There is no branch divergence anymore — so:
**no merge commit is necessary**.

---

## When You Can Use Rebase

Use rebase when:
✔ You are on a **feature branch**  
✔ You want to update it with the latest changes from `dev`  
✔ You haven’t merged the feature into dev yet  
✔ You want clean history without merge commits

---

## How Rebase Helps Your Workflow

### Your Typical Workflow (cleaner)

1. Create a feature branch:
```

git checkout -b feat/my-feature

```

2. Work and commit:
```

D, E, F

```

3. Update your feature branch with latest `dev`:

```

git fetch origin
git rebase origin/dev

```

History now is linear:
```

A — B — C — D' — E' — F'

```

4. Push feature branch (force if needed):
```

git push --force-with-lease

```

5. Open a pull request:
- No merge commit for feature
- Dev history stays clean

6. Merge feature into dev:
- Use "Fast-forward" or "Squash" if possible

7. Later merge dev into main
- May generate a merge commit unless fast-forward is possible

---

## What Happens On Merge After Rebase

### After rebase before feature merge

```

A — B — C — D' — E' — F'   (feat)

```

Now dev can incorporate it cleanly:

```

git checkout dev
git merge feat/my-feature  # fast-forward

```

Dev becomes:

```

A — B — C — D' — E' — F'

```

Now dev has no merge commits for this feature.

---

## What Happens When Merging Dev → Main

If main was behind dev:

```

main: A — B — C
dev:  A — B — C — D' — E' — F'

```

If you merge normally:

```

git checkout main
git merge dev

```

Git will create a **merge commit** if it cannot fast-forward.

So main may look like:

```

A — B — C — M  (merge of dev)
/   
D' — E' — F'

```

If dev is ahead and there were commits on main that aren’t on dev, a merge commit happens.

If main is strictly behind dev with no divergence, Git can fast-forward and no merge commit is needed.

---

## Why Your Feature Was Already Skipped

You saw a message:

```

warning: skipped previously applied commit <hash>

```

That happens when:
✔ The commit you are trying to reapply (from feature branch)  
✔ Already *exists* in the history of the branch you are rebasing onto

This often occurs when:
- The feature was already merged into dev before
- Or dev and feature had overlapping commits

Git sees “these changes already exist” → skips them.

So your feature branch didn’t need to replay anything.

---

## Summary: What Rebase Does

| Concept | Before Rebase | After Rebase |
|---------|---------------|--------------|
| Branch divergence | Present | Removed |
| Merge commits | Possibly present | Removed (for that feature) |
| Commit order | Interleaved | Linear |
| Feature commits | `D, E, F` | `D′, E′, F′` (new copies) |
| History readability | Complex | Cleaner |

---

## Short Workflow Recap

### Clean feature workflow

```

git checkout feat/xyz
git fetch origin
git rebase origin/dev
git push --force-with-lease

```

### Then merge into dev

✔ Minimizes merge commit noise  
✔ Feature history becomes linear with dev

---

## Rebase Key Takeaways

- Rebase moves your feature commits on top of another branch
- This produces cleaner, linear history
- It removes the need for merge commits for that feature
- Rebase only affects your branch, not dev/main
- After rebase you push with `--force-with-lease`
- Use rebase before creating PR to dev for cleaner history

---

## Visual Example (Before vs After)

**Before rebase:**
```

dev: A — B — C — M

feat:     D — E — F

```

**After rebase:**
```

dev: A — B — C — D′ — E′ — F′

```

---

## Important Notes

✔ Don’t rebase dev/main once shared publicly (dangerous)  
✔ Rebase only feature branches for collaboration  
✔ Use squash commits if you want even cleaner history  
✔ Use fast-forward merge when possible

---

## Conclusion

Rebase removes history divergence for your feature branch by replaying your unique commits on top of dev, resulting in linear history without merge commits. When merging to dev and later to main, dev becomes clean, and main may or may not generate a merge commit depending on whether it can fast-forward.

```

---
