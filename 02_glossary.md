---
title: Glossary
permalink: /glossary
---


# Glossary

For me good part of the difficulty of studying security comes from terminology. There is always that moment in which I confuse terms and I end up misunderstanding or being unclear.

Even while writing R.I.S.K.S. I often find myself using terms as synonymous when they are not, like password and passphrase and this is confusing.

So I maintain a glossary of terms and I stick to these terms as strictly as I can.


| TERM                        | MEANING                                                                                                             |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| encryption                  | a technique used to hide something from anyone who's not supposed to have access to it.                             |
| secret                      | anything that none else than you should know. It could be any file or directory of files.                           |
| secret-file                 | a LUKS encrypted filesystem embedded in a file where secrets are stored.                                            |
| credentials                 | a username and a password.                                                                                          |
| username                    | a mnemonic string of text identifying an identity/person/account.                                                   |
| password                    | a mnemonic string of text and used to encrypt/decrypt something. Can be used in combination with username.          |
| key                         | anything used to encrypt/decrypt something or to identify something or both.                                        |
| passphrase                  | a long mnemonic string of text and used to encrypt/decrypt something. Can be used in combination with a key.        |
| key-file                    | a unique file used to encrypt/decrypt a secret-file. Can be used in combination with a passphrase.                  |
| pass                        | a password manager software. It stores credentials and secrets in GPG encrypted files.                              |
| pass-file                   | a text file GPG encrypted containing (at least) relevant information related to credentials or passphrases.         |
| mpw                         | a password generator based on a single master password.                                                             |
| mpw-file                    | a text file GPG encrypted containing part of the inputs for `mpw`, a password generator. It contains no secrets.    |
| coffin-file                 | a secret-file which contains GPG-files and password-files for a specific identity.                                  |
| identity-coffin             | a way to indicate two coffin-files, one containing GPG-files and one containing pass-files.                         |
| tomb-file                   | a secret-file which contains any kind of secret.                                                                    |
| graveyard                   | a directory where secret-files are laid.                                                                            |
| GPG: **key-pair**           | as set made of a private and a public key mathematically linked one to another                                      |
| GPG: **key-flag**           | a key-pair feature and means "this key-pair is used for this activity/activities"                                   |
| GPG: **primary key-pair**   | the first key-pair generated when you are creating an new GPG identity                                              |
| GPG: **subkey-pair**        | a key-pair depending on a primary key-pair                                                                          |
| GPG: **key-ring**           | a collection of key-pairs (belonging to you) and public keys (belonging to others)                                  |
| GPG: **revoke certificate** | a special file, generated when at the primary key-pair creation time used to revoke the validity of the key-par   |
| GPG: **trust ring**         | core concept of the _web of trust_. It's the set of signatures applied by others to your public signing key         |
