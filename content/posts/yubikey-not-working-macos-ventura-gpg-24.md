---
title: "Yubikey fails on macOS Ventura with \"sign_and_send_pubkey: signing failed\""
date: 2023-02-24T20:37:50-05:00
description: Fixing the "sign_and_send_pubkey: signing failed" error on macOS with a Yubikey
draft: false
---

```
$ ssh hambone
sign_and_send_pubkey: signing failed for RSA "cardno:11_9e5_49a" from agent: agent refused operation
user@hambone: Permission denied (publickey).
```

```
echo "disable-ccid" >> ~/.gnupg/scdaemon.conf
```

After adding that line to `scdaemon.conf`, I just restarted the GPG daemon and I was finally able to use my Yubikey again.

```
pkill gpg-agent; gpg-agent --homedir $HOME/.gnupg --daemon
```