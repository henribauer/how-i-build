---
type: code_log
title: git init with a separate git dir fails on a missing parent directory
date: 20260918
tags: [git, icloud]
---

# git init --separate-git-dir fails when the parent directory is missing

## Task

Give the workspace root a git history without putting the object store in iCloud.

## What happened

```
$ git init --separate-git-dir="$HOME/claude-local/workspace/git"
fatal: Invalid path '/Users/<me>/claude-local/workspace': No such file or directory
```

Git creates only the last path component of `--separate-git-dir`, not its parents.

## Fix

```
mkdir -p "$HOME/claude-local/workspace"
git init --separate-git-dir="$HOME/claude-local/workspace/git"
```

Only a `.git` pointer file lands in iCloud. `git status` works from the iCloud folder.

## Reuse

Any new repo in the workspace: create the parent first. The workflow's git-check step now includes
the `mkdir -p` in the command it proposes.
