---
title: GPG setup - part 3
permalink: gpg-setup-part3
---

---
<!-- TOC -->

- [tomb-file creation](#tomb-file-creation)
- [Cabal's mitigation](#cabals-mitigation)
- [Publish the public key](#publish-the-public-key)
- [GPG-SPLIT setup](#gpg-split-setup)
    - [joe-devq template configuration](#joe-devq-template-configuration)
    - [joe-devq configuration](#joe-devq-configuration)
    - [Qubes configuration](#qubes-configuration)
    - [Vault configuration](#vault-configuration)
    - [Test](#test)
    - [Skip Qubes authorization step](#skip-qubes-authorization-step)

<!-- /TOC -->
---


GPG for `${IDENTITY}` is functional with a full backup but it lacks Cabal's mitigation.
I need a tomb-file where I can move some files.


# tomb-file creation

I set these variables and I create the tomb-file following the standard procedure.

``` bash
IDENTITY="joe"
RECIPIENT="joe@foo.bar"
LABEL="cabal"
TOMBID="${IDENTITY}-${LABEL}"
SIZE=15
DIR="/tmp/cabal"
```

I open it with:

``` bash
risks open ${LABEL} ${IDENTITY}
```

which mounts the tomb file in `${HOME}/.tomb/${LABEL}`

# Cabal's mitigation

When the tomb-file is ready I move away from `~/.gnupg` the files that I want to protect:

* my primary key-pair private key
* my revoke certificate for the primary key-pair

Thanks to the "Keygrip" GPG option I can easily identify the file containing it:

``` bash
echo "$(gpg -K | grep Keygrip | head -n 1 | cut -d= -f 2).key"

50123401D7AA3CBF514F23D5FA98D3D485075B2E.key
```

``` bash
cp ~/.gnupg/private-keys-v1.d/50123401D7AA3CBF514F23D5FA98D3D485075B2E.key ${HOME}/.tomb/${LABEL}/
cp ~/.gnupg/openpgp-revocs.d/72052A26269F7552DDED36626596C5FA775EB9AB.rev ${HOME}/.tomb/${LABEL}/
```

To be sure that the primary private key and the recovery file are erased from the gpg coffin-file:

``` bash
wipe -f ~/.gnupg/private-keys-v1.d/50123401D7AA3CBF514F23D5FA98D3D485075B2E.key
wipe -f ~/.gnupg/openpgp-revocs.d/72052A26269F7552DDED36626596C5FA775EB9AB.rev
dd if=/dev/random of=~/.gnupg/delme
rm ~/.gnupg/delme
```

This is what I'm left with in my `~/.gnupg` directory:

``` bash
tree ~/.gnupg
.
├── gpg.conf
├── openpgp-revocs.d
├── private-keys-v1.d
│   ├── 3880DC542784CFC56B5773E50FF00289620071EA.key
│   ├── 4CB933841DCDF831B4CFFBC688B055BE472AC778.key
│   └── 4E37F90BFA7E9A823283F3976C79499A67E0EA03.key
├── pubring.gpg
├── pubring.kbx
├── pubring.kbx~
├── random_seed
├── tofu.db
└── trustdb.gpg
```

Let's see what's different. When I run:

``` bash
gpg -K

/home/user/.gnupg/pubring.kbx
-----------------------------
sec#  rsa4096/0x6596C5FA775EB9AB 2018-10-08 [SC]
    Key fingerprint = 7205 2A26 269F 7552 DDED  3662 6596 C5FA 775E B9AB
    Keygrip = 50123401D7AA3CBF514F23D5FA98D3D485075B2E
uid                   [ultimate] Joe Tester <joe@foo.bar>
sub   rsa4096/0xFCFB0E137879D519 2018-10-08 [E] [expires: 2019-12-31]
    Key fingerprint = 6598 5A20 E7D5 871E E0EF  5A52 FCFB 0E13 7879 D519
    Keygrip = 4CB933841DCDF831B4CFFBC688B055BE472AC778
sub   rsa4096/0xB7CB9817FF022A44 2018-10-08 [S] [expires: 2019-12-31]
    Key fingerprint = 5FF4 7D3F FD9F 95DB 9C96  1455 B7CB 9817 FF02 2A44
    Keygrip = 4E37F90BFA7E9A823283F3976C79499A67E0EA03
sub   rsa4096/0xB358AAABA8703BD8 2018-10-08 [A] [expires: 2019-12-31]
    Key fingerprint = DB1C 141B 9080 57E0 09AA  3F68 B358 AAAB A870 3BD8
    Keygrip = 3880DC542784CFC56B5773E50FF00289620071EA
```

I notice that the primary key-pair has a different label: **sec#** instead of **pub**

> excuse my ignorance: I don't know what _sec#_ stands for

# Publish the public key

If I want to build a "web of trust" for Joe I should publish his public key on some key server which will spread it to other key servers.

By publishing Joe's public key other people will be able to find it simply by looking for Joe's email address. They can add it to their key-ring and use it for sending him encrypted files or emails that can be decrypted **only** by Joe.

This is the least a journalist should have.

If _vault_ had network connection I could use this command for publishing the key:

``` bash
gpg --send-key ${fingerprint}
```

but _vault_ is network less so I need another way.

These are the steps:

* I copy my armored **public** key to another qube (like _joe-devq_) which has network accees
* I open a browser and go to any GPG key server URL (like https://pgp.surfnet.nl/)
* I open the armored key with a text editor and I copy its content in the clipboard
* I look for the upload button and I paste in the text-area the content of your public key (now it should be clear why the photo should be small)

Usually the key server, after a successful upload, replies with something like this:

> Key block added to key server database. New public keys added:
> 1 key(s) added successfully.

As a test I search for Joe's email inside the key server.

---

# GPG-SPLIT setup

Qubes OS offers a fantastic solution called [Qubes Split GPG](https://www.qubes-os.org/doc/split-gpg/)

> Split GPG implements a concept similar to having a smart card with your private GPG keys, except that the role of the _smart card_ plays another Qubes AppVM. This way one, not-so-trusted domain, e.g. the one where Thunderbird is running, can delegate all crypto operations, such as encryption/decryption and signing to another, more trusted, network-isolated, domain. This way the compromise of your domain where Thunderbird or another client app is running  arguably a not-so-unthinkable scenario  does not allow the attacker to automatically also steal all your keys.

I configure _vault_ and _joe-devq_ with GPG Split so that I can use GPG from _joe-devq_ without having any GPG configuration file in it.

## joe-devq template configuration

In the template of _joe-devq_ (_debian-10-dev_)

``` bash
sudo apt-get install qubes-gpg-split
ln -s $(which qubes-gpg-client) /usr/bin/gpg
sudo shutdown -P 0
```

## joe-devq configuration

In _joe-devq_ (not the template)

``` bash
echo 'export QUBES_GPG_DOMAIN="vault"' >> ~/.bashrc && sudo shutdown -P 0
```

## Qubes configuration

In dom0:

``` bash
echo '$anyvm $anyvm ask' > /etc/qubes-rpc/policy/qubes.Gpg
```

## Vault configuration

The package `qubes-gpg-split` should be already installed but in case it's not:

``` bash
sudo apt-get install qubes-gpg-split
```

Optionally I can change for how long _vault_ will grant access to GPG clients. The default is 5 minutes but I can make it an hour in this way.

> I tried adding this global var to ~/.profile or ~/.bashrc but it doesn't work for me. Not sure why.

``` bash
sudo sh -c 'echo "export QUBES_GPG_AUTOACCEPT=3600" >> /etc/profile.d/qubes-gpg.sh'
```

## Test

In _vault_:

``` bash
risks open gpg joe
```

I restart _joe-devq_ and I run:

``` bash
gpg -K
```

I immediately see the Qubes OS interface the authorization.

Cleared the Qubes authorization step, I see in the terminal the same outpet I see by running `gpg -K` in _vault_

The same happens if I run:

``` bash
qubes-gpg-client -K
```

## Skip Qubes authorization step

Optionally, if I trust _joe-devq_ enough, I can disable the Qubes authorization step in this way:

In dom0:

``` bash
echo 'joe-devq vault allow' > /etc/qubes-rpc/policy/qubes.Gpg
echo '$anyvm $anyvm ask' >> /etc/qubes-rpc/policy/qubes.Gpg
```
