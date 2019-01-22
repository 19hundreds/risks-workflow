---
title: Get started
permalink: get-started
---

---

- [Before starting](#before-starting)
    - [Isolated place](#isolated-place)
    - [Paper password](#paper-password)
- [Get started](#get-started)
- [Vault configuration](#vault-configuration)
    - [Generic software](#generic-software)
    - [Software for secrecy](#software-for-secrecy)
    - [Improved entropy](#improved-entropy)
    - [Turn off swap](#turn-off-swap)
    - [Install tomb](#install-tomb)
    - [Risks-scripts repository](#risks-scripts-repository)
- [joe-fsq and joe-devq configuration](#joe-fsq-and-joe-devq-configuration)

---

# Before starting

## Isolated place

When working on R.I.S.K.S. be sure to be in a peaceful and isolated environment so that you can work with calm and reasonably sure that none is watching.

## Paper password

Writing down passphrases on paper is usually not a good idea. However it's a mandatory step until there is a better place where to store them: the risk is to be locked out of the system.

I have the habit of creating passphrases (diceware) before starting the encryption process and I write them down on paper. I call it _password paper_.

I memorize the passphrases by repeating them by heart hundreds times until I feel confident that I won't forget them anymore.

I mentally repeat my passwords on daily basis even if I'm not using my devices. It has become a traffic-light habit.

When I'm done with the software configuration I destroy the password paper by shredding it and flushing it down the toilet.

Soon you'll need:

* one passphrase for Qubes OS filesystem (diceware 6 words)
* one passphrase for the sdcard (diceware 6 words)
* one passphrase for Joe's GPG (diceware 6 words)
* one master password for `mpw` * (length >= 14)
* one password for the Qubes user (length >= 8)

Each passphrase must be different. Eventually the two passwords can match but I don't advise it.

# Get started

Follow these steps:

1. find a pc that you can use just for Qubes OS. Don't use Qubes OS in dual boot with another operating system.
2. if you can't afford another pc just buy a small SSD drive (avoid mechanic hard drives mostly because of low speed) and plug it inside your current pc.
3. [download Qubes OS R4.0](https://www.qubes-os.org/downloads/) and burn the Qubes OS iso on dvd or [dd-copy](https://www.qubes-os.org/doc/installation-guide/) it on a usb hard-drive or pen-drive
4. follow step by step the [official Qubes OS guide](https://www.qubes-os.org/getting-started/) and install Qubes OS on your pc. No need to look around for third parties tutorials.
5. spend a few days/weeks on Qubes OS until you understand its basics
6. learn how to create Debian 9 based templates following these [instructions](https://www.qubes-os.org/doc/templates/debian/)
7. learn how to create Debian 10 based templates
8. learn how to create standaloneVMs and AppVMs based on Debian 9 and 10. Juggle with this a little.

> Be sure to use a good diceware passphrase for Qubes OS filesystem. It's asked during Qubes installation process.

From now I assume that all this has been done.

# Vault configuration

As already mentioned I have a qube called _vault_ which handles secrets, passwords and GPG.

My _vault_ is a standalone qube created from an official Qubes OS Debian 9 image from which I remove the unneeded software.

No need to create the _vault_ as AppVM because there is just one _vault_ serving all the identities.

If you don't like Debian you can use your preferred Linux distribution but you need to adjust R.I.S.K.S. instructions accordingly.

> see the file `vault_packages.txt` in _risks-scripts_ for a list of suggested packages for _vault_.

_vault_ is supposed to be a network-less qube but during its configuration it needs to be connected to the internet (at least until I find an answer to this [question](https://www.reddit.com/r/Qubes/comments/agyune/how_to_use_update_proxy_on_standalonevm/) ).

A neater way would be to download all the required .deb packages and the additional software in a different network-connected qube and pass it to _vault_ via `qvm-copy`.

## Generic software

I like to install these general purpose packages:

``` bash
    sudo apt install apt-file wipe coreutils locate tree pwgen git
```

## Software for secrecy

This some security-focused software:

``` bash
    sudo apt update
    sudo apt install cryptsetup pass gnupg2 qubes-gpg-split e2fsprogs steghide
```

> WARNING: the GPG version must be >= 2.1 `apt-cache show gnupg2 | grep -i version`

I install `e2fsprogs` because I need `chattr`

I also install `sox` (which contains `play`) because I like to hear some sound when the sdcard is mounted or dismounted. This is optional.

```bash
    apt install sox
```

## Improved entropy

The `haveged` service increases the system's entropy. Higher entropy improves the quality of the keys during their generation.

``` bash
    sudo apt install haveged rng-tools
```

I check that the haveged service is running:

``` bash
    sudo systemctl is-active haveged.service
```
Output:

``` bash
    active
```

I check how much entropy my system is capable to generate:

``` bash
    cat /proc/sys/kernel/random/entropy_avail
```

Output:

``` bash
    2113
```

The value ranges between 0 and 4096. The higher the better. (`man random.4` -> entropy_avail). 2113 is probably not great but it seems ok.

Now, I test my entropy level with `rngtest`:

``` bash
    cat /dev/random | rngtest -c 1000
```

Output

``` bash
    rngtest 2-unofficial-mt.14
    Copyright (c) 2004 by Henrique de Moraes Holschuh
    This is free software; see the source for copying conditions.  There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

    rngtest: starting FIPS tests...
    rngtest: bits received from input: 20000032
    rngtest: FIPS 140-2 successes: 1000
    rngtest: FIPS 140-2 failures: 0
    rngtest: FIPS 140-2(2001-10-10) Monobit: 0
    rngtest: FIPS 140-2(2001-10-10) Poker: 0
    rngtest: FIPS 140-2(2001-10-10) Runs: 0
    rngtest: FIPS 140-2(2001-10-10) Long run: 0
    rngtest: FIPS 140-2(2001-10-10) Continuous run: 0
    rngtest: input channel speed: (min=1.241; avg=35.026; max=19073.486)Mibits/s
    rngtest: FIPS tests speed: (min=2.612; avg=33.875; max=87.493)Mibits/s
    rngtest: Program run time: 1116847 microseconds
```

Look for _rngtest: FIPS 140-2 successes_: it should score as close as possible to 1000.

## Turn off swap

I turn off swap to prevent that something can be written in the swap area and later recovered by an attacker. It's also a Tomb requirement.

    sudo swapoff -a

I test that the swap if off:

```
    free -h
                total        used        free      shared  buff/cache   available
    Mem:           465M        203M         36M        6.2M        225M        243M
    Swap:            0B          0B          0B
```

The swap size is 0B. Good.

I make this permanent by modifying the `rc.local` script

```bash
    sudo sh -c 'echo "swapoff -a" >> /rw/config/rc.local'
```

## Install tomb

Tomb requires some additional packages:

```bash
    sudo apt-get install pinentry-curses zsh
```

Tomb doesn't have a debian package but it's just a collection of bash scripts and so the installation is smooth.

```bash
    cd /tmp
    wget -c https://files.dyne.org/tomb/Tomb-2.5.tar.gz
    wget -c https://files.dyne.org/tomb/Tomb-2.5.tar.gz.sha
    sha256sum -c Tomb-2.5.tar.gz.sha
```
Output:

> Tomb-2.5.tar.gz: OK

Then:

```bash
    tar xvfz Tomb-2.5.tar.gz
    cd Tomb-2.5
    sudo make install
    cd ..
    rm -fR Tomb-2.5
```

## Risks-scripts repository

The last bit is to download the _risks-script_ repository.

``` bash
    git clone https://github.com/19hundreds/risks-scripts.git
```

I will then use `qvm-copy` to copy what I need to other qubes or dom0.

# joe-fsq and joe-devq configuration

Both the _joe-fsq_ and _joe-devq_ qubes are Debian 10 AppVMs generated from a Debian 10 Template which has no special features.

They could be Debian 9 AppVMs (it would be better from the security standpoint) but the most common applications (like browser, file manager ... ) wouldn't be much up to date. From R.I.S.K.S. standpoint there would be no difference, all the commands and packages are the same.