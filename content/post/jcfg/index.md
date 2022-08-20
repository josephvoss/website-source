---
title: Rants about configuration management/engine to declare system state as json
date: 2021-12-29T00:00:00-7:00
draft: false
type: post
tags:
  - software
  - configuration management
description: |
  March 2021:
  Wrote a configuration management tool to render json catalogs
---

Selected rants about configuration management.

Configuartion

Idempotent and declarative. Repeatable, should have a compilied
catalog/resource list describing the state of the system. Needs to be flexible
and unopinionated. Every company using a configuration management tool adapts
(and often misuses) it to fit their needs, and eventually they inevitably hit
walls that require significant time and energy to either implement in their
current tooling, or to untangle everything and switch to a new tool. The
current configuration management tooling landscape requires locking in to a
particular ecosystem and accumulating technical debt. Isn't there another way
this could be structed?

*Yes* - build a language agnostic system management engine. Don't worry about
generating the resources to configure, plenty of other tools do that in a
better way. Focus on reading resource definitions and the order they should be
applied in, and enforce them on the system. Somewhat analogous to Kubernetes
resources, just define a pod or service, and don't worry about anything else.
Rely on kustomize/helm/ksonnet/etc to do that for you. Additionally, create
custom resource definitions at will. We can't anticipate every use case, and
maintaining a library of every needed resource that often makes [arbitrary
decisions](https://github.com/puppetlabs/facter/blob/5a5f5da9c58be43c0167e2410a66c469a72f38ca/lib/src/facts/linux/filesystem_resolver.cc#L80)
about the limitations of each resource can lead to issues for users. Since
these decision are often hard coded in to a compiled binary, modifying them to
work for an organization often requires hacky and not easily maintained
workarounds.

Rant about NixOs - right ideas and good direction, but very high learning curve - breaks too much about normal system management/administration - like arch but harder.

For my nix setup - flake to set declarative input, probably use to source secrets from module? Alternatively define system config as modules, and save system render w/ secrets as input in private repo. Or just use sops and age like planned, but something about the project maintainers saying "100% keep these files private tho" and using them for gpg and ssh keys seems a little too risky.
Secrets managed by age/sops then? How to bootstrap age key. Ugh this is bootstrapping authentication again. Need to add a secret file from somewhere. Vault based to fetch age (probs last pass) and then sops sounds like the best idea. Sops repo would be private though, so need ssh key. Back to probably just should fetch this stuff from last pass

What's the end goal here again? Save system config in cloudflare's bitbucket to reprovision from just my phone or yubikey. Personal stuff - yubikey and pass. Work stuff - oauth via okta or lastpass. System config should be public, sourced for deployments in private bitbucket and github. Personal uses yubikey/gpg, work uses age from lastpass.

