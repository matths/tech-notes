---
title: Git server
tags: git, server, linux, debian
---

# Git server

Setup of Git ist straight forward, there are dozens of manual an howtos out there, here's without the clutter and only few explanations.

**Because Git using SSH doesn't allow more sophisticated access control, see also my notes about [gitolite](gitolite.md).**

## Install package

```sh
sudo apt update
sudo apt install git
```

## Create user, bare repository and limited access

Create the user git, who can not log in using a password but using a SSH Key and switch into the user using su.

See my separate notes for more info about [sudo and su](sudo.md).

```sh
sudo adduser --disabled-password git
sudo su git
```

Now as git user add the allowed public SSH Keys to the `~/.ssh/authorized_keys` file. Thus you can give all possible users access to the repository by adding their public SSH key as authorized keys for the git user.

```sh
cd
mkdir ~/.ssh && chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
vim ~/.ssh/authorized_keys
exit
```

Leave the users shell again and its shell to the git-shell for security reason. You may need to add git-shell to `/etc/shells` first.

```sh
sudo echo $(which git-shell) >> /etc/shells
sudo chsh git -s $(which git-shell)
```

Now that we can't use the users shell anymore, we need sudo instead of su to init/create a bare repository. This has no working tree. It consists of the repository files instead, which are otherwise hidden in the .git subfolder of a working tree.

```sh
# sudo, because su not working, now that there's no usable shell anymore
cd /home/git
sudo git init --bare testing.git
sudo chown -R git.git testing.git/
```
You're done.

## Access repository using SSH

As owner of a corresponding private SSH key, you can now access the git server as you usually would at other git servers and services.

Either you clone the repositoy…

```sh
git clone git@[HOST]:testing
# short for, but same as
git clone git@[HOST]:/testing.git
```

…or you add it as a remote for an existing workingtree/repository.

```sh
git remote add origin git@[HOST]:/testing.git
```
