---
title: How to maintain directory level git configs
date: '2021-10-18'
categories: [Programming]
tags: [programming, git]
pin: false
math: false
mermaid: false
---

When working with multiple git profiles, managing different git configs is hard. For example, you may be working on three repositories `A-first-project`, `A-second-project` and `B-first-project`, commits for which need to be from different E-Mail IDs. What's the best way to acheive this?
<!--more-->

For example, lets assume you want to use E-Mail `you@A.com` for commits in `A-first-project` and `A-second-project`, but the E-Mail `you@B.com` for `B-first-project`. 

One option will be to navigate to each repository and run `git config user.email <desired@email.com>` in each of them, but that is not scalable because that is something which needs to be done each time you clone or create a new repository. So, is there a better way? Turns out, there is - using conditionals and directory level git configs.

**Note that this needs atleast `git 2.13` to work.**

First, you need to structure your repositories in a fashion like this, for example:

```bash
.
└── ~ # ~ refers to your home directory
    └── workspaces
        └── company_A
            ├── .gitconfig
            ├── /A-first-project/
            └── /A-second-project/
        └── personal
            ├── .gitconfig
            └── /B-first-project/
```

Now, time to add conditionals to your your global gitconfig, which should reside at `~/.gitconfig`. (if it doesn't exist, run `git config --global user.name "<Your Name>"` to generate it.)

```bash
$ vi ~/.gitconfig
```
```
[user]
    name = Ayush Gupta

[includeIf "gitdir:~/workspaces/company_A/"] # for all git repositories in `~/workspaces/company_A/`
    path = ~/workspaces/company_A/.gitconfig # override global git config with the config file at this path

[includeIf "gitdir:~/workspaces/personal/"] # for all git repositories in `~/workspaces/personal/`
    path = ~/workspaces/personal/.gitconfig # override global git config with the config file at this path
```

Now, modify `~/workspaces/company_A/.gitconfig` and `~/workspaces/personal/.gitconfig` as follows:

`~/workspaces/company_A/.gitconfig`:

```bash
$ vi ~/workspaces/company_A/.gitconfig
```
```
[user]
    email = you@A.com
```

`~/workspaces/personal/.gitconfig`:

```bash
$ vi ~/workspaces/personal/.gitconfig
```
```
[user]
    email = you@B.com
```

And you're done. To try it out:

```bash
$ cd ~/workspaces/company_A/A-first-project/
$ git config user.email
you@A.com
```

```bash
$ cd ~/workspaces/personal/B-first-project/
$ git config user.email
you@B.com
```