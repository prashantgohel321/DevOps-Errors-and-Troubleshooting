# Git Errors

<details>
<summary><strong style="color:red"> 01. fatal: not a git repository (or any of the parent directories): .git </strong></summary>
<br>

### Cause
> The command is executed outside a Git repository or the repository is corrupted.

### Solution
1. Verify you're in the correct directory: pwd.
2. Navigate to the repository: cd <repository-path>.
3. If missing, reinitialize the repository: git init. 

</details>

<br>

<details>
<summary><strong style="color:red"> 02. "error: failed to push some refs </strong></summary>
<br>

### Cause
> Local branch is out of sync with the remote branch. 

### Solution
1. Pull latest changes with rebase: `git pull --rebase origin [branch_name]`.
2. Resolve conflicts if prompted, then commit and push.

</details>

<br>

<details>
<summary><strong style="color:red"> 03. Permission denied (publickey) </strong></summary>
<br>

### Cause
> SSH key is missing or not recognized by the remote repository. 

### Solution
1. Generate a key pair: `ssh-keygen -t rsa -b 4096`. 
2. Add the private key to the agent: `ssh-add ~/.ssh/id_rsa`. 
3. Add the public key to the repository settings. 


</details>

<br>

<details>
<summary><strong style="color:red"> 04. Merge conflict in [file] </strong></summary>
<br>

### Cause
> Simultaneous changes to the same part of a file. 

### Solution
1. Open the conflicting file to resolve changes. 
2. Use markers like <<<<<<< and >>>>>>> to identify conflicts.
3. After resolution, commit: `git add <file>` && `git commit`. 


</details>

<br>

<details>
<summary><strong style="color:red"> 05. Detached HEAD state </strong></summary>
<br>

### Cause
> A specific commit is checked out instead of a branch. 

### Solution
1. Create a branch from the detached state: `git checkout -b <new_branch>`. 
2. Continue working or merge with another branch. 

</details>

<br>

<details>
<summary><strong style="color:red"> 06. fatal: remote origin already exists </strong></summary>
<br>

### Cause
> Adding a remote repository that already exists. 

### Solution
1. Update the existing remote: `git remote set-url origin <new-URL>`.

</details>

<br>

<details>
<summary><strong style="color:red"> 07. Large file exceeds limit </strong></summary>
<br>

### Cause
> Attempt to push a file exceeding the repository limit (e.g., GitHub's 100MB limit). 

### Solution
1. Use Git Large File Storage (LFS): `git lfs track "<file-pattern>"`. 

</details>

<br>

<details>
<summary><strong style="color:red"> 08. Submodule update failed </strong></summary>
<br>

### Cause
> Incorrect or inaccessible submodule configuration.

### Solution
1. Update submodules: `git submodule update --init --recursive`.

</details>

<br>

<details>
<summary><strong style="color:red"> 09. "fatal: cannot lock ref </strong></summary>
<br>

### Cause
> A .lock file is preventing operations due to an interrupted process. 

### Solution
1. Remove the .lock file: `rm -f .git/index.lock`. 

</details>

<br>

<details>
<summary><strong style="color:red"> 10. Untracked files prevent switching branches </strong></summary>
<br>

### Cause
> Untracked changes conflict with the branch switch. 

### Solution
1. Stash changes: `git stash`. 
2. Switch branches: `git checkout <branch>`. 

</details>
