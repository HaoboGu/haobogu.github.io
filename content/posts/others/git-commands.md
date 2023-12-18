---
title: "Powerful Git Commands"
author: "Haobo Gu"
tags: [git]
date: 2019-12-18T19:47:24+08:00
summary: This post records some powerful git commands which I was not familiar with
---

# Git commands

This post records some powerful git commands which I was not familiar with. 

## Merge a specific commit to another branch

```shell
# Merge a commit to current branch
git cherry-pick COMMIT_ID

# Merge commits in a range(without start commit)
git cherry-pick START_COMMIT_ID..END_COMMIT_ID

# Merge commits in a range(with start commit)
git cherry-pick START_COMMIT_ID^..END_COMMIT_ID
```

In the commands above, the first 6 chars of `START_COMMIT_ID` and `END_COMMIT_ID` are enough. 

For example, command`git cherry-pick 496986^...acc9bb`  is same as `git cherry-pick 49698603a9c7d1dfdfef0505b52ab2cd4f7ed1a7^...acc9bb290a39275b59cb18e5eb661563fbf08c43`.


## --patch 

If you want to commit some specific changes, you can use `--patch` option:

```shell
git add --patch <filename>
```

`--patch`(or `-p` for short) is a powerful option, which allows you to pick your interested changes in a file or a commit. 

If you use `--patch` or `-p`, the system will ask you whether to stage changes at every chunk:

```
Apply this hunk to index and worktree [y,n,q,a,d,j,J,g,/,e,?]?
```

The explanation of those options:

```
y - apply this hunk to index and worktree
n - do not apply this hunk to index and worktree
q - quit; do not apply this hunk or any of the remaining ones
a - apply this hunk and all later hunks in the file
d - do not apply this hunk or any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
e - manually edit the current hunk
? - print help
```

The most commonly used options are `y`(accept this chunk) and `n`(ignore this chunk).