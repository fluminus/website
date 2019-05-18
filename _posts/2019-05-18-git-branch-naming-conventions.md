---
layout: post
title: Git Branch Naming Conventions
date: 2019-05-18 17:54 +0800
categories: fluminus git
---

This is for the reference of our development.

<!--more-->

# How to name branches properly


## Main branches

TL;DR: always merge to `master`, when `master` is stablized and deployed, merge to `stable` tagged with a **release number**.

| Instance | Branch | Description                                      |
|----------|--------|--------------------------------------------------|
| Stable   | stable | Accepts merges from Working and Hotfixes         |
| Working  | master | Accepts merges from Features/Issues and Hotfixes |

The main branch should be considered `origin/master` and will be the main branch where the source code of HEAD always reflects a state with the latest delivered development changes for the next release. As a developer, you will be branching and merging from master.

Consider `origin/stable` to always represent the latest code deployed to production. During day to day development, the stable branch will not be interacted with.

When the source code in the master branch is stable and has been deployed, all of the changes will be merged into stable and tagged with a release number.

## Format for supporting branches

```
reason__details--tag
```

Pattern is divided into 3 parts : reason, details and tag. 

At max a Branch name can have at **4 words**, but not more than that.

### Reason

| Instance     | Branch   | Description                                                                        | Branch from ... |
|--------------|----------|------------------------------------------------------------------------------------|-----------------|
| Feature      | feature  | New features that add noticeable functionality to the app.                         | master          |
| Minor update | minor    | Minor changes to the code, documentation, UI, etc.                                 | master          |
| Fix          | fix-[id] | Fixing an issue, [id] should be the issue(s)' GitHub ID. (e.g. fix-3__fix-details) | master          |
| Hotfix       | hotfix-[id]   | Fixing a bug in production.                                                        | stable          |

#### Work with a `feature` or `minor` branch

```bash
$ git checkout -b feature__feature-detail--tag master                 // creates a local branch for the new feature
$ git push origin feature__feature-detail--tag                        // makes the new feature remotely available
```

Perform merge via GitHub webpage.

#### Work with a `fix` branch

```bash
$ git checkout -b fix-id__fix-detail master                     // creates a local branch for the new bug
$ git push origin fix-id__fix-detail                            // makes the new bug remotely available
```

#### Work with a `hotfix` branch

This is unlikely to happen soon since we're not in production... but just in case we hit that stage.

```bash
$ git checkout -b hotfix-id__fix-detail stable                  // creates a local branch for the new hotfix
$ git push origin hotfix-id__fix-detail                         // makes the new hotfix remotely available
```

### Tag

It is optional and can be used under special circumstances, like :

- if you use any external tracker like pivotal and want to add its tracking id to it. or
- any other extra details you want to add to your branch.

Use only one-word description for this.

## Workflow diagram

![workflow](https://camo.githubusercontent.com/f011896cab0a6e086954a10d3a5132d57ca69468/687474703a2f2f662e636c2e6c792f6974656d732f3369315a336e3154316b3339327231413351306d2f676974666c6f772d6d6f64656c2e3030312e706e67)


References:
[1](https://codeburst.io/let-the-branch-name-do-all-the-talking-in-git-e614ff85aa30), [2](https://gist.github.com/digitaljhelms/4287848)