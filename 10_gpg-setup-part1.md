---
title: GPG setup - part 1
permalink: gpg-setup-part1
---

---

- [Ramdisk setup](#ramdisk-setup)
- [GPG configuration](#gpg-configuration)
- [Avoid information leaked](#avoid-information-leaked)
- [Options for keys listing](#options-for-keys-listing)
- [Displays the validity of the keys](#displays-the-validity-of-the-keys)
- [Limits preferred algorithms](#limits-preferred-algorithms)
- [Options for asymmetric encryption](#options-for-asymmetric-encryption)
- [Options for symmetric encryption](#options-for-symmetric-encryption)
- [GPG keys and subkeys creation](#gpg-keys-and-subkeys-creation)
    - [Primary key-pair creation](#primary-key-pair-creation)
    - [Primary key-pair fingerprint](#primary-key-pair-fingerprint)
    - [Encryption subkey-pair creation](#encryption-subkey-pair-creation)
    - [Signature subkey-pair creation](#signature-subkey-pair-creation)
    - [Tofu.db](#tofudb)
    - [Authentication subkey-pair creation](#authentication-subkey-pair-creation)
    - [Adding a photo to the public key](#adding-a-photo-to-the-public-key)
- [at the prompt write:](#at-the-prompt-write)
- [addphoto](#addphoto)
- [Then:](#then)
- [type photo's absolute path](#type-photos-absolute-path)
- [type your passphrase](#type-your-passphrase)
- [type "save"](#type-save)
- [type "quit"](#type-quit)
- [SSL key creation](#ssl-key-creation)
- [Coffin creation](#coffin-creation)
    - [Coffin key creation](#coffin-key-creation)
    - [Coffin creation](#coffin-creation-1)
    - [Coffin setup](#coffin-setup)
- [Recap for copy & paste](#recap-for-copy--paste)

---

For all the reasons listed in the previous chapter I decided to implement Cabal's mitigation in R.I.S.K.S. and to store the GPG primary key inside a tomb-file that can be opened only in the cases listed in the Debian wiki. Unfortunately the configuration process is quite long and delicate.

# Ramdisk setup

I use a [ramdisk](https://en.wikipedia.org/wiki/RAM_drive) to store the GPG and pass configuration directly in ram so that nothing is written on the hard drive and therefore nothing can be recovered by an attacker in possession of the hard drive.

Before creating the ramdisk I need to know how much free space I have in ram:

```bash
free -h
            total        used        free      shared  buff/cache   available
Mem:           465M        203M         36M        6.2M        225M        243M
Swap:            0B          0B          0B
```

I have 36 MB free. The size of the ramdisk must be smaller than the free space but 10MB are more than enough.

> Warning: my `~/.gnupg` directory is empty so I can safely remove it. Be sure that yours is empty too otherwise you end up deleting your valuable GPG account.

I create the ramdisk:

```bash
RAMDISK="${HOME}/.gnupg"
rm -fR ${RAMDISK}
mkdir ${RAMDISK}
sudo mount -t tmpfs -o size=10m ramdisk ${RAMDISK}
sudo chown ${USER} ${RAMDISK}
sudo chmod 0700 ${RAMDISK}
```

Test:

``` bash
mount | grep ramdisk
```

Output:

> ramdisk on /home/user/ramdisk type tmpfs (rw,relatime,size=10240k)
> ramdisk on /rw/home/user/ramdisk type tmpfs (rw,relatime,size=10240k)

Test: I create and delte a file in ramdisk

``` bash
touch ${RAMDISK}/delme
rm ${RAMDISK}/delme
```

> **Warning**:
> * the space used by the ramdisk won't be freed until reboot
> * whatever stored in the ~/ramdisk will be lost in case the ramdisk is umounted
> * whatever stored in the ~/ramdisk will be lost in case of vault is turned off or rebooted

# GPG configuration

I create the skeleton for GPG's home directory `~/.gnupg` so that it won't throw errors or warnings at the first run:

``` bash
dd if=/dev/random of=~/.gnupg/delme
rm ~/.gnupg/delme
mkdir -p ~/.gnupg/{openpgp-revocs.d,private-keys-v1.d}
```

This is the final content of the directory:

``` bash
tree ~/.gnupg
.gnupg
├── openpgp-revocs.d
└── private-keys-v1.d

2 directories, 0 files
```

I create a customized configuration file for GPG which improves a little GPG's common behavior.

``` bash
cat >${RAMDISK}/gpg.conf <<EOF
# Avoid information leaked
no-emit-version
no-comments
export-options export-minimal

# Options for keys listing
keyid-format 0xlong
with-fingerprint
with-keygrip
with-subkey-fingerprint

# Displays the validity of the keys
list-options show-uid-validity
verify-options show-uid-validity

# Limits preferred algorithms
personal-cipher-preferences AES256
personal-digest-preferences SHA512
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed

# Options for asymmetric encryption
cipher-algo AES256
digest-algo SHA512
cert-digest-algo SHA512
compress-algo ZLIB
disable-cipher-algo 3DES
weak-digest SHA1

# Options for symmetric encryption
s2k-cipher-algo AES256
s2k-digest-algo SHA512
s2k-mode 3
s2k-count 65011712
EOF
```

I find particularly useful the "listing options", especially the _"with-keygrip"_ and _"with-subkey-fingerprint"_ options which force GPG to show the name of the file where each key is stored.

# GPG keys and subkeys creation

## Primary key-pair creation

I create a batch file for GPG so that the keys can be created without interaction.

> **Warning**:
> * modify these fields: Name-Real, Name-Email and **Passphrase**
> * make sure that each identity has its own different passphrase. The risk is to end up sending messages with the wrong identity and you will break the isolation.
> * the passphrase must be a diceware passphrase
> * write down on paper the passphrase

Eventually I can add a comment (maybe a password hint?) to the primary key by adding this line to the batch file:

``` batch
Name-Comment: -comment goes here-
```

``` bash
cat >${RAMDISK}/primary_key_unattended <<EOF
%echo Generating a standard key
Key-Type: RSA
Key-Usage: sign
Key-Length: 4096
Name-Real: Joe Tester
Name-Email: joe@foo.bar
Expire-Date: 0
Passphrase: knapp tiber fist lush hatred we're
%commit
%echo done
EOF
```

I'm ready to create the GPG primary key-pair:

``` bash
gpg --batch --gen-key ${RAMDISK}/primary_key_unattended
```

Output:

``` bash
gpg: keybox '/home/user/.gnupg/pubring.kbx' created
gpg: Generating a standard key
gpg: /home/user/.gnupg/trustdb.gpg: trustdb created
gpg: key 0x6596C5FA775EB9AB marked as ultimately trusted
gpg: revocation certificate stored as '/home/user/.gnupg/openpgp-revocs.d/72052A26269F7552DDED36626596C5FA775EB9AB.rev'
gpg: done
```

I wipe the `primary_key_unattended` file:

``` bash
wipe ${RAMDISK}/primary_key_unattended
```

I ask GPG to display the keys available in my key-ring at this stage:

``` bash
gpg -K

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
/home/user/.gnupg/pubring.kbx
-----------------------------
pub   rsa4096/0x6596C5FA775EB9AB 2018-10-08 [SC]
    Key fingerprint = 7205 2A26 269F 7552 DDED  3662 6596 C5FA 775E B9AB
    Keygrip = 50123401D7AA3CBF514F23D5FA98D3D485075B2E
uid                   [ultimate] Joe Tester <joe@foo.bar>
```

The signing key-pair (also called primary key-pair) has been created and this is the current content of `~/.gnupg`

``` bash
tree .gnupg

.gnupg
├── gpg.conf
├── openpgp-revocs.d
│   └── 72052A26269F7552DDED36626596C5FA775EB9AB.rev
├── private-keys-v1.d
│   └── 50123401D7AA3CBF514F23D5FA98D3D485075B2E.key
├── pubring.kbx
├── pubring.kbx~
└── trustdb.gpg

2 directories, 6 files
```

The super cool "keygrip" parameter shows that the binary file `50123401D7AA3CBF514F23D5FA98D3D485075B2E.key` contains the private key.

> Keygrip = 50123401D7AA3CBF514F23D5FA98D3D485075B2E

The binary file `72052A26269F7552DDED36626596C5FA775EB9AB.rev` is the revoke key for the primary key-pair which can be used when the primary key-pair is compromised or stolen.

Both these files are crucial.

The binary file `pubring.kbx` is the keyring and it contains the public key of the signing key-pair. In the future it will also contain:

*  all the public keys (belonging to others) that I import
*  all the public keys of the key-pairs that I create

The `pubring.kbx` file, as described [here](https://www.gnupg.org/documentation/manuals/gnupg/kbxutil.html), works like a database and can be queried for statistics:

``` bash
kbxutil --stats ~/.gnupg/pubring.kbx

Total number of blobs:  2
header:                 1
empty:                  0
openpgp:                1
x509:                   0
non flagged:            1
secret flagged:         0
ephemeral flagged:      0
```

To be clear: creating a new key-pair won't produce another 2 files (for a total of 4) but only 1 (for a total of 3): there will be 2 private key files under `private-keys-v1.d` and the new public key will be stored inside `pubring.kbx`

> **WARNING**: I've tried to describe which file contains/does what but consider that the correct way to operate on keys is via GPG's own commands.


## Primary key-pair fingerprint

In order to create the subkeys I need to know the primary key-pair's fingerprint which shown  by:

``` bash
gpg -K | grep fingerprint
```

Output:

> Key fingerprint = 7205 2A26 269F 7552 DDED  3662 6596 C5FA 775E B9AB

I can either copy if from the terminal (and manually remove all the spaces) or be lazy and use this command:

``` bash
fingerprint=$(gpg -K | grep fingerprint | cut -d= -f2 | sed 's/ //g')
echo ${fingerprint}
```

## Encryption subkey-pair creation

It's now time to create an encryption subkey:

``` bash
gpg --quick-add-key ${fingerprint} rsa4096 encr 2019-12-31
```

I'm asked to provide the passphrase for the primary key (in this example _"knapp tiber fist lush hatred we're"_)

Test:

``` bash
gpg -K

/home/user/.gnupg/pubring.kbx
-----------------------------
pub   rsa4096/0x6596C5FA775EB9AB 2018-10-08 [SC]
    Key fingerprint = 7205 2A26 269F 7552 DDED  3662 6596 C5FA 775E B9AB
    Keygrip = 50123401D7AA3CBF514F23D5FA98D3D485075B2E
uid                   [ultimate] Joe Tester <joe@foo.bar>
sub   rsa4096/0xFCFB0E137879D519 2018-10-08 [E] [expires: 2019-12-31]
    Key fingerprint = 6598 5A20 E7D5 871E E0EF  5A52 FCFB 0E13 7879 D519
    Keygrip = 4CB933841DCDF831B4CFFBC688B055BE472AC778
```

## Signature subkey-pair creation

It's now time to create a signature subkey:

``` bash
gpg --quick-add-key ${fingerprint} rsa4096 sign 2019-12-31
```

I'm asked to provide the passphrase for the primary key (in this example _"knapp tiber fist lush hatred we're"_)

Test:

``` bash
gpg -K

/home/user/.gnupg/pubring.kbx
-----------------------------
pub   rsa4096/0x6596C5FA775EB9AB 2018-10-08 [SC]
    Key fingerprint = 7205 2A26 269F 7552 DDED  3662 6596 C5FA 775E B9AB
    Keygrip = 50123401D7AA3CBF514F23D5FA98D3D485075B2E
uid                   [ultimate] Joe Tester <joe@foo.bar>
sub   rsa4096/0xFCFB0E137879D519 2018-10-08 [E] [expires: 2019-12-31]
    Key fingerprint = 6598 5A20 E7D5 871E E0EF  5A52 FCFB 0E13 7879 D519
    Keygrip = 4CB933841DCDF831B4CFFBC688B055BE472AC778
sub   rsa4096/0xB7CB9817FF022A44 2018-10-08 [S] [expires: 2019-12-31]
    Key fingerprint = 5FF4 7D3F FD9F 95DB 9C96  1455 B7CB 9817 FF02 2A44
    Keygrip = 4E37F90BFA7E9A823283F3976C79499A67E0EA03
```

This is the content of `~/.gnupg`

``` bash
tree ~/.gnupg

/home/user/.gnupg
├── gpg.conf
├── openpgp-revocs.d
│   └── 72052A26269F7552DDED36626596C5FA775EB9AB.rev
├── private-keys-v1.d
│   ├── 4CB933841DCDF831B4CFFBC688B055BE472AC778.key
│   ├── 4E37F90BFA7E9A823283F3976C79499A67E0EA03.key
│   └── 50123401D7AA3CBF514F23D5FA98D3D485075B2E.key
├── pubring.kbx
├── pubring.kbx~
├── tofu.db
└── trustdb.gpg

2 directories, 9 files
```
## Tofu.db

From the output of the previous command I notice that, after adding the 2 sub-keys, a new file appears in the directory: `tofu.db`.

What is it?

With a brief research I've found out this:

From [GPG mailing list](https://lists.gnupg.org/pipermail/gnupg-users/2015-October/054608.html):
> TOFU stands for Trust on First Use and is a concept that will be
familiar to anyone who regularly uses ssh.  When you ssh to a host for
the first time, ssh asks you to verify the host's key (most people
just say yes here).  When connecting to the same host in the future,
ssh checks that the key hasn't changed.  If it has, ssh displays a
warning.

> TOFU for GnuPG works similarly.  When you verify a message from some
user for the first time, GnuPG saves the binding between the user id
(actually, the normalized email address) and the key.  When you verify
another message from that user, the saved bindings with that user's
address are retrieved.  If there is at least one such binding, but
none of them include the signer's key, then either the signer is using
a new key or someone is attacking you.  In this case, GnuPG displays a
warning and prompts you to verify the key and set an appropriate
policy (e.g., the key should be considered untrusted).

## Authentication subkey-pair creation

The authentication subkey is not relevant from the encryption and signature stand point but I'm interested in the [monkeysphere project](http://web.monkeysphere.info/why/) which uses GPG for the _SSH authentication_ and for the _HTTPS validation_ of websites.

So I'm going the extra mile:

``` bash
gpg --quick-add-key ${fingerprint} rsa4096 auth 2019-12-31
```

Now, these are these key-pairs available in my GPG key-ring:

``` bash
gpg -K

/home/user/.gnupg/pubring.kbx
-----------------------------
pub   rsa4096/0x6596C5FA775EB9AB 2018-10-08 [SC]
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

and this is the content of the `~/.gnupg/` directory:

``` bash
tree ~/.gnupg

/home/user/.gnupg
├── gpg.conf
├── openpgp-revocs.d
│   └── 72052A26269F7552DDED36626596C5FA775EB9AB.rev
├── private-keys-v1.d
│   ├── 3880DC542784CFC56B5773E50FF00289620071EA.key
│   ├── 4CB933841DCDF831B4CFFBC688B055BE472AC778.key
│   ├── 4E37F90BFA7E9A823283F3976C79499A67E0EA03.key
│   └── 50123401D7AA3CBF514F23D5FA98D3D485075B2E.key
├── pubring.kbx
├── pubring.kbx~
├── tofu.db
└── trustdb.gpg
```

## Adding a photo to the public key

This step is optional but I like the idea to have a small photo embedded in the public key.
In this way it will be easier for others to identify my public key among many others.
It doesn't have to be a personal photo, perhaps it can be an avatar.

GPG accepts only the jpg type and it should be small in size and in weight. 50x50 pixel and 2 to 20 KB is OK.

I add the photo with this interactive command:

``` bash
gpg --edit-key ${fingerprint}

# at the prompt write:
# addphoto

# Then:
# type photo's absolute path
# type your passphrase
# type "save"
# type "quit"
```

---


# SSL key creation

I don't want to remember Joe's GPG passphrase and I'm happy to use `openssl` to store the passphrase (encrypted with AES 256) next to the coffin key inside the _hush-partition_.

I check that the sdcard is mounted:

``` bash
mount | grep hush
```

Output:

> /dev/mapper/hush on /home/user/.hush type ext4 (rw,relatime,data=ordered)
> /dev/mapper/hush on /rw/home/user/.hush type ext4 (rw,relatime,data=ordered)

If the output is empty then I have to run :

* dom0: `attach_hush_to vault`
* vault: `risks mount hush`


When `openssl` asks for the encryption password, I use the same that I use for `mpw`

``` bash
echo "knapp tiber fist lush hatred we're" | openssl enc -aes-256-cbc -a > ${HUSH_DIR}/${IDENTITY}-gpg.ssl
```

I immediately protect the ssl key against accidental deletion or change by making it immutable:

``` bash
sudo chattr +i ${HUSH_DIR}/${IDENTITY}-gpg.ssl
```

Test:

``` bash
risks gpgpass ${IDENTITY}
```

I should be able to see again _"knapp tiber fist lush hatred we're"_. This is the command I use before opening any


# Coffin creation

I use one coffin-file to store the GPG configuration files of each identity.

This allows to:

* quickly load any identity
* keep identities isolated

## Coffin key creation

I create a random binary file (human unreadable) that I use as key for the _gpg-coffin_:

``` bash
IDENTITY="joe"
head -c 512 < /dev/urandom > ${HUSH_DIR}/${IDENTITY}-gpg.key
```

Alternatively I can use a human readable random key:

``` bash
pwgen -y -s -C 64 > ${HUSH_DIR}/${IDENTITY}-gpg.key
```

I immediately protect the key against accidental deletion or change by making it immutable:

``` bash
sudo chattr +i ${HUSH_DIR}/${IDENTITY}-gpg.key
```

I can test the immutability of files with:

``` bash
lsattr ${HUSH_DIR}
```

Output:

> ----i---------e---- /home/user/.hush/joe-gpg.key

The `i` character means that the file is marked as immutable.


## Coffin creation

I'm ready to create the gpg-coffin container (50MB):

``` bash
dd if=/dev/urandom of=${GRAVEYARD}/${IDENTITY}-gpg.coffin bs=1M count=50
```

I lay the gpg-coffin inside the container and I bond it to its key, all in one command:

``` bash
sudo cryptsetup -v -q --cipher aes-xts-plain64 --master-key-file ${HUSH_DIR}/${IDENTITY}-gpg.key --key-size 512 --hash sha512 --iter-time 5000 --use-random luksFormat ${GRAVEYARD}/${IDENTITY}-gpg.coffin ${HUSH_DIR}/${IDENTITY}-gpg.key
```

> WARNING:
>
> Here I'm not using a passphrase on purpose. I could use a tomb-file instead of a coffin-file and add a passphrase but this would increase a lot my mnemonic effort and, in my situation, it would be too much.
>
> It's true that who has access to the gpg-coffin and its key is able to open it without a passphrase but it's also true that GPG is also protected by its own passphrase.
>
> Therefore just having the coffin and its key is not enough to use GPG.


Test:

``` bash
sudo cryptsetup luksDump ${GRAVEYARD}/${IDENTITY}-gpg.coffin
```

Output:

``` bash
LUKS header information for /home/user/.graveyard/joe-gpg.coffin

Version:       	1
Cipher name:   	aes
Cipher mode:   	xts-plain64
Hash spec:     	sha512
Payload offset:	4096
MK bits:       	512
MK digest:     	9c 6f 45 11 92 01 5c 08 54 60 8b d6 9b 25 0b 53 0d 53 d9 26
MK salt:       	1e fa f4 dc c0 cc 2e 15 57 01 5d 52 c3 a2 f8 42
                aa 07 68 54 a5 a4 d5 10 e5 d8 1d d2 50 75 d3 bf
MK iterations: 	560000
UUID:          	fb718c68-add3-4a3d-9bb0-0c89cb81237df

Key Slot 0: ENABLED
    Iterations:         	4596049
    Salt:               	3b c7 98 a3 b3 30 49 43 b0 71 26 8f 57 f8 92 08
                            be c2 49 d2 4c 8b fb 70 5d d4 91 b5 de 9a eb b9
    Key material offset:	8
    AF stripes:            	4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

which displays the UUID of the gpg-coffin and shows that only one key has been configured for it.

## Coffin setup

I open the gpg-coffin (no passphrase is asked):

``` bash
sudo cryptsetup open --type luks ${GRAVEYARD}/${IDENTITY}-gpg.coffin coffin-${IDENTITY}-gpg --key-file ${HUSH_DIR}/${IDENTITY}-gpg.key
```

Test:

``` bash
sudo cryptsetup status coffin-${IDENTITY}-gpg
```

Output:

``` bash
/dev/mapper/coffin-joe-gpg is active.
type:    LUKS1
cipher:  aes-xts-plain64
keysize: 512 bits
device:  /dev/loop0
loop:    /home/user/.graveyard/joe-gpg.coffin
offset:  4096 sectors
size:    98304 sectors
mode:    read/write
```

The gpg-coffin is open and ready to be formatted. I use the ext4 filesystem:

``` bash
sudo mkfs.ext4 -m 0 -L coffin-${IDENTITY}-gpg /dev/mapper/coffin-${IDENTITY}-gpg
```

I create a temporary directory and I mount the gpg-coffin on it:

``` bash
TMP="/tmp/${IDENTITY}-gpg"
mkdir ${TMP}
sudo mount /dev/mapper/coffin-${IDENTITY}-gpg ${TMP}
sudo chown ${USER} ${TMP}
_```


Test:

``` bash
mount | grep ${IDENTITY}-gpg
```

I copy the GPG files in the gpg-coffin:

``` bash
cp -fR ${RAMDISK}/* ${TMP}
```

The GPG keys shouldn't be modified nor deleted, especially by mistake, so I make them immutable with `chattr`

``` bash
risks open gpg ${IDENTITY}
sudo chattr +i ${HUSH_DIR}/${IDENTITY}-gpg.key
sudo chattr +i ${TMP}/private-keys-v1.d/*
sudo chattr +i ${TMP}/openpgp-revocs.d/*
```

I close the coffin for good:

``` bash
sudo umount ${TMP}
sudo cryptsetup close /dev/mapper/coffin-${IDENTITY}-gpg
```

and I wipe and umount the ramdisk:

``` bash
cd
sudo wipe -r ${RAMDISK}
sudo umount ${RAMDISK}
```

This is the final result from the filesystem stand point:

``` bash
tree ~/.hush ~/.graveyard
/home/user/.hush
├── joe-gpg.key

/home/user/.graveyard
├── joe-gpg.coffin
```


Now the GPG files for `${IDENTITY}` exists only inside the gpg-coffin which I open or close with `risks` from _vault_:

``` bash
risks open coffin ${IDENTITY}
risks close coffin ${IDENTITY}
```

At this stage the GPG configuration for `${IDENTITY}` is complete and GPG can be used from _vault_ but Cabal's mitigation is still missing and before proceeding I need to create my first tomb-file.

---

# Recap for copy & paste

I recap here all the commands in a row so that I can later on use "copy and paste" for creating the gpg-coffin for the other identities:

``` bash
IDENTITY="other_identity"

head -c 512 < /dev/urandom > ${HUSH_DIR}/${IDENTITY}-gpg.key

dd if=/dev/urandom of=${GRAVEYARD}/${IDENTITY}-gpg.coffin bs=1M count=50

sudo cryptsetup -v -q --cipher aes-xts-plain64 --master-key-file ${HUSH_DIR}/${IDENTITY}-gpg.key --key-size 512 --hash sha512 --iter-time 5000 --use-random luksFormat ${GRAVEYARD}/${IDENTITY}-gpg.coffin ${HUSH_DIR}/${IDENTITY}-gpg.key

sudo cryptsetup open --type luks ${GRAVEYARD}/${IDENTITY}-gpg.coffin ${IDENTITY}-gpg --key-file ${HUSH_DIR}/${IDENTITY}-gpg.key

sudo mkfs.ext4 -m 0 -L ${IDENTITY}-gpg /dev/mapper/${IDENTITY}-gpg

TMP="/tmp/${IDENTITY}-gpg"

mkdir ${TMP}

sudo mount /dev/mapper/${IDENTITY}-gpg ${TMP}

sudo chown ${USER} ${TMP}
```

Then to close the coffin:

``` bash
sudo umount ${TMP}
sudo cryptsetup close /dev/mapper/${IDENTITY}-gpg
```
