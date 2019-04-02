---
layout: home
---

In this step-by-step guide I describe my method to manage credentials and secrets also known as R.I.S.K.S.

## What to expect from implementing R.I.S.K.S.

Setting up RISKS and following its workflow results in:

* just four (4) passphares to memorize to manage all the secrets (doesn't matter how many identities or how many secrets)
* an encrypted sdcard configured for hosting private keys
* an isolated qube (virtual machine) configured for:
    * managing GPG configurations for multiple identities
    * managing credentials using [pass](https://www.passwordstore.org/)
    * managing SSH configurations for multiple identities
    * managing multiple encrypted filesystems (embedded in a file) for multiple identities
    * providing a secure access to GPG and SSH keys from any other qube
* a set of scripts to manage the workflow comfortably
* disaster recovery and backup procedures

## Real time example

This screencast shows in real time how:

* the sdcard encrypted partition is decrypted and mounted
* the identity "1900" is opened
* the identity "1900" is closed
* the sdcard encrypted is umounted

![RISKS common workflow](https://github.com/19hundreds/risks-workflow/raw/master/images/what_to_expect_from_risks.gif "RISKS common workflow")
