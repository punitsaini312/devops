# Git & GitHub Interview Questions (DevOps - 1 Year Experience)

## 1. Git vs GitHub

**Q:** What is the difference between Git and GitHub?

**Answer:** - **Git** is a distributed version control system used to
track code changes. - **GitHub** is a cloud platform that hosts Git
repositories and provides collaboration features like Pull Requests,
Issues, and Actions.

------------------------------------------------------------------------

## 2. Initialize a Repository

**Q:** How do you initialize a Git repository?

``` bash
git init
```

Creates a `.git` directory and starts tracking the project.

------------------------------------------------------------------------

## 3. Clone a Repository

``` bash
git clone <repo-url>
```

Downloads a remote repository and its complete history.

------------------------------------------------------------------------

## 4. Create a Branch

``` bash
git checkout -b feature-branch
# or
git switch -c feature-branch
```

Creates and switches to a new branch.

------------------------------------------------------------------------

## 5. Basic Workflow

``` bash
git status
git add .
git commit -m "message"
git push origin feature-branch
```

------------------------------------------------------------------------

## 6. Merge

**What happens?**

``` text
        X──Y──Z
       /      \
1──2──3──4─────M
```

-   Combines two branches.
-   Creates a **merge commit** (`M`) when needed.
-   Preserves branch history.

------------------------------------------------------------------------

## 7. Rebase

``` text
Before:

        X──Y──Z
       /
1──2──3──4

After:

1──2──3──X'──Y'──Z'──4'
```

-   Replays commits on top of another branch.
-   Produces a **linear history**.
-   No merge commit.

------------------------------------------------------------------------

## 8. Cherry-pick

``` bash
git checkout main
git cherry-pick <commit-hash>
```

Copies **one specific commit** from another branch.

Example:

``` text
main:
1──2──3──4

dev:
1──2──3──4──5──6──7──8

Cherry-pick commit 7

Result:

main:
1──2──3──4──7'
```

------------------------------------------------------------------------

## 9. Branching Example

``` text
                     5──6──7──8 (main)
                    /
1──2──3──4
                    \
                     X──Y──Z (feature)
```

Both branches share commits 1--4 and then diverge.

------------------------------------------------------------------------

## 10. Local Project → GitHub

``` bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <repo-url>
git branch -M main
git push -u origin main
```

------------------------------------------------------------------------

## 11. Existing GitHub Repository

``` bash
git clone <repo-url>
git checkout -b feature
git add .
git commit -m "message"
git push origin feature
```

Create a Pull Request after pushing.

------------------------------------------------------------------------

## 12. Configure Git

Global configuration:

``` bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Repository only:

``` bash
git config user.name "Office Name"
git config user.email "office@example.com"
```

Check configuration:

``` bash
git config --list
git config --global --list
```

------------------------------------------------------------------------

## Important Commands to Remember

``` bash
git status
git log --oneline
git branch
git checkout
git switch
git add
git commit
git push
git pull
git fetch
git merge
git rebase
git cherry-pick
git stash
git stash pop
git reset
git revert
git reflog
git remote -v
git tag
```

------------------------------------------------------------------------

## Frequently Asked Interview Questions

1.  What is Git?
2.  What is GitHub?
3.  Git vs GitHub?
4.  What is a branch?
5.  Why do we create feature branches?
6.  Merge vs Rebase?
7.  Cherry-pick use case?
8.  Fetch vs Pull?
9.  Reset vs Revert?
10. Soft, Mixed and Hard reset?
11. What is HEAD?
12. What is origin?
13. What is a merge conflict?
14. How do you resolve merge conflicts?
15. What is git stash?
16. What is git reflog?
17. What is a Pull Request?
18. Fast-forward merge vs non-fast-forward merge?
19. How do you recover a deleted commit?
20. Explain your team's Git workflow.
