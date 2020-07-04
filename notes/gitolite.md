---
title: Gitolite
tags: gitolite, acl, access control, git, server, linux, hooks, perl
---

# Gitolite

## General

see https://gitolite.com/ for more informations.

### Motivation to use gitolite

* it offers the access control layer to have your repositories 
accessible to a defined set of users without creating a whole unix/linux user account for them, but by adding their public SSH key
* thus repositories are found at the usual git@example.com:repo url
* it has only few dependencies and is not bloated, easy to understand and setup
* it offers hooks to trigger custom commands e.g. when pushing to a master branch, which is nice for deployments

### Trivia

* its written mostly in Perl
* its used by the Linux kernel developers

## Setup

Installation is quite similar to my notes about a plain [Git server](git-server.md).

### Install Git

```sh
sudo apt update
sudo apt install git
```

### Create user

Create the user git, who can not log in using a password but using a SSH Key and switch into the user using su.
But here we specify `/var/git`as home folder.

```sh
adduser --disabled-password --home /var/git git
su git
```

### Install Gitolite

As user git create all neccessary folders.

And copy the public SSH key into a file called `admin.pub`. The owner of the associated private key will become the gitolite administrator.

```sh
cd
mkdir -m 0700 ~/.ssh ~/bin
vim admin.pub
```


Download the latest gitolite version using git, what else. ;)

```sh
git clone git://github.com/sitaramc/gitolite
```

Link binaries and use gitolite setup script to apply admin.pub. It gets copied into `/var/git/.ssh/authorized_keys`. Thus we can remove it afterwards.

```sh
gitolite/install -ln
bin/gitolite setup -pk admin.pub
rm admin.pub
```

You're done. As owner of the matching private key for `admin.pub`you should be able to reach gitolite instead of logging into a shell.

```sh
ssh -T git@example.com
```


## Configuration

As admin you can configure further users and a lot more. You need to checkout the `gitolite-admin`repository, where you have read and write access to.

```sh
git clone git@example.com:gitolite-admin
```

To add a users public SSH key create a `./keydir/user.pub` file.
Giving a user the right to access a repository or create a new repository automagically, you add another section into `./conf/gitolite.conf`

```plain
repo gitolite-admin
    RW+     =   admin

repo testing
    RW+     =   @all

repo new-repo
    RW+     =   user
```

Don't forget to add your new files, stage your changes and commit/push. Everything else is done by gitolite. So try to access new-repo as user.

```sh
git clone git@example.com:new-repo
```

## Hooks

Hooks are a great way to automize things triggered by e.g. `git push`

### Enable hooks

To enable hooks uncomment the folling lines in `/var/git/.gitolite.rc`

```plain
LOCAL_CODE                =>  "$ENV{HOME}/local",
'repo-specific-hooks'
```
### Write hooks

As admin create the neccessary folders and our example `show-hook` and make it executable.

```sh
su git
mkdir -p local/hooks/repo-specific
vim local/hooks/repo-specific/show-hook
chmod +x local/hooks/repo-specific/show-hook
```

Here's the content of the hook. Although there are environment variables related to the repository available, we can get a bunch more.

```plain
GL_REF=$1
GL_COMMIT=$(git rev-parse $GL_REF)
GL_BRANCH=$(git rev-parse --abbrev-ref $GL_REF)
GL_CLIENT_IP=${SSH_CLIENT%% *}
GL_MESSAGE=$(git log --pretty=format:%s -n 1 $GL_COMMIT)
GL_AUTHOR_NAME=$(git log --pretty=format:%an -n 1 $GL_COMMIT)
GL_AUTHOR_EMAIL=$(git log --pretty=format:%ae -n 1 $GL_COMMIT)
GL_DATE=$(git log --pretty=format:%aD -n 1 $GL_COMMIT)
GL_TIMESTAMP=$(git log --pretty=format:%at -n 1 $GL_COMMIT)

echo user: $GL_USER
echo client ip: $GL_CLIENT_IP
echo ref: $GL_REF
echo branch: $GL_BRANCH
echo repository: $GL_REPO
echo commit: $GL_COMMIT
echo message: $GL_MESSAGE
echo author: $GL_AUTHOR_NAME, $GL_AUTHOR_EMAIL
echo date: $GL_DATE
```

### Configure hooks

As admin add you repository specific post-update hook into `./conf/gitolite.conf`

```plain
repo gitolite-admin
    RW+     =   admin

repo testing
    RW+     =   @all

repo new-repo
	option hook.post-update = show-hook
    RW+     =   admin
    RW+     =   user
```


### Automatic deployment hook

You might trigger a deployment script right in you post update hook. But be careful as you have to think about, which user should run the needed commands.

Instead of the `show-hook` imagine an `auto-deploy` hook like this.

```plain
GL_REF=$1
GL_BRANCH=$(git rev-parse --abbrev-ref $GL_REF)
if [ "$GL_BRANCH" = "master" ]
then
    echo "automatic deployment for master branch"
    if [ "$GL_REPO" = "web" ]
    then
        sudo -u web /home/web/deploy.sh
    fi
fi
```

But for the git / gitolite user to be able to run the `/home/web/deploy.sh` script as if it was the `web` user, you would need to adapt your sudoers configuration.

```sh
cat > /etc/sudoers.d/git << EOF
git     ALL=(web:web) NOPASSWD:/home/web/deploy.sh
EOF
```

Imagine the user web needs to restart the `web.service` it would need sudo rights as well.

```sh
cat > /etc/sudoers.d/web << EOF
web     ALL= NOPASSWD: /bin/systemctl restart web.service
web     ALL= NOPASSWD: /bin/systemctl stop web.service
web     ALL= NOPASSWD: /bin/systemctl start web.service
EOF
```

**related fun fact:** the user web wouldn't need a sudoers configuration to run `/bin/systemctl status web.service`. status ist accessible anyways.

But you might find more of my notes about [sudo and su](sudo.md) here.
