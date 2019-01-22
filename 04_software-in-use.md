---
title: Software in use
permalink: /software-in-use
---

---

<!-- TOC -->

- [Software in use](#software-in-use)
    - [Luks](#luks)
    - [Tomb](#tomb)
    - [GPG](#gpg)
    - [Pass](#pass)
    - [Master Password (mpw)](#master-password-mpw)

<!-- /TOC -->

---

# Software in use

R.I.S.K.S. relies on these software, all open source:

| TOOL          | FEATURES                                                        |
| ------------- | --------------------------------------------------------------- |
| luks          | provides encrypted filesystem                                   |
| tomb          | an encryption-manager using LUKS                                |
| gpg           | *unequivocally identifies a user*, encrypts files and messages  |
| pass          | common Linux script for managing credentials and password-files |
| mpw           | manages multiple credentials using just one password            |
| gpg-split     | a technique used in Qubes Os to protect GPG                     |
| steghide      | a technique for embedding secret text inside a digital picture  |
| risks-scripts | set of scripts to automate and simplify the workflow            |

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
