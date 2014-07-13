---
layout: post
title: "Rewriting git commit"
date: 2014-07-14 00:13
comments: true
categories: 
- techincal
- git
---

Sometimes when you commit something nasty, you need to rewrite it. Or you may need to split commit into parts. Or whatever.

Use case: You want to remove some files that you added with your commit.

<!-- more -->

# Find the commit

```
git log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
```

Let it be `4ca80f0`.

# Remove files that you don't want in the commit.

```
git rebase -i 4ca80f0~1
```

Replace pick with edit and save file.

Then you need to unstage files in the commit:

```
git reset HEAD^
```

now all files are in state before that commit, and you can add/remove/stage them.

to undo `windows` file changes:

```
git checkout -- windows
```

or if this is a new file you can just delete it

```
rm -rf windows
```

also you can split this commit into several commits.

# Change commit message

Now you probably want to change commit message:

```
git commit --amend
```

# Apply changes

```
git rebase --continue
```

# Profit!
Enjoy you splitted/amended commit.

# References

* [Stackoverflow: How do I selectively revert some changes in a single commit to a single file?](http://stackoverflow.com/questions/11729030/how-do-i-selectively-revert-some-changes-in-a-single-commit-to-a-single-file)
* [Stackoverflow: Reverting part of a commit with git](http://stackoverflow.com/questions/4795600/reverting-part-of-a-commit-with-git/4796144#4796144)
* [Stackoverflow: git selective revert local changes from a file](http://stackoverflow.com/questions/1109069/git-selective-revert-local-changes-from-a-file)