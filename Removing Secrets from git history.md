# Removing Secrets from Git History (.env Cleanup Guide)

<br>
<br>

- [Removing Secrets from Git History (.env Cleanup Guide)](#removing-secrets-from-git-history-env-cleanup-guide)
  - [Important Rule (Read First)](#important-rule-read-first)
  - [What We Are Trying to Do](#what-we-are-trying-to-do)
  - [Prerequisites](#prerequisites)
    - [General](#general)
    - [Check Git version](#check-git-version)
  - [Common Mistakes (What NOT to Do)](#common-mistakes-what-not-to-do)
  - [OPTION 1: GitHub Codespaces (BEST \& SAFEST)](#option-1-github-codespaces-best--safest)
    - [Step 1: Open Codespaces](#step-1-open-codespaces)
    - [Step 2: Install git-filter-repo](#step-2-install-git-filter-repo)
    - [Step 3: Add it to PATH](#step-3-add-it-to-path)
    - [Step 4: Run history cleanup](#step-4-run-history-cleanup)
    - [Step 5: Re-add GitHub remote (IMPORTANT)](#step-5-re-add-github-remote-important)
    - [Step 6: Force push clean history](#step-6-force-push-clean-history)
    - [Step 7: Verify cleanup](#step-7-verify-cleanup)
  - [OPTION 2: Linux / macOS (Bash)](#option-2-linux--macos-bash)
    - [Install tool](#install-tool)
    - [Run cleanup](#run-cleanup)
  - [OPTION 3: Windows (Local Machine)](#option-3-windows-local-machine)
    - [Problem 1: pip not recognized](#problem-1-pip-not-recognized)
    - [Problem 2: git filter-repo not found](#problem-2-git-filter-repo-not-found)
  - [After Cleanup (MANDATORY SECURITY STEPS)](#after-cleanup-mandatory-security-steps)
  - [Prevent This in Future](#prevent-this-in-future)
  - [Final Notes](#final-notes)


<br>
<br>

This document explains how to **completely remove a leaked `.env` file from Git history**, not just delete it in a new commit.

<br>

**This situation happens when sensitive data like:**
* MongoDB URI
* Secret keys
* Passwords

are accidentally pushed to a GitHub repository.

Deleting the file normally is **NOT enough**.

Git history must be rewritten.

<br>

**This guide covers:**
* What the problem is
* Why normal Git commands fail
* How to fix it on:
  * Windows
  * Linux / macOS (Bash)
  * GitHub Codespaces (browser-based)
* Common errors and their solutions

---

<br>
<br>

## Important Rule (Read First)

**If secrets were ever pushed to GitHub:**

* Assume they are compromised
* History cleanup is required
* Secrets **must be rotated** (new passwords, new keys)

History cleanup alone does NOT make leaked secrets safe.

---

<br>
<br>

## What We Are Trying to Do

**Goal:**

* Remove `.env` from **all commits**
* Make it look like `.env` never existed
* Push clean history back to GitHub

<br>

**Tool used:**

* **`git filter-repo`** is used to rewrite Git history. It helps remove or change files across all past commits, like deleting secrets or large files. Once used and pushed, history changes and others must re-clone or reset.

---

<br>
<br>

## Prerequisites

### General

* Git installed
* Access to the GitHub repository
* Permission to force-push (you are repo owner)

### Check Git version

```bash
git --version
```

Git 2.22 or newer is recommended.

---

<br>
<br>

## Common Mistakes (What NOT to Do)

These do not fix the problem:

```bash
git rm .env
```

Deleting the file on GitHub UI

```bash
git revert
```

Making a new commit without the file

All of these leave secrets in old commits.

---

<br>
<br>

## OPTION 1: GitHub Codespaces (BEST & SAFEST)

**This is the best option when:**

* Office PC is locked
* No admin access
* No Git Bash
* No ability to install system tools

<br>
<br>

### Step 1: Open Codespaces

Go to the GitHub repo
Click **Code**
Click **Codespaces**
Create a new codespace
Wait for the browser terminal to open.

<br>
<br>

### Step 2: Install git-filter-repo

In Codespaces terminal:

```bash
python3 -m pip install --user git-filter-repo
```

<br>
<br>

### Step 3: Add it to PATH

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

<br>
<details>
<summary><mark><b>Explanation of above commands</b></mark></summary>
<br>

- These commands add `~/.local/bin` to your PATH and apply the change immediately.
- The first line permanently updates your .bashrc so tools installed in `~/.local/bin` can be run from anywhere. The second line reloads .bashrc so the updated PATH works right away without restarting the terminal.

</details>
<br>

Verify:

```bash
git filter-repo --help
```


<br>
<br>

### Step 4: Run history cleanup

First attempt (expected to fail):

```bash
git filter-repo --path .env --invert-paths
```

<br>
<details>
<summary><mark><b>Explanation</b></mark></summary>
<br>

This command rewrites the Git history and removes the `.env` file from all commits.

- `--path .env` targets the `.env` file.
- `--invert-paths` means “delete this path instead of keeping it.”

In simple words, it completely erases `.env` from the repository’s entire history, not just the latest commit.

</details>
<br>

You will see:

```bash
Refusing to destructively overwrite repo history
```

This is normal.

Now run with force:

```bash
git filter-repo --path .env --invert-paths --force
```

What this does:

* Removes .env from every commit
* Rewrites Git history
* Deletes old commit hashes

<br>
<br>

### Step 5: Re-add GitHub remote (IMPORTANT)

`git filter-repo` removes origin on purpose.

**Check:**

```bash
git remote -v
```

**If empty, add it back:**

```bash
git remote add origin https://github.com/<username>/<repo-name>.git
```

**Verify again:**

```bash
git remote -v
```

<br>
<br>

### Step 6: Force push clean history

```bash
git push origin --force --all
git push origin --force --tags
```

<br>
<details>
<summary><mark><b>Explanation</b></mark></summary>
<br>

These commands force-push rewritten history to GitHub.

- `git push origin --force --all` pushes all branches and replaces the remote history with your updated one.
- `git push origin --force --tags` does the same for tags.

In simple words, this makes GitHub match your cleaned local history. Anyone else using the repo will need to re-clone or reset because the old history is gone.

</details>
<br>

This replaces GitHub history with the clean version.

<br>
<br>

### Step 7: Verify cleanup

```bash
git log --all -- .env
```

<br>
<details>
<summary><mark><b>Explanation</b></mark></summary>
<br>

- This command checks whether the .env file still exists anywhere in Git history.
- It searches all branches and all past commits for .env. If nothing is shown, it means .env has been completely removed from the repository history.

</details>
<br>

If there is no output, `.env` is fully removed.

---

<br>
<br>

## OPTION 2: Linux / macOS (Bash)

### Install tool

```bash
pip3 install git-filter-repo
```

Or:

```bash
brew install git-filter-repo
```

### Run cleanup

```bash
git filter-repo --path .env --invert-paths --force
git push origin --force --all
git push origin --force --tags
```

---

<br>
<br>

## OPTION 3: Windows (Local Machine)

### Problem 1: pip not recognized

**Fix:**

```powershell
python -m pip install git-filter-repo
```

<br>
<br>

### Problem 2: git filter-repo not found

Windows Git does not include it by default.

**Recommended fix:**

* Use GitHub Codespaces instead
* Avoid admin permission issues

**If local Windows must be used:**

* Download git-filter-repo
* Place it inside Git’s libexec/git-core folder
* Requires admin access

---

<br>
<br>

## After Cleanup (MANDATORY SECURITY STEPS)

**Even after history is clean:**

**MongoDB Atlas:**

* Delete old DB user
* Create a new DB user
* Generate new connection string

**Application:**

* Generate new SECRET_KEY

**Update everywhere:**

* Jenkins credentials
* ECS task definition
* Local .env

---

<br>
<br>

## Prevent This in Future

**Add to .gitignore:**

```gitignore
.env
.env.*
```

**Never:**

* Commit secrets
* Store credentials in repo
* Trust "private repo" for security

---

<br>
<br>

## Final Notes

Rewriting Git history is safe for personal projects.

Dangerous for large team repos (coordinate first).

Knowing how to fix this is a real DevOps skill.

This guide exists so future-me does not panic again.

---

