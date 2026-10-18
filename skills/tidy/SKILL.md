---
name: tidy
description: Clean up a messy git checkout safely. Inspects every uncommitted change, untracked file, stash and unpushed commit, works out whether each is already on origin, worth keeping, generated junk or unclear, and proposes keep / discard / ask for each. After one OK it backs everything up to a stash, moves keepers to a branch + PR, discards the rest and fast-forwards the checkout. Use when the user invokes /tidy, or asks to "clean up this checkout", "make this repo clean", "what should I commit or throw away", "why is main dirty".
argument-hint: "[path to the checkout, default: current folder]"
---

# /tidy: make a checkout clean without losing anything

Target = `$ARGUMENTS`, or the current folder. It must be a git checkout.

## 1. Inventory (read-only)

- Run `git fetch origin`, then `git status --porcelain=v1 -uall`, `git stash list`, and `git log --oneline origin/<default>..HEAD` (unpushed commits). Also list other branches with unpushed commits.
- Note the default branch (`git symbolic-ref refs/remotes/origin/HEAD`) and how far behind/ahead the checkout is.
- Skip anything ignored by `.gitignore`.

## 2. Classify every item

For each modified file, untracked file, stash and unpushed commit, find out:

| Check | How |
|---|---|
| Already on origin? | Compare with `origin/<default>` and recent branches: `git diff origin/<default> -- <file>`, and `git log --all -S '<distinctive line>' --oneline` to find the same change elsewhere. |
| Part of open or merged work? | `gh pr list --state all --search <file or keyword>` and branch names that match. |
| Generated or local-only? | Build output, screenshots, store assets, `.env`-like files, editor files, lockfile churn. |
| What is it? | Read the diff; say in one line what the change does. |

Put each item in one bucket:
- **Discard:** already on origin, superseded, or pure churn.
- **Keep → branch:** real work that isn't anywhere else. It goes to a new branch and PR per the repo's `AGENTS.md` naming.
- **Keep → gitignore:** local files that should never be committed (add a `.gitignore` line).
- **Keep → move:** personal assets that don't belong in the repo (say where, e.g. `~/Documents/<project>-assets/`).
- **Ask:** genuinely unclear. Say why.

## 3. Propose, then ask once

Show one table (item · what it is · evidence · bucket · action) and ask: **"OK to apply?"** The owner may change any bucket in their reply.

## 4. Apply (after the yes)

1. **Safety net first:** `git stash push -u -m "tidy backup <date>"` and then `git stash apply`, so the full state is recoverable even after cleanup. Report the stash ref.
2. For **Keep → branch** items: `git switch -c <branch>`, commit only those files with a clear message, push, open a PR, then switch back to the default branch.
3. **Keep → gitignore / move:** do it; commit `.gitignore` changes on the same branch as other keepers (or a small `chore/` branch).
4. **Discard:** `git restore` / `git clean` only the listed files, never a blanket `git clean -fdx`.
5. Fast-forward the default branch: `git merge --ff-only origin/<default>`.
6. Finish with `git status` clean (or listing only the **Ask** items), plus the PR links and the backup stash ref. Tell the owner the backup stash can be dropped (`git stash drop <ref>`) once they're happy.

Never force-push, never delete branches with unpushed commits, and never discard an **Ask** item.
