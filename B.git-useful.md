# 🌿 Git Branch Sync Guide

## 📌 The Problem: Why You Can't See Remote Branches Locally

When someone creates a branch **on GitHub**, your local machine **doesn't automatically know about it**.

Your local Git only knows about branches it has downloaded.

```
GitHub (Remote):   main ── Mangala ── Shermal   ✅ All 3 branches
Your Machine:      main ── Mangala               ❌ Missing Shermal
```

### Fix: Fetch all remote branches

```bash
git fetch --all       # Download info about all remote branches
git branch -r         # See all remote branches
git branch -a         # See both local + remote branches
```

To check out Shermal locally:

```bash
git checkout Shermal
# or (newer Git versions)
git switch Shermal
```

---

## 🔄 The Problem: Shermal's Changes Are Not in Your Branch

When Shermal merges his changes into **main**, your **Mangala** branch doesn't automatically update.

```
main    :  A ──► B ──► C ──► D     ← Shermal's merged changes are here
                 ↑
Mangala :  A ──► B                 ← You're stuck here, missing C & D
```

---

## ✅ How to Get Shermal's Changes

### Option 1 — Merge (Recommended: Simpler & Safer)

```bash
# Step 1: Go to main and pull the latest changes
git checkout main
git pull origin main

# Step 2: Go back to your branch and merge main into it
git checkout Mangala
git merge main
```

### Option 2 — Rebase (Cleaner History)

```bash
git checkout Mangala
git fetch origin
git rebase origin/main
```

> 💡 **No need to switch to main at all!**
> `git fetch` doesn't care which branch you're on — it updates all remote info in the background.
> Then `rebase origin/main` pulls those changes directly into your current branch.

#### Why rebase doesn't need you to go into main:

| | Merge approach | Rebase approach |
|---|---|---|
| Need to switch to main? | ✅ Yes | ❌ No |
| Steps needed | 4 steps | 2 steps (if already on Mangala) |
| Why? | You pull into **local main** first, then merge | You rebase directly onto **origin/main** (remote) |

> `origin/main` is your local copy of GitHub's main — `fetch` keeps it up to date without you ever switching branches.

---

## ⚖️ Merge vs Rebase — Which Should You Use?

| | `merge` | `rebase` |
|---|---|---|
| **History** | Keeps full history, adds a merge commit | Cleaner, linear history |
| **Safety** | ✅ Safer, easier to undo | ⚠️ Rewrites history |
| **Best for** | Shared or team branches | Personal branches (like Mangala) |
| **Complexity** | Simple to understand | Slightly more advanced |

> 💡 **Tip:** Since `Mangala` is your personal branch, either works fine.
> Use **merge** if you're unsure — it's the safest option.

---

## 🧠 Quick Summary

| Situation | Command |
|---|---|
| See all remote branches | `git branch -r` |
| See all branches (local + remote) | `git branch -a` |
| Get latest from GitHub | `git fetch --all` |
| Pull latest into main | `git checkout main` → `git pull origin main` |
| Bring main changes into your branch | `git checkout Mangala` → `git merge main` |
| Check out a remote branch locally | `git checkout Shermal` |

---

## 🔁 The Full Flow (Step-by-Step)

```bash
git fetch --all               # Get latest info from GitHub
git checkout main             # Switch to main
git pull origin main          # Pull Shermal's merged changes
git checkout Mangala          # Switch back to your branch
git merge main                # Bring those changes into your branch
```

After this, your `Mangala` branch will have **all of Shermal's changes** plus your own work. ✅
