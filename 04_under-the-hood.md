---
title: Under the hood
permalink: /under-the-hood
---

---
<!-- TOC -->

- [Software in use](#software-in-use)
    - [Luks](#luks)
    - [Tomb](#tomb)
    - [GPG](#gpg)
    - [Pass](#pass)
    - [Master Password (mpw)](#master-password-mpw)
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

# Software in use

R.I.S.K.S. relies on these software, all open source:

| TOOL          | FEATURES                                                        |
| ------------- | --------------------------------------------------------------- |
| luks          | provides encrypted filesystem                                   |
| tomb          | an encryption-manager using LUKS                                |
| gpg           | *unequivocally identifies a user*, encrypts files and messages  |
| risks-scripts | set of scripts to automate and simplify the workflow            |
| pass          | common Linux script for managing credentials and password-files |
| mpw           | manages multiple credentials using just one password            |
| gpg-split     | a technique used in Qubes Os to protect GPG                     |
| steghide      | a technique for embedding secret text inside a digital picture  |

## Luks

[LUKS](https://gitlab.com/cryptsetup/cryptsetup/blob/master/README.md) is the standard for Linux hard disk encryption.

> By providing a standard on-disk-format, it does not only facilitate compatibility among distributions, but also provides secure management of multiple user passwords.
> LUKS stores all necessary setup information in the partition header, enabling to transport or migrate data seamlessly.

## Tomb

[Tomb](https://www.dyne.org/software/tomb/) generates encrypted storage folders to be opened and closed using their associated key-files, which are also protected with a password chosen by the user.

> A tomb-file is like a locked folder that can be safely transported and hidden in a filesystem; its keys can be kept separate, for instance keeping the tomb file on your computer harddisk and the key files on a USB stick.

## GPG

[GPG](http://www.gnupg.org) is a cross-platform tool used for:

1. identifying someone or something
2. encrypting/decrypting files

GPG is a generic purpose encryption tool with a long history and it's used by a variety of other softwares like file managers, email clients, chat software and password managers.

## Pass

Since long time _Linux_ relies on _[pass](https://www.passwordstore.org/)_ for password management.

Although mostly used for storing websites credentials, `pass` stores any kind of private information in GPG encrypted text files.

`pass` awesomeness sits in its simplicity: it's a bash script supporting extensions, it uses no database, has no hierarchical constraints, has native git support and uses GPG for encrypting/decrypting data.

On top of this it features a reliable history and a supporting community.

This is an example of a decrypted pass-file.

    > pass website--com

    1Pe4Hpz-sc637tsaKk
    ---
    username: user@youremail.com

The first line is the password and below it any kind of information can be stored.

This content is stored in _.gpg_ file inside the directory `~/.password-store` which looks like this:

    /home/user/.password-store/
    ├── website--com.gpg
    ├── addons.example.com.gpg
    ├── anyname-any_format.IwantHere.gpg

`pass` supports git and this means that the content of `~/.password-store` can be versioned and distributed on different machines.

Considering the protection provided by GPG, pass-files can be synchronized on the cloud. This activity raises some concerns but `mpw` can be used to add an additional layer of security and make the distribution much safer.

Additionally `pass` has:

* [extensions](https://www.passwordstore.org/#extensions)
* [browser plugins](https://www.passwordstore.org/#other), applications and GUIs for several operating systems
* [migration tools](https://www.passwordstore.org/#migration) to import credentials from several widely used password managers

## Master Password (mpw)

[Master password](https://masterpassword.app/) (aka `mpw`) is a single-password password-manager. Its core is written in C.

The key feature of `mpw` is an algorithm which takes some parameters (the url of a website and the username) and a **master password** as input and creates a unique password **which is stored nowhere**. The master password is stored in human memory. The algorithm is idempotent meaning that whenever it's fed with the same inputs it returns the same password. A slight change in the input generates a different output. The output password is then used for websites credentials or other purposes.

Additionally `mpw` has:

* browser plugins
* GUIs for several operating systems
* android app


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
