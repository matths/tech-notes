---
title: sudo and su
tags: access control, server, linux, user, root, super user, substitute user
---

# `sudo` and `su`

While configuring my webserver with different users and permissions, I either use the commands `sudo` and `su` a lot or need to give users permission to do so, when automizing stuff.

## some trivia

A nice explanation what these commands do is the replace **su** with **substitue user** instead of **super user**, don't you agree?
* su = substitue user
* sudo = substitue user; do

The man pages state these instead:
* su - change user ID or become superuser
* sudo, sudoedit — execute a command as another user


## `su`

While installation the user is changed often from root to another user by using `su user`.

In this case, the users shell is opened and run in context of user:groupOfUser.


## `sudo`

Very often this command is used to execute stuff as root by entering your own, not roots password, e.g.

```sh
sudo apt-get install package
```

For this to work, a corresponding configuration is needed, which happens in the /etc/sudoers file, we come to talk about later.

So while running a command with `sudo command` it will ask for your password, while running the command with `su -c command` it is going to ask you for roots password.

You might come in a situation where there's no sudo. In that case, install it using apt-get (when being root ;) ).

```sh
apt-get install sudo
```

## `sudo su -`

there's this command which is easier to look up the answer [here](https://askubuntu.com/a/376386) than to describe another time.

## `/etc/sudoers`

In this file, there is the configuration of who can run commands as who from where and with or without password.

The format follows this construction.

```plain
[WHO]   [AT WHICH HOST]=([AS USER]:[AS GROUP]) [[OPTINAL OPTIONS]][IS ALLOWED TO EXECUTE]
```

So, for example the following case allow the user git to execute deploy.sh as if it was run by web:web without asking git for a gits password.

```plain
git     ALL=(web:web) NOPASSWD:/home/web/deploy.sh
```

It is good practive to use `visudo` to edit the /etc/sudoers file, and it doesn't mean, that it uses `vi` but you default editor, maybe `nano`. :) 

## `/etc/sudoers.d/[files]`

instead of adding all sudo rules into a single sudoers file, one can instead put semantically related stuff in separate files in `/etc/sudoers`, e.g. with a command like

```sh
cat > /etc/sudoers.d/web << EOF
web     ALL= NOPASSWD: /bin/systemctl restart web.service
web     ALL= NOPASSWD: /bin/systemctl stop web.service
web     ALL= NOPASSWD: /bin/systemctl start web.service
EOF
```
