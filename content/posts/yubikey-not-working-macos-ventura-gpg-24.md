---
title: "Yubikey fails on macOS Ventura with \"sign_and_send_pubkey: signing failed\""
date: 2023-02-24T20:37:50-05:00
description: >
  Fixing the "sign_and_send_pubkey: signing failed" error on macOS with a Yubikey
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

## Other causes

I have also found that this error can be caused by having an incorrect `pinentry-program` in `~/.gnupg/gpg-agent.conf`. In my case, I had this in my config:

```
pinentry-program /opt/homebrew/bin/pinentry-mac
```

But somewhere along the line, `pinentry-mac` was removed. You can verify if it does/doesn't exist by running `/opt/homebrew/bin/pinentry-mac` in your shell. I ran `brew install pinentry-mac` and it resolved the issue for me.
