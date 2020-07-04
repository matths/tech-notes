---
title: Locales
tags: shell, linux, locales, debian
---

# Locales

Recently I stumbled, espacially in context of unix commands using perl, over these reoccurring error messages similar or equal to this.

```plain
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_TERMINAL = "iTerm2",
	LC_TERMINAL_VERSION = "3.3.12",
	LANG = "de_DE.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
```
To overcome these messages in general, I eventually run these commands, the latter one is interactive, where I picked en_US.UTF-8 as default well.

```sh
apt-get install -y locales
locale-gen en_US.UTF-8
dpkg-reconfigure locales
```
