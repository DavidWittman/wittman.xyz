---
title: "Matching Elliptic Curve Private Keys and Certificates with openssl"
date: 2022-05-05T12:33:15-05:00
draft: false
tags: [openssl, cryptography]
---

There are many examples on the internet for matching up TLS certificates and private key files for RSA keys, but it's 2022 and Elliptic Curve (EC) keys are becoming a lot more prevalent. The method for matching the certificate and EC private key are similar to RSA: run an `openssl` command on each file to print out the public key and compare the result to ensure they match. If the values output by these commands are different, then the certificate was generated with a different private key.

Without further ado, here are the commands to run. In this example, the EC private key is named `key.pem` and the certificate is `cert.pem`.

## Certificate

``` bash
$ openssl x509 -noout -pubkey -in cert.pem | openssl md5
7aa9358d37f0f31267f62224723dd17a
```

## Private Key

``` bash
$ openssl ec -pubout -in key.pem 2>/dev/null | openssl md5
7aa9358d37f0f31267f62224723dd17a
```

These md5 values match, so the certificate and key are safe to use together.
