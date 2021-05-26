---
title: "\"Unable to decrypt password data\" of Windows EC2 instance"
date: 2021-05-26T17:56:17-05:00
draft: false
tags: [windows, aws]
---

I recently rotated my SSH keypair, and everything was working great until I booted up a Windows EC2 instance and
went to get the password:

```
$ aws ec2 get-password-data --instance-id i-0dd1f5bbeefa4625d --priv-launch-key ~/.ssh/key.pem

Unable to decrypt password data using provided private key file.
```

I triple checked that I booted this instance using my new keypair, and confirmed that the keypair wasn't encrypted,
but I still wasn't able to retrieve the password. However, when looking at the key file, one thing stood out to me:

```
$ head -1 ~/.ssh/key.pem
-----BEGIN OPENSSH PRIVATE KEY-----
```

It turns out that more recent versions of `ssh-keygen` on macOS generate SSH keys in this format by default, and awscli
doesn't really play well with them... at least when it comes to decrypting the Windows Administrator passwords. After some
further Googling, I was able to find a way to [convert my new SSH key to RSA format](https://serverfault.com/a/950686):

```
$ ssh-keygen -p -m PEM -f ~/.ssh/key.pem
```

Now, my key is in RSA format and I can finally retrieve the Windows password:

```
$ aws ec2 get-password-data --instance-id i-0dd1f5bbeefa4625d --priv-launch-key ~/.ssh/key.pem

------------------------------------------------------------------------------------------
|                                     GetPasswordData                                    |
+---------------------+------------------------------------+-----------------------------+
|     InstanceId      |           PasswordData             |          Timestamp          |
+---------------------+------------------------------------+-----------------------------+
|  i-0dd1f5bbeefa4625d|  ORLYuwae6deiSh3guu1ievohn7aimon8  |  2021-05-26T22:41:08+00:00  |
+---------------------+------------------------------------+-----------------------------+
```
