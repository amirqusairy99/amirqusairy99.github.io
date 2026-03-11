---
title: "GIT Cheatsheet"
description: A quick reference guide for essential Git commands including staging, committing, and managing changes.
date: 2026-03-11
categories: [Devops, Git]
tags: [git, github, cheatsheet]
---

##Introduction
Git is a free, open-source, distributed `version control system (VCS)` designed to manage and track changes in source code and other files during software development

| Area | Description |
|------|-------------|
| Working Directory | your actual project files |
| Staging Area (Index) | where you prepare changes for the next commit |
| Repository (Commit history) | where commits are stored permanently |

Think of it like this:

- Working directory → you're editing a document
- Staging area → you're selecting which edits to include
- Commit → you're saving a final version

---
## Git Setup
Configuring user information used across all local repositories
```
git config --global user.name “[firstname lastname]”
```
set a name that is identifiable for credit when review version history

```
git config --global user.email “[valid-email]”
```
set an email address that will be associated with each history marker

```
git config --global color.ui auto
```
set automatic command line coloring for Git for easy reviewing

```
git config --global --list
```
to check global list settings

`Example output:`
```
user.name=John Doe
user.email=johndoe@email.com
core.editor=vim
```

---
## Setup & Init
Configuring user information, initializing and cloning repositories
```
git init
```
initialize an existing directory as a Git repository

```
git clone <url>
```
retrieve an entire repository from a hosted location via URL

## Stage and Snapshot
Working with snapshots and the Git staging area

```
git status
```
show modified files in working directory, staged for your next commit

```
git add [file] / git add . --> add everything
```
moves a file from working directory → staging area

```
git reset [file]
```
unstages a file but keeps your changes

```
git diff \ git diff <filename>
```
shows the exact code changes you made but haven't staged yet

```
git diff --staged
```
diff of what is staged but not yet committed

```
git commit -m “[descriptive message]”
```
save changes to the local repository

---
## Branch & Merge
```
git branch
```
manage and view branches

```
git checkout
```
switch between branches or restore files

```
git merge <branch>
```
merge the specified branch’s history into the current one

```
git log
```
show all commits in the current branch’s history

```
git log branchB..branchA
```
shows commits that are in branchA but not in branchB
- useful before merging or creating a pull request to see what new commits you’re bringing in

```
git log --follow [file]
```
--follow tells Git to track the file history across renames

```
git diff branchB...branchA
```
shows the difference (diff) between the tips of two branches relative to their common ancestor

```
git show [SHA]
```
show any object in Git in human-readable format

---
## Share & Update
Retrieving updates from another repository and updating local repos

```
git remote add [alias] [url]
```
add a new remote repository to your local Git project
- alias is like a nickname for the remote repository (e.g., origin, upstream).
- url is the repository’s location (HTTPS or SSH).
For example: `git remote add upstream https://github.com/otheruser/project.git`

```
git fetch [alias]
```
fetch down all the branches from that Git remote

```
git merge [alias]/[branch]
```
merge changes from a remote branch into your current branch

```
git push [alias] [branch]
```
upload your local commits to a remote repository

```
git pull
```
fetch and merge any commits from the tracking remote branch

Credits: `https://education.github.com/git-cheat-sheet-education.pdf`
Cheatsheet PDF: assets/img/files/git-cheat-sheet-education.pdf


