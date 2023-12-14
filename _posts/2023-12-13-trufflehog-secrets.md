---
title: "Secret Scanning with Trufflehog"
date: 2023-12-13 15:00:00 +0100
toc: false
toc_sticky: false
categories:
  - GitHub
  - Azure
tags:
  - Blog
  - Security
excerpt: Using Trufflehog to scan your repos
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
A while a go I looked into using [GitLeaks]({{ site.baseurl }}{% link _posts/2022-02-16-secret-scanning.md %}) to scan for secrets in your repo's.

Well, now I'm on a different project and its uses another tool! Here we will look at [Trufflehog](https://github.com/trufflesecurity/trufflehog)

What's the difference I hear you ask...Well, not much!

The biggest difference that I can see is that you don't need to create a separate config file to scan and exclude values, Trufflehog has this built in with the option to ignore values.

## Installation

Very simple in this case:

For Homebrew on the Mac, we have:
```bash
brew install trufflesecurity/trufflehog/trufflehog
```

Anything else, grab the binary from the [releases page](https://github.com/trufflesecurity/trufflehog/releases)

Don't want to install anything and would rather run it as a docker container? Sure no problem, there's an image for that!

## Use

You can scan a repo:

```bash
-> trufflehog git https://github.com/pritpalp/pritpalp.github.io
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2023-12-13T16:56:26Z	info-0	trufflehog	finished scanning	{"chunks": 65, "bytes": 123944, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "1.241692334s"}
```

Want to run it locally on your repo rather than on GitHub? Sure:

```bash
-> trufflehog git file://opt/Projects/pritpalp.github.io
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2023-12-13T16:57:46Z	info-0	trufflehog	finished scanning	{"chunks": 0, "bytes": 0, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "60.536875ms"}
```

There are plenty of other ways to use it, eg scanning s3 buckets or docker images. Just have a read through the readme in the repo.

For our use, we put in a pre-push hook to scan the repo before pushing back to the repo.

This is what happens every time we push:

```bash
Pre-push hook: RUNNING truffleHog to test for leaked secrets.
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2023-12-14T17:11:23Z	info-0	trufflehog	finished scanning	{"chunks": 3, "bytes": 2964, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "67.336583ms"}
Pre-push hook: OK proceeding with push
```

Trufflehog runs, scans the local repo before allowing the push. If secrets are found the push fails and you can remove any secrets highlighted - or create an exception for them.

Nearly forgot, this is an example of how secrets are reported back to you:

```bash
Found verified result 🐷🔑
Detector Type: Infura
Decoder Type: PLAIN
Raw result: 0b88ff4d03a647b8a4649e9bfdf6644f
Commit: 9c88febac7fc0257929916020b0c019aeceb0e61
Email: Björn Kimminich <bjoern.kimminich@owasp.org>
File: routes/nftMint.ts
Line: 14
Repository: https://github.com/juice-shop/juice-shop
Timestamp: 2023-09-21 22:05:29 +0000

Found unverified result 🐷🔑❓
Detector Type: PrivateKey
Decoder Type: PLAIN
Raw result: -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA68SlXtdvW8fqYzUs7XPWWdzbgdZUF5rr0TO5Za/tfv7mO981
fU0dnSPIfkPx5W1inceOkOj4S6f2SwuliuFfp4bAREmI3weZSFckyaYgyE+mJdPE
VzQ3riYDsi2iR/BEpB40aho/jqBTQu8jIwDGGwlxF0XedWDSLn3zfM2Ga0LOd44A
/psCNia91cPT+JTrGyTg/vVpESgJqlK/MmZPNclnymkoyiGQiBcUpKexeKLq3UyW
lkiQdByAwxxCiyW4UjPzwYfMTz9vPCPzqVH97/y4KYVuc5dmOaDH1GI+ZtRFEh5S
sFdntMoQ/1pEejwoTZsTStHMzPBFS9gGUx+QdwIDAQABAoIBAQDXRuJ9HBepZXym
g4p3cvr5aMnh3xM/zoyepC0YJbCk8hjF+IT92ak9r8vFR0Mb62pNiUarGJ6HbmFc
mtDYY/uUm1z5vW9Fvsl+nVuQ6KksXlXmWlwACEUDNeDQFA0GxOPYO2A+taLtF4WP
K44Yyv8Y9uEVkA2VfgfMveLTRVMEoeXzU/VHfOvgnf6y9y77V4juLREkjwkbGTnI
gpAOsmNDPdy+43mVp63QvVaSFa3gDly3Ak84Ta9P7TSTjGwtS3qnJwODu9h/2ty7
/aBoGjw9G/sP1DsCMCJgx4rrrpW1RlPp/bOeKZQqu7rHjUX9f9oRSpW0bGjwq+gP
SGiJj42hAoGBAPk6uOEOfQzN8imdVslTiXyvIn+FPXzYwkCq5Lkv2o8udGLEnTKl
7Nk0G4pfGXrttOmzNpndo+4jRXe0ceMssjin6dkl6MufD2BFFgls9393oZJ4CECP
au4EPKXMe/xkzz99CilgQ3r1v0DN/hBl41cd0xi8wJjwFBTSob7YkGTnAoGBAPIs
TpZnC27s+3mPYHow/1Ca/7fuZVzseXXPEMK1EKjHfsXieOkcLeQEwNg3EbKaBYAV
JjRBdD0XYUFpmDAbVcVk2bAgDKfNGNBEa6YLqJQ2uNODgOOUodakPjxCIjzxflhd
V6CN3lQcK6hWdXMsJWocoDF9Hxz0JEQsQd589XXxAoGAd+LEbhYPFyq180ipJ50U
hLKmMJtCMZz/DCZocaBQTRG2kJAtYeCo5u6G1O/cDOLtZIF9oVQZeALldqiJJBMr
A8/Z0EfJDLHNrqxs5knRYDKGuTMeHRggArBtEAAmIAnKG6slSTPyIeK2hhDQxsiM
LCq/kaWyK59IuZ98iJYaFz8CgYA0LSehcAIenCBySFnY+cWIcFy4HDzqkGh64WoT
CT/VnWXK7MhwMQoSHpQOAY9mk5irx+K7T37jyq3Bkiaf9sO8C8Z7E+ymGqJF/PfU
hp6DkGax65tRbSyROkHOadFGoCFAmJvQk8BbDta5JieX8OL+wbwh7XtOmatWpNJs
RS/9gQKBgQDBbcmigS5ROXYTE8XR3j5iXKETK3YjTksTPEtgRjIxfE44dmB8291S
XSm89IbAsAqzbAfIKCAURV8wJ0HvD9UiimdUxn6yqX6q6UGvBuInvCI6cZ4U6PWM
b0Si6/hMiINTpqG8RT9rRj8kCF383y8AGMQle7wbUJzxOJbKt43Gfw==
-----END RSA PRIVATE KEY-----
Commit: 48174b3e4188811db7b6f128cbf11da7fd9cde82
Email: Björn Kimminich <bjoern.kimminich@owasp.org>
File: vagrant/.vagrant/machines/default/virtualbox/private_key
Line: 1
Repository: https://github.com/juice-shop/juice-shop
Timestamp: 2017-03-03 01:12:32 +0000

2023-12-14T17:21:03Z	info-0	trufflehog	finished scanning	{"chunks": 64108, "bytes": 86247764, "verified_secrets": 5, "unverified_secrets": 5, "scan_duration": "25.709870958s"}
```

These examples were found in the [JuiceShop](https://owasp.org/www-project-juice-shop/) project from OWASP (and that's not a complete list of the result).