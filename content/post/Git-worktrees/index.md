---
title: Why `git worktree` is fantastic + vim tabs
date: 2025-05-03T00:00:00-05:00
draft: false
type: post
tags:
  - software
description: |
  May 2025:
  Plug for `git worktree`
---
Earlier this year I learned about `git worktree`, and it's been a lifesaver
for my development workflow. I've talked to a couple people about why it's so
great for me (ymmv), but I wanted to spell out some reasons in a longer form.

## Alright but what is this?

Okay scenario. You're in the middle of working on developing a new feature, and
got roped into an incident, push a critical bug fix, asked to review someone,
or any other hard context switch. What I used to do in this situation:

```
$ git add <touched files>; git commit -m "bad in progress commit fix me later"
or
$ git stash -m "working on <new feature>"
and then
$ git checkout origin/main
...and work
```

But then I would lose all context on whatever I was doing, and would have to
inspect the diff of the commit or stashed files and try decipher what my next
steps are. Also, this usually required closing my editor and losing all the
open files and marks for what I was looking at. Clawing back conte
back to where I was before the hard switch can take some time.

*But wait!* What if you're interrupted in the middle of *that* context switch? Do
you tell someone "no I'm busy go away", or try to manage another switch?
Eventually at the end of a bad day I could have 3 or 4 in progress commits all
in various stages of broken-ness on separate branches, or (worse) several
dangling files saved in my stash list. I might just not be great at git, but
for me trying to figure out what is going on for each of these branches is
challenging and takes a lot of energy (before even working on what I was trying to do).

Enter `git worktree`. Essentially these are just a copy of the repository in a
different directory. It's a step above having the same repo cloned to several
different places on disk, but lets you share the local git object store between
them. You can (and likely should) get started using them in any repo you're
currently using, but eventually it's easier to have a `bare` repo and only use
worktrees.

## What does it look like?

In a non-bare repo, it makes a new directory containing the index files for the
repo. In this [website's source
repo](https://github.com/josephvoss/website-source), it looks like this.
```
$ ls
archetypes  build.sh  config.yaml  content  layouts  LICENSE.md  static
$ git worktree add demo-worktrees
Preparing worktree (new branch 'demo-worktrees')
HEAD is now at 720eaf5 Merge pull request #5 from josephvoss/git-fsck/update-codeblock
[jvoss@DCXCQXXY0K:website-source (git-worktree)]$ ls
archetypes  config.yaml  demo-worktrees  LICENSE.md
build.sh    content      layouts         static
$ ls demo-worktrees/
archetypes  build.sh  config.yaml  content  layouts  LICENSE.md  static
$ cd demo-worktrees/
$ git status
On branch demo-worktrees
nothing to commit, working tree clean
```

This new directory creates and checks out a new branch (by default the name of
the worktree), and is independent of the root repository. You can develop here
without impacting root or any other worktree. I wasn't kidding, it's literally
a step above `git clone <repo-url> repo-feature-a && git clone <repo-url>
repo-feature-b`.

## My workflow

Note this is using a couple vim plugins (🤓), but you can do something similar
in whatever editor you use.
"Hey Joseph can you do \<x\>"
"Sure thing"
```
with the linked ticket / incident / whatever short name I want to use
# open new vim tab (s/o https://github.com/gcmt/taboo.vim)
:TabooOpen ABC-1234
# create new worktree and create a new branch
# (s/o https://github.com/tpope/fugitive.vim)
:G worktree add ABC-1234 -B jvoss/ABC-1234
# Open files, make commits, do whatever work you need to
```

Then, whenever I need to context switch either back to what I was doing before
(or something new), I just need to switch back to the other tab (`gT`) that's already
open and ready to go. Bonus points to save the session so these tabs and buffers persist through reboots.
```
:mksession! -s ~.vim/sessions/<repo-name>
```

Slight issue though: Vim buffers are global, not per tab. Trying to switch
buffers by file name will pull up all versions of that file open for each
worktree. For me it's not awful to tab over to the one I'm looking for, and use
`:bd <worktree> C-a` to unload all buffers under a worktree when finished. YMMV though 🤷.

## Converting to bare repos
To convert old repo style to worktrees, just delete all the non-worktree dirs and files, then tell `.git` to be a bare repo.
```
# Delete all files in . other than `.git` and the worktrees
# (it's a nasty oneliner, I know)
$ find . -d 1 -not -name ".git" -not -name \
    \"$(git worktree list --no-porcelain | tail -n +2 | cut -d ' ' -f 1 | \
    rev | cut -d '/' -f 1 | rev | paste -sd ',' | sed 's/,/" -not -name "/g') \
    -exec rm -rf {} \;
# Set the repo to bare
$ git config core.bare true
```

