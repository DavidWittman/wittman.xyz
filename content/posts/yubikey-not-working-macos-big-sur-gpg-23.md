---
title: "Yubikey fails on macOS with \"Operation not supported by device\""
date: 2021-05-20T09:50:50-05:00
description: Fixing the "Operation not supported by device" error on GPG 2.3 with a Yubikey on macOS.
draft: false
---

After accidentally (!!) upgrading all of my homebrew packages, I noticed that I was no longer able to authenticate over SSH using my Yubikey. Upon further inspection, I noticed that GPG was updated to 2.3, and my Yubikey was no longer listed in `ssh-add -l`. Things weren't looking so hot when trying to interact with my Yubikey via GPG directly either:

```
$ gpg --card-status
gpg: selecting card failed: Operation not supported by device
gpg: OpenPGP card not available: Operation not supported by device
```

I tried downgrading to GPG 2.2 but that didn't seem to fix the issue either. Eventually, I [found the solution](https://dev.gnupg.org/T5409):

```
echo "disable-ccid" >> ~/.gnupg/scdaemon.conf
```

After adding that line to `scdaemon.conf`, I just restarted the GPG daemon and I was finally able to use my Yubikey again.

```
pkill gpg-agent; gpg-agent --homedir $HOME/.gnupg --daemon
```
