---
title: Treating S3 as a git object store
date: 2021-05-14T00:00:00-07:00
draft: false
type: post
tags:
  - software
description: |
  May 2021:
  Built a rust command line tool to store git repositories in S3
---

In May 2021 my interest in learning Rust had really taken off, but I needed to
find a toy project to work on so I could dive into it. I'd been fascinated by
Git's internals since I learned about them in 2018, and I used this as an
excuse to build a remote helper letting git treat an S3 object store as a
remote repository.

## What's an object store?

An object store, the most common of which is Amazon's S3, is a storage solution
where you can store and generic data using a key. They've exploded in
popularity in the past few years due to their flexibility and low cost. People
can store retrive whatever data they want in the cloud using a standardized
API. Pricing for these objects stores is a different story, but it's been
really helpful to developers to have such a flexible and standardized storage
option.

## How does this relate to git?

First, let's take about what git is internally. Borrowing from the [git
reference book](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) -
"the core of Git is a simple key-value data store". Git works by building a
Merkle tree, where objects are referred to by their hashes and combined
together to show a complete description of the source code in a given state.
There are three main git objects used to do this; commits, trees, and blobs.
Blobs are just raw content, usually text, but really can be any file. When git
indexes any file it prepends a header to the file content saying it's a binary
blob, zlib compresses it, and then stores it using the SHA-1 hash of the
content and header as the key. This blob is then referred to in a tree object,
describing each directory under version control. This tree object is normally
just a list of file names and the hashes of the blob that include the file
content, and is compressed and hashed in a similar way to blobs. These tree
objects can reference other trees, meaning that there's one tree that can
reference the state of the entire filesystem at once. This tree is then saved
with a short description and metadata about that filesystem state into a commit
object. These separate commits together also reference the commits previous to
them, building a second Merkle tree to walk through the different filesystem
states.

That was a long description, but what's crazy is *that's it*. Git has some
additional features with garbage collection and efficiently sharing multiple
objects, but at it's core it's really just a hash tree saved using an object
store. That simplicity is crazy to me, but it begs the question, can we use S3
as a git repository?

Short answer: *absolutely*. Should we though? Probably not.

## Putting it together

So far we've just talked about how git saves objects locally. To keep different
repositories in sync it has a concept of remote repositories. These are just
remote servers that you can publish a object store to so multiple people can
download and upload their changes and collaborate. Built in git is able to
communicate to these external repositories using https and ssh, but they
include support for
[remote-helpers](https://git-scm.com/docs/gitremote-helpers). These are helper
scripts that git will call to be able to communicate with different remotes.
There's a couple well known helper scripts floating around, namely ones for
[dropbox](https://github.com/anishathalye/git-remote-dropbox) and
[ipfs](https://github.com/cryptix/git-remote-ipfs).
Using this as an excuse to get more familiar with git's internals, and to
develop something using rust, I set out to built a remote helper that could
communicate with a S3 bucket.

## How does a git remote-helper work?

## Lessons learned (mostly about rust)

* Rust error handling with Enums is really nice
* No free-lunch, compilation time and binary size with rust is crazy.

And it works! 
```
# Add local binary to path so git will run it
$ export PATH=$PATH:$(pwd)/target/release

# Push a repository to s3
$ git remote add backblaze s3://s3.us-west-002.backblazeb2.com/remote-s3-test
$ git push backblaze main
<wait a *very* long time>
Everything up-to-date

# What's in the bucket? The git object store
$ b2 ls remote-s3-test | tail -n 5
e1d1bce81d6776b6dbbdeddfe2244f3d8725cb0e
e7298343fb974f86e154c4b3011d85f118d0a195
ee0a1159d9a4184234dc8917b02f40988cba5af3
ef1dc7ed3ad089ee9fe1492439bbe73e4e9c54cc
refs/
$ b2 ls remote-s3-test refs/heads
refs/heads/main

# Clone the repo (we need to specify the branch b/c we can't don't have a default set)
$ git clone s3://s3.us-west-002.backblazeb2.com/remote-s3-test -b main
Cloning into 'remote-s3-test'
$ cd remote-s3-test
$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

There are definite more improvements to be made, like handling pack files, garbage
collection in the external repository, parallelism, and compressing objects in
S3. Additionally it requires a compatible version of libssl on the host (using
rust-native ssl libraries made the binary 250 MB!). This also begs the question
of what value this has. Normally when a git repository is hosted, there's a
fair bit of management that needs to be done on the server side. This includes
managing SSH keys, ensuring the different repositories have enough space and
are backed up, managing the ssh/git server processes, etc. In short you would
need to manage a compute and storage system to self-host git, or build
automation to add authentication to a remote git server. In this case you only
to worry about the storage consumed by the repositories. Authentication can be
easier integrated with exisiting IAM tools already tied into S3. However,
granular role controls are lost. Write access to the bucket to push testing
branches allows overridding production branches.

Regardless of the downsides, currently this tool it works
as a proof of concept to push and pull repositories to S3 buckets. If you're
interesting in reading more about it, the code is hosted on
[Github](https://github.com/josephvoss/git-remote-s3). along with pre-built
binaries.

## Want to learn more? Go read these posts by people who know what they're doing

* ["How to Write a New Git Protocol"](https://rovaughn.github.io/2015-2-9.html)
* ["Developing a Custom Remote Git Helper"](https://www.apriorit.com/dev-blog/715-virtualization-git-remote-helper)
