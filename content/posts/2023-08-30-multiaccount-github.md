---
title: Multi-account work on GitHub or GitLab.
date: 2023-08-30 
tags: [git, github, gitlab]
description: How to work with multiple accounts on GitHub.
draft: false
disableComments: true
---

**Multitple accounts**

There are situations that you need to work with multiple accounts on GitHub (or GitLab) and use each of them independently of each other.
When that happens you might run into problem that each accounts has it's own SSH key uploaded and when you try to connect to your 
non default account you get an error message saying "Permission denied!".
All of that happened to and below you my solution to it.

{{< notice note >}}
I assume that you use `SSH` method for working with git on GitHub/GitLab.
{{< /notice >}}


Let's assume that you already have multiple accounts on GitHub and each of them have uploaded SSH key. 
Each key is different hence GitHub does not allow to have the same key uploaded to multiple accounts.

As soon as you tried to push anything to your repository on GitHub you receive an error message:

```shell
$ git checkout -b test
Switched to a new branch 'test'
[...]
$ git push test
fatal: 'test' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

In my case that message is 100% correct beacause `git` used my `default` SSH-key which does not have access to that specific repo.
Since we know where is the problem let's make `git` use the proper SSH key.

**Repository already cloned**

Let's check current git remotes:
```shell
$ git remote -v                                                 
origin  git@github.com:madblackbox/madblackbox.github.io.git (fetch)
origin  git@github.com:madblackbox/madblackbox.github.io.git (push)
```

***Configuring ssh config***

Since we use `ssh` method for connecting with GitHub we need to make changes to `ssh` itself, let's edit `~/.ssh/config` [^1]

```conf
# ~/.ssh/config

Host github-default
    HostName github.com
    User git

Host github-mbbx # we will use that name for git remote
    Hostname github.com
    User git
    IdentityFile  ~/.ssh/id_mbbx_25519 # path to your non-default user's SSH private key
```

***Configuring remotes***

Let's add a new remote, please pay attention that here we use **github-mbbx** that we put in `~/.ssh/config` earlier:

```shell
$ git remote add origin-mbbx git@github-mbbx:madblackbox/madblackbox.github.io.git
```

let's confirm that we successfully added new remote:
```shell
$ git remote -v                       
origin  git@github-mbbx:madblackbox/madblackbox.github.io.git (fetch)
origin  git@github-mbbx:madblackbox/madblackbox.github.io.git (push)
origin-mbbx git@github-mbbx:madblackbox/madblackbox.github.io.git (fetch)
origin-mbbx git@github-mbbx:madblackbox/madblackbox.github.io.git (push)
```

let's check whether do we have access now:

```shell
$ git push origin-mbbx test
```

if there are no error message we can replace old **origin** with the new one:

```shell
$ git remote remove origin
$ git remote rename origin-mbbx origin
```

now let's check current remote configuration, result should be similar to:

```shell
$ git remote -v                       
origin  git@github-mbbx:madblackbox/madblackbox.github.io.git (fetch)
origin  git@github-mbbx:madblackbox/madblackbox.github.io.git (push)
```

and if everything went ok you should be able to use that repo as any other without keeping in mind that this one is conneced with your non-default account.

{{< notice warning >}} 
You propably should also configure git itself!
```shell
$ git config --local user.email you-alternative@email.com
$ git config --local user.name YouAlternative UserName
[etc.]
```
{{< /notice >}}

[^1]: https://www.ssh.com/academy/ssh/config