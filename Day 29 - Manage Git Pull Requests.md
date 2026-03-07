# Day 29: Manage Git Pull Requests

## Overview
This lab covers creating and merging a Pull Request (PR) in Gitea using two users: **Max** (PR creator) and **Tom** (reviewer/merger).

Max want to push some new changes to one of the repositories but we don't want people to push directly to master branch, since that
would be the final version of the code. It should always only have content that has been reviewed and approved. We cannot just allow 
everyone to directly push to the master branch. So, let's do it the right way as discussed below:

SSH into storage server using user max, password Max_pass123. There you can find an already cloned repo under Max user's home.

Max has written his story about The 🦊 Fox and Grapes 🍇

Max has already pushed his story to remote git repository hosted on Gitea branch story/fox-and-grapes


Check the contents of the cloned repository. Confirm that you can see Sarah's story and history of commits by running git log and 
validate author info, commit message etc.

Max has pushed his story, but his story is still not in the master branch. Let's create a Pull Request(PR) to merge Max's 
story/fox-and-grapes branch into the master branch.

Click on the Gitea UI button on the top bar. You should be able to access the Gitea page.

UI login info:

- Username: max
- Password: ********

PR title: Added fox and grapes story

Review and merge it.

Great stuff!! The story has been merged! 🎉
---

## Part 1: SSH into Storage Server & Inspect Repository (as Max)

### Step 1 — Login to the storage server
```bash
ssh max@ststor01
```

### Step 2 — Check available files in home directory
```bash
ls
```
> ✅ Verify that the `story-blog` repository folder is present.

### Step 3 — Navigate into the cloned repository
```bash
cd story-blog/
```
> 💡 This simulates working inside the repo, similar to a VS Code integrated terminal.

### Step 4 — Check files inside the repository
```bash
ls
```
> ✅ Files visible here depend on the current branch and git context.

### Step 5 — Check the git commit history
```bash
git log
```

### Step 6 — Check available branches
```bash
git branch
```
> ✅ Confirm you are on the branch mentioned in the scenario.

### Step 7 — Switch to the master branch
```bash
git checkout master
```

### Step 8 — List files on master branch
```bash
ls
```
> ⚠️ The file mentioned in the scenario should **not** be visible here — it only exists on the feature branch. This confirms the need for a Pull Request to merge it in.

---

## Part 2: Create a Pull Request in Gitea UI (as Max)

### Step 9 — Login to Gitea UI
- Open the Gitea web interface in a browser.
- Log in with **Max's credentials** (as provided in the scenario).

### Step 10 — Review commits & create a new Pull Request
- Navigate to the `story-blog` repository.
- Go to the **Pull Requests** tab → click **New Pull Request**.

### Step 11 — Set PR source and target branches
| Field | Value |
|-------|-------|
| **Base branch** (merge INTO) | `master` |
| **Compare branch** (merge FROM) | feature branch (as per scenario) |

> ✅ Double-check the direction: changes flow **from the feature branch → into master**.

### Step 12 — Fill in PR details
- **Title:** Use the exact PR title specified in the scenario.
- **Reviewer:** Add the reviewer as specified in the scenario (e.g., Tom).

### Step 13 — Submit the Pull Request
- Click **Create Pull Request**.

### Step 14 — Logout from Gitea as Max
- Click on your avatar/profile → **Sign Out**.

---

## Part 3: Review & Merge the Pull Request (as Tom)

### Step 15 — Login to Gitea as Tom
- Log in using **Tom's credentials** (as provided in the scenario).

### Step 16 — Navigate to the Pull Request
- Go to the `story-blog` repository.
- Click the **Pull Requests** tab.
- Open the PR created by Max.

### Step 17 — Add a comment on the PR
- In the comment box, type the **PR title** (as specified in the scenario).
- Click **Comment** to submit the comment.

### Step 18 — Merge the Pull Request
- Scroll down to the merge section.
- Click **Merge Pull Request** to complete the merge into master.

> ✅ The feature branch changes are now merged into `master`.

---

## Summary Flow

```
Max (Dev)                          Tom (Reviewer)
─────────────────────────────────────────────────────
ssh into ststor01
cd story-blog/
git log / git branch
git checkout master
  └─ notice file is missing on master

Login Gitea UI
  └─ Create Pull Request
       ├─ From: feature-branch
       ├─ To: master
       ├─ Title: <scenario title>
       └─ Reviewer: Tom
Logout Gitea
                                   Login Gitea UI
                                   Open Pull Request
                                   Add comment (PR title)
                                   Merge Pull Request ✅
```

---

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Pull Request (PR)** | A request to merge changes from one branch into another |
| **Base branch** | The target branch you want to merge INTO (usually `master`/`main`) |
| **Compare branch** | The source branch containing your new changes |
| **Reviewer** | A team member assigned to review code before merging |
| `git log` | Shows the commit history of the current branch |
| `git branch` | Lists all local branches; highlights the active one |
| `git checkout <branch>` | Switches to the specified branch |

---

> 📝 **Note:** Always verify branch contents with `ls` and `git log` before creating a PR to ensure the correct changes are on the feature branch and not yet on master.
