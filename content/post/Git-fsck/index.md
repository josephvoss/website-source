---
title: How `git fsck --lost-found` saved my lost work
date: 2024-09-24T00:00:00-05:00
draft: false
type: post
tags:
  - software
description: |
  Sept 2024:
  Just a blog post showing off a helpful git feature
---

tl;dr I did `git reset HEAD` without thinking, but was able to recover
my lost work using a helpful git feature that I wanted to highlight. Learn from
my mistakes and how to undo them.

I've been working on changes for a local project when my editor started spewing
weird linter errors, so I went digging a little to fix that. This involved
restarting vim several times. My computer had also crashed a couple times in
recent days which left several `.swp` files dangling. I had been good about
removing most of them but I missed one `.swo` file that was remaining. Long
story short when I recovered from this file it removed several hours worth of
recent changes.

No worries though, I'll just restore from the last commit right? Except it
wasn't until after I ran `git reset HEAD` that I realized my latest commit
was several hours old. Thankfully I had run git add consistently, so I knew my
changes existed somewhere in my local object store. They just weren't tied to a
commit or really any tracking tools.

Enter `git fsck --lost-found`

```bash
--lost-found
    Write dangling objects into .git/lost-found/commit/ or
    .git/lost-found/other/, depending on type. If the object is a blob, the
    contents are written into the file, rather than its object name.
```

My changes existed somewhere as a dangling blob in the object store, so with a
list of these blobs I figured I might be able to recover. Turns out it was much
easier than I though it'd be.

```bash {linenos=table}
$ git fsck --lost-found | grep blob | tail -5
Checking object directories: 100% (256/256), done.
Checking objects: 100% (3986/3986), done.
dangling blob 7cfd69a20f6fe079b76e0eac62585acb544edd57
dangling blob f43e095668cdcb2274936e2cfdb86b3ef4326f62
dangling blob f6fee461567174ad01662420347bf979569de806
dangling blob 43ff423b391e642b53083bbe00f361863ff67ab2
dangling blob e17f0d9bed49f29d1fe101332a79026a54620a92
```

Other than just writing out objects to the `.git/lost-found` dir, `git fsck`
also prints the blobs/commits it finds dangling. We can extract these blobs and
grep through them to hopefully find my changes.

```bash {linenos=table}
$ for blob in $(git fsck --lost-found | grep blob | cut -d ' ' -f3); \
    do git cat-file -p $blob | grep "package foobar" && echo $blob; done
Checking object directories: 100% (256/256), done.
Checking objects: 100% (3986/3986), done.
package foobar 
069dac305daa21948d2ec3f52f2b680992b58fec
$ git cat-file -p 069dac305daa21948d2ec3f52f2b680992b58fec | head
package foobar
 
import (
        "fmt"
...
```

In sum, firstly don't get into this state
* Commit early and often, you can clean up changes later
* Don't blindly reload from old swp files
* Don't blindly run `git reset`

If you ignore all that advice though and lose some changes, all hope isn't
lost. Git has some extremely useful recovery tools if you know where to look.
