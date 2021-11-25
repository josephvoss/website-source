---
title: Treating S3 as a git object store
date: 2021-05-14T00:00:00-07:00
draft: false
tags:
  - software
description: |
  May 2021
  Built a rust command line tool to store git repositories in S3

---

## What's an object store?

An object store, the most common of which is Amazon's S3, is a storage solution
where you can store and generic data using a key. They've exploded in
popularity in the past few years due to their flexibility and low cost. People
can store retrive whatever data they want in the cloud using a standardized
API. Pricing for these objects stores is a different story, but it's been
really helpful to developers to have such a flexible and standardized storage
option.

## How does this relate to git?

Borrowing from the [git reference book]() - "the core of Git is a simple key-value
data store". Git works internally by building a Merkle tree, where objects are
referred to by their hashes and combined together to show a complete
description of the source code in a given state. There are three main git
objects used to do this; commits, trees, and blobs. Blobs are just raw content,
usually text, but really can be any file. When git indexes any file it
prepends a header to the file content saying it's a binary blob, zlib
compresses it, and then stores it using the SHA-1 hash of the content and
header as the key. This blob is then referred to in a tree object, describing
each directory under version control. This tree object is normally just a list
of file names and the hashes of the blob that include the file content, and is
compressed and hashed in a similar way to blobs. These tree objects can
reference other trees, meaning that there's one tree that can reference the
state of the entire filesystem at once. This tree is then saved with a short
description and metadata about that filesystem state into a commit object.
These separate commits together also reference the commits previous to them,
building a second Merkle tree to walk through the different filesystem states.

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
include support for "remote-helpers". These are helper scripts that git will
call to be able to communicate with different remotes. There's a couple well
known helper scripts floating around, namely ones for [dropbox]() and [ipfs]().
Using this as an excuse to get more familiar with git's internals, and to
develop something using rust, I set out to built a remote helper that could
communicate with a S3 bucket.

And it works! There were a few small challenges like how to handle
authentication to S3, but overall the process was relatively smooth. There are
definite more improvements to be made, like handling pack files, garbage
collection in the external repository, parallelism, and compressing objects in
S3. Additionally it requires a compatible version of libssl on the host (using
rust-native ssl libraries made the binary 250 MB!). Currently though it works
as a proof of concept to push and pull repositories to S3 buckets, and the
binaries can be downloaded from the latest release on [Github].
