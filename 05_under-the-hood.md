---
title: Under the hood
permalink: /under-the-hood
---

---
<!-- TOC -->

- [R.I.S.K.S. architecture](#risks-architecture)
    - [Why an sdcard?](#why-an-sdcard)
    - [Vault and secrets](#vault-and-secrets)
    - [Sdcard and keys](#sdcard-and-keys)
    - [Secrets: coffins and tombs](#secrets-coffins-and-tombs)
    - [Credentials and passphrases](#credentials-and-passphrases)
- [Real life example](#real-life-example)
    - [How much mnemonic effort is that?](#how-much-mnemonic-effort-is-that)

<!-- /TOC -->
---

# R.I.S.K.S. architecture

Implementing R.I.S.K.S. provides:

* one qube for managing secrets, called _vault_
* one qube for storing files, called _joe-fsq_
* one qube for software development or writing, called _joe-devq_
* a mechanism, based on sdcard, for managing key-files
* a disaster recovery procedure
* a backup procedure

_joe-fsq_ and _joe-devq_ are the tool-set for a virtual identity called Joe.

With R.I.S.K.S. GPG, SSH and pass-files are encrypted and isolated but pretty easy to retrieve and use.

The attempt is to obtain a reasonable workflow for for daily use.

From the operational standpoint I implement the R.I.S.K.S. workflow with two bash scripts `risks` and `risq` provided in the [RISKS-SCRIPTS repository](https://github.com/19hundreds/risks-scripts)

## Why an sdcard?

I choose a [Secure Digital Card](https://en.wikipedia.org/wiki/Secure_Digital) for storing key-files because:

* it's a cheap and common device
* sdcard readers are available out of the box in a good number of laptops
* USB sdcard readers are common and cheap
* it's small factor, light and comfortable
* in _microsd_ format, it can be easily swallowed (in case of need!)
* forgotten inside the slot, it doesn't break when the pc is pushed inside the hand bag
* it's a raw container with few electronic parts: robust and hard to tamper with
* it can be replaced with a common usb pen-drive or usb hard-drive
* easy to clone
* a clone can be easily hidden in some secure place
* it can be hidden among other photo-camera sdcard and can pass checkpoints undetected
* it can be easily sent via regular mail
* it can be used read-only mode but it can be written any time

## Vault and secrets

_vault_ which handles secrets, passwords and GPG. Nothing else.

It's a minimal Debian with no network connection.

_vault_ hosts an its filesystem a directory named _graveyard_ `(~/.graveyard)` which contains _secret-files_ where data is encrypted and stored.

## Sdcard and keys

I use a partitioned sdcard to store the key-files required to open the _secret-files_ laying in the _graveyard_.

I connect the sdcard to the _vault_ via a script (part of the _risks-script_ repository).

The partition containing the key-files, called _hush-partition_, is encrypted with LUKS and protected by a passphrase (no binary-key).

## Secrets: coffins and tombs

I classify the _secret-files_ contained in the _graveyard_ as _coffin-files_ and _tomb-files_.

Both are LUKS filesystem with the same security level. The two types differ in:

* the management: _tomb-files_ are managed by [Tomb](https://www.dyne.org/software/tomb), a very comfortable wrapper for `cryptsetup`, while _coffin-file_ are managed by `cryptsetup`
* the passphrase: _tomb-files_ require a key-file plus a mnemonic passphrase to be opened. _Coffin-files_ need just a key-file.

In the _coffin-file_ I store just the GPG-files of the identity (Joe). One _coffin-file_ for a each identity.

The _coffin-file_ uses a binary key-file without passphrase (passphrase-less) stored inside the _hush-partition_. Opening a coffin-file requires no mnemonic effort and it's done with a passphrase-less script (`risq`). When a _coffin-file_ is open, _vault_ is able to provide GPG functionality to any other qube.

In _tomb-files_ I usually store generic secret files and directories of files. Each one is protected by a different GPG encrypted binary key stored in the _hush-partition_.
Opening a _tomb-file_ requires both the binary key and the GPG passphrase. This means that no _tomb-file_ can be opened until a _coffin-file_ is opened in the _vault_ qube.

Basically the _vault_ qube is configured with the GPG-split technique (and other split-techniques) which lets other qubes to access the _vault_ GPG agent after the Qubes OS authorization step is cleared (it asks for the GPG-passphrase).

Both the _coffin-file_ and the _tomb-file_ can be managed inside the _vault_ qube with `risks`.

Inside any other authorized _qube_ I can manage just _tomb-files_ with `risq`.

Using passphrase-less _coffin-files_ means that who can open the _hush-partition_ can also access my _coffins_. This is indeed a weak point not so weak as it seems because GPG is also protected by its passphrase. This is why GPG passphrases must be strong and well done.

## Credentials and passphrases

I manage the _pass-files_ for each identity with `pass`. In most of the cases I use `mpw` seamlessly within `pass`.


This grants R.I.S.K.S. credentials management to match these features:

* each identity has its own _tomb-file_ for credentials
* I need just one master password for all the identities but each pass-tomb-file is protected by a different GPG
* password-files follow a lean scheme with very few rules. This allows to store any kind of text and offers maximum freedom

# Real life example

This is what I do when I need to work with identity _Joe_:

1.  I turn on my Qubes OS pc
2.  I insert the Qubes OS filesystem passphrase
3.  I login as Qubes user
4.  I load the _vault_ qube
5.  I plug in the sdcard in the socket
6.  I connect the sdcard to _vault_
7.  I insert the passphrase to decrypt the sdcard
8.  I decrypt Joe's _identity-coffin_ using a script (`risks`)
9.  I access Joe's pass-files using his GPG passphrase and, eventually, the `mpw` master password
10. I access any of Joe's _tomb-files_ using his GPG passphrase

## How much mnemonic effort is that?

Let's measure it:

* one passphrase right after boot (step 2)
* one password for the Qubes user (step 3)
* one passphrase for the sdcard (step 7)
* one GPG passphrase for each identity (step 9 and 10)
* one master password (for all the identities) for `mpw` (step 9 and 10)

=> if I have 1 identities I need to remember:

* 3 passphrases   (Qubes fs, sdcard, GPG)
* 2 passwords     (Qubes user, master password)

=> if I have 2 identities I need to remember:

* 4 passphrases   (Qubes fs, sdcard, 2 x GPG)
* 2 passwords     (Qubes user, master password)

=> if I have 3 identities I need to remember:

* 5 passphrases   (Qubes fs, sdcard, 3 x GPG)
* 2 passwords     (Qubes user, master password)

This is the formula:

> 2 passwords + (#number_of_identities + 2) passphrases
