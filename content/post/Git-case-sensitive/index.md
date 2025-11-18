---
title: "PSA: Git is case-insensitive (on MacOS)"
date: 2025-11-18T10:00:00-05:00
draft: false
type: post
tags:
  - git
  - software
description: |
  Nov 2025:
  The joys of case sensitivity
---

Did you know git doesn't care about case sensitivity? Make sense, it should
treat `foobar` and `FOOBAR` as totally different objects. **Except** when it's
running on one of the wonderful case-insensitive filesystems like APFS
(Apple File System), which is the default on MacOS.

This can manifest in weird ways. Obviously it will cause issues with files
that are identical except for capitalization, but what about internal git
objects? I found out about this because someone pushed both `branch-name` and
`BRANCH-name` to a git server. The error from git was less than clear.

```
error: cannot lock ref 'refs/remotes/origin/branch-name': Unable to create
'.git/refs/remotes/origin/branch-name.lock': File exists.

Another git process seems to be running in this repository, e.g.
an editor opened by 'git commit'. Please make sure all processes
are terminated then try again. If it still fails, a git process
may have crashed in this repository earlier:
remove the file manually to continue.
```

There's a great [StackOverflow answer][so-schism] explaining this in more
detail, but the short version is once my local git binary fetches the ref it
doesn't see any problem because it's just reading the branch name from the
packed-ref. Trying to extract it to the filesystem fails though, and manifests
as an `Unable to create` file error.

This can be "fixed" by converting the refs in the local repo from the
`file` format to [`reftable`][reftable], where the refs are stored in a local binary
format instead of raw filesystem objects. GitLab has a great
[blog][gitlab-reftable] describing this feature in more detail.

```
$ git refs migrate --ref-format=reftable
error: migrating repositories with worktrees is not supported yet
```

Guess I'll have to wait a while for this feature to be more mature though.

In the meantime:
* (if possible) APFS supports case sensitivity, but changing this requires reformatting the filesystem.
    * I can't easily do this on my work machine due to external controls, but this is the best fix if you can.
* Add a pre-receive hook to block case-insensitive filenames
    * Most git servers support regex matching, but matching against already exisiting branches is harder
    * This would only catch one type of errors, and I suspect there's several others :(
    * Also would allow new branches to be pushed but it's still possible for a user to have an old local tracking branch ref that would conflict
* Ask politely for devs to not push conflicting branch names
    * good luck :)
* Be aware of case issues and nuke remote branches if it becomes a problem

## Refs

* [Reftable man page][reftable]
* [StackOverflow - torek][so-schism]
* [StackOverflow - branches][so-branches]
* [GitLab - What's new in 2.45][gitlab-reftable]

[reftable]: https://git-scm.com/docs/reftable
[so-schism]: https://stackoverflow.com/a/55053489
[so-branches]: https://stackoverflow.com/a/38494084
[gitlab-reftable]: https://about.gitlab.com/blog/whats-new-in-git-2-45-0/#reftables-a-new-backend-for-storing-references
