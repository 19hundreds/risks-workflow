---
title: GPG setup - part 2
permalink: gpg-setup-part2
---

---
<!-- TOC -->

- [Pen-drive initialization](#pen-drive-initialization)
- [GPG backup](#gpg-backup)
    - [Primary key-pair backup](#primary-key-pair-backup)
    - [Subkey-pairs backup](#subkey-pairs-backup)
    - [Comfortable is better](#comfortable-is-better)
- [HUSH backup](#hush-backup)

<!-- /TOC -->
---

At this stage the GPG configuration is fully functional because all the keys have been created.

Loosing the GPG configuration at this moment wouldn't be too much of a problem because the public is key is not yet published on any key-server on the internet and because the key has never been used so far.

I could destroy the coffin-file and start over again without any issue.

In the moment I start using Joe's GPG for instance to create a tomb-file I add extra-value and extra-responsibility to it so I need to take in consideration some form of disaster recovery before it's too late. Gambling on this things is not smart.

Unfortunately there isn't a single definitive solution for protecting files from all the possible threats so I start with backing up the keys on a dedicated external USB pen-drive which hosts.

# Pen-drive initialization

I'm going to use one pen-drive for all my identities

This step is not very different from the one used for the sdcard.

I open a terminal in _vault_ and I keep an eye on syslog:

``` bash
sudo multitail /var/log/syslog

#hit enter to place the red line marker
```

I plug the pen-drive in the USB socket and I connect it to _vault_ using Qubes _devices widget_

From syslog I can understand what's the device name that _vault_ gives to the pen-drive which, usually, is `/dev/xvdi` but in my case is `/dev/xvdj`.

``` bash
blkfront: xvdj: barrier or flush: disabled; persistent grants: enabled; indirect descriptors: enabled;

```

I erase all the content on the pen-drive:

``` bash
IDENTITY="joe"
PENDRIVE="/dev/xvdj"
MAPPER="pendev"
MOUNT_POINT="/tmp/pendrive"

sudo dd if=/dev/urandom of=${PENDRIVE} bs=1M
sync
```
It's a long operation which ends when the pen-drive is completely filled up.

When finished, without creating any partition, I encrypt with LUKS the entire pen-drive and I protect it with the same passphrase I use for Qubes OS. I choose this way because Qubes OS passphrase is used only at boot time and it's hard to believe that a key-logger can spot it. Most likely the only way to have this passphrase compromised is to be monitored meanwhile typing it and this makes it rather safe for me.

``` bash
sudo cryptsetup -v -q -y --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random luksFormat ${PENDRIVE}
```

Then I open, format and mount the LUKS filesystem:

``` bash
mkdir ${MOUNT_POINT} &> /dev/null
sudo cryptsetup open --type luks ${PENDRIVE} ${MAPPER}

sudo mkfs.ext4 -m 0 -L "gpg-backup" /dev/mapper/${MAPPER}
sudo mount /dev/mapper/${MAPPER} ${MOUNT_POINT}
sudo chown ${USER} ${MOUNT_POINT}
BCKDIR="${MOUNT_POINT}/gpg/${IDENTITY}"
mkdir -p ${BCKDIR}
```


# GPG backup

I open `${IDENTITY}` gpg-tomb-file and I obtain its `${FINGERPRINT}

``` bash
risks gpgpass ${IDENTITY}

risks open gpg ${IDENTITY}

FINGERPRINT=$(gpg -K | grep fingerprint | head -n 1 | cut -d= -f2 | sed 's/ //g')
```

For future memory, I backup the output of `gpg -k`. It can come in handy in case things go south at some point.

``` bash
gpg -k | tee ${BCKDIR}/gpg-k_output.txt
```

## Primary key-pair backup

I backup the private key of the primary key-pair in an armored backup.

_armored backup_ means that the content of the backup key will be in plain text (ASCII) so that I can think, later on, to print it as QRCODE, transfer it on paper and physically store it somewhere I consider safe.

``` bash
gpg --export-secret-keys --armor ${FINGERPRINT} > ${BCKDIR}/private-primary-keypair.arm.key
```

> Note: if you want to add an additional layer of encryption for the GPG keys you can consider to use `openSSL` like this:

``` bash
gpg --export-secret-keys --armor ${FINGERPRINT} | openssl enc -aes-256-cbc -a  > ${BCKDIR}/private-primary-keypair.arm.key
```

I backup the public key of the primary key-pair:

``` bash
gpg --export --armor ${FINGERPRINT} > ${BCKDIR}/public-primary-keypair.arm.key
```

## Subkey-pairs backup

Then the subkeys but this time in a single binary file:

``` bash
gpg --export-secret-subkeys ${FINGERPRINT} > ${BCKDIR}/private-subkeys.bin.key
```

This is what I have now in my pen-drive:

``` bash
ls -l

-rw-r--r-- 1 user user   906 Oct  8 19:23 gpg-k_output.txt
-rw-r--r-- 1 user user 14126 Oct  8 19:15 private-primary-keypair.arm.key
-rw-r--r-- 1 user user  9040 Oct  8 19:18 private-subkeys.bin.key
-rw-r--r-- 1 user user  6880 Oct  8 19:15 public-primary-keypair.arm.key
```

## Comfortable is better

For my convenience I keep a copy of Joe's public key and its signature somewhere accessible on my hard drive so that I can reach it out any time and send it to someone else.

``` bash
echo ${FINGERPRINT} > ${HOME}/gpg-${IDENTITY}-fingerprint.txt
gpg --export --armor ${FINGERPRINT} > ${HOME}/gpg-${IDENTITY}-public-armored.key
```

# HUSH backup

I also backup the _hush partition_ and the _graveyard_.

``` bash
risks close gpg ${IDENTITY}
risks umount hush

sudo dd if=/dev/hush of=${MOUNT_POINT}/hush.img

mkdir ${MOUNT_POINT}/graveyard
cp -fR ${HOME}/.graveyard/* ${MOUNT_POINT}/graveyard
```

These are the files stored in the pen-drive:

``` bash
tree

.
├── graveyard
│   ├── joe-gpg.coffin
├── hush.img
└── gpg
    |── joe
        ├── gpg-k_output.txt
        ├── private-primary-keypair.arm.key
        ├── private-subkeys.bin.key
        └── public-primary-keypair.arm.key
```

The backup stage is finished

``` bash
sudo umount ${MOUNT_POINT}
sudo cryptsetup close ${MAPPER}
```
