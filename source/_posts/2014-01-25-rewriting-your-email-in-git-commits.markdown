---
layout: post
title: "rewriting your email in git commits"
date: 2014-01-25 18:57
comments: true
categories: 
- techincal
- git
- dirty tricks
---

Sometimes you need to rewrite your email in your commits in some git repository.
Do it with command:

    git filter-branch --env-filter 'if [ $GIT_AUTHOR_EMAIL = bad@email ]; then GIT_AUTHOR_EMAIL=correct@email; fi; export GIT_AUTHOR_EMAIL'

For your collaborators this operatioin might be dangerous. According to [Jakub Narębski](http://stackoverflow.com/users/46058/jakub-narebski) collaborators will need to:

* if they didn't base their work on history pre-rewrite, just `git reset --hard origin/master` or just `git pull origin` (which should fast-forward). 
* If they based their change, they have to rebase using `git rebase origin/master` or just `git pull --rebase origin`

More info:

* [Stackoverflow: Rewrite author in history](http://stackoverflow.com/questions/3401732/rewrite-author-in-history)
* [Pro Git Book: 6.4 Git Tools - Rewriting History](http://git-scm.com/book/ch6-4.html)
