---
title: Keeping GitHub pull request body and Git commit message in sync
date: 2022-12-15
tags:
  - linux
---

TL;DR: We should write proper commit messages - not only GitHub pull request descriptions.

<!--more-->

Every pull request on GitHub should have some introduction - this is called body.
Then we have some commits and those commits have their introduction - that is called comment.

Then after your pull request gets merged it is not trivial to find it.
But the commits are more visible and easily accessible from web and terminal.

So it is good to not be lazy and write your commit message properly. But
nowadays people tend to rather work on the GitHub web UI and kinda forget about
commits.

Here are some basic `git` commands:

```bash
# Show last commit message
$ git log -1 --pretty=%B

# Edit last commit message
$ git commit --amend
```

But we have also GitHub's CLI tool `gh`:

```bash
# Show current pull request
$ gh pr view

# Show body of current pull request
$ gh pr view --json body | jq -r '.body'

# Change body of current pull request
$ gh pr edit

# Change body of current pull request non-interactivelly
$ gh pr edit -b 'sadfasdf\nsfsfd'
```

So what if we mix those two tools together? Let's take a look:

```bash
# Set pull request body the same as last commit message
$ gh pr edit -b "$(git log -1 --pretty=%B)"

# Set last commit message the same as pull request body
$ git commit --amend -m "$(gh pr view --json title,body | jq -r '(.title) + "\n\n" + (.body)')"
```

## TO-DO:
 * Adjust the last two commands for multi commit pull requests

