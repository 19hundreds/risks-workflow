---
title: Get started
permalink: get-started
---

---

<!-- TOC -->

- [Get ready](#get-ready)
    - [Isolated place](#isolated-place)
    - [Paper password](#paper-password)
- [Get started](#get-started)
- [Vault configuration](#vault-configuration)
    - [Network-less vault & Updates Proxy](#network-less-vault--updates-proxy)
    - [Generic software](#generic-software)
    - [Software for secrecy](#software-for-secrecy)
    - [Improved entropy](#improved-entropy)
    - [Disable bash history](#disable-bash-history)
    - [Turn off swap](#turn-off-swap)
    - [Install tomb](#install-tomb)
    - [Risks-scripts repository](#risks-scripts-repository)
    - [Vault global vars](#vault-global-vars)
- [joe-fsq and joe-devq configuration](#joe-fsq-and-joe-devq-configuration)

<!-- /TOC -->
---

# Get ready

## Isolated place

When working on R.I.S.K.S. be sure to be in a peaceful and isolated environment so that you can work with calm and reasonably sure that none is watching.

## Paper password

Writing down passphrases on paper is usually not a good idea but I believe it's still good until there is a better place where to store them.

At least it prevents the worst risk: being locked out.

I have the habit of creating passphrases before starting the encryption process and I write them down on what I call _password paper_.

I memorize the passphrases by repeating them by heart hundreds times until I feel confident that I won't forget them.

I mentally repeat my passwords on daily basis even if I'm not using my devices. It has become a traffic-light habit.

Soon I'll need:

* a passphrase for Qubes OS filesystem
* a passphrase for the sdcard
* a passphrase for Joe's GPG
* a master password for `mpw`
* a password for the Qubes user

Each passphrase and password must be different.

Each passphrase and password difficulty must be calibrated accordingly to the environmental risk.

For example the difficulty of the Qubes user's password depends on the chances to have someone hacking the pc meanwhile it's locked with the screensaver.

This is an example of how my _password paper_ looks like.

**MUST BE REMEMBERED**

| DESCRIPTION                               | PASSPHRASE                                 |
| ----------------------------------------- | ------------------------------------------ |
| Qubes OS partition                        | man smiles in front ## fireplace           |
| Qubes user                                | boring puzzword                            |
| HUSH partition                            | cat climbs tree dog pisses tree            |
| MPW master password                       | fly smacks glass =*                        |


**CAN BE FORGOTTEN** (if openssl is used, see later)

| DESCRIPTION                               | PASSPHRASE                                 |
| ----------------------------------------- | ------------------------------------------ |
| JOE's GPG coffin                          | knapp tiber fist lush hatred we're         |
| ...                                       | ...                                        |
| MIKE's GPG coffin                         | cleft cam synod lacy yr wok                |


When I'm done with the software configuration I destroy the password paper by shredding it and flushing it down the toilet.

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

The _vault_ qube handles secrets, passwords and GPG.

My _vault_ is a standalone network-less qube created from an official Qubes OS Debian 9 image from which I remove the unneeded software.

> If you don't like Debian you can use your preferred Linux distribution but you need to adjust R.I.S.K.S. instructions accordingly.

No need to create the _vault_ as AppVM because there is just one _vault_ serving all the identities.

It uses only one CPU and it has memory balancing. The initial memory is 180MB and the max is 512MB. Max storage size: 2048MiB.

All these parameters can be adjusted from _Qube Manager_.

I left the file `vault_packages.txt` in _risks-scripts_ for a list of suggested packages for _vault_.

## Network-less vault & Updates Proxy

_vault_ is a network-less qube so, from dom0 terminal:

``` bash
qvm-prefs vault netvm none
```

Because it's a StandaloneVM, it won't even get access by default to the _[Qubes Updates Proxy](https://www.qubes-os.org/doc/software-update-vm/#updates-proxy)_.

To solve this problem I proceed as follow:

In dom0 terminal, I add a tag to _vault_

``` bash
qvm-tags vault add standaloneVM
```

Tags can be listed with this command:

``` bash
qvm-tags vault list
```

A tag identifies a group of qubes. Then I edit `vim /etc/qubes-rpc/policy/qubes.UpdatesProxy` in this way:

``` bash
# Upgrade Whonix TemplateVMs through sys-whonix.
$tag:whonix-updatevm $default allow,target=sys-whonix
# Deny Whonix TemplateVMs using UpdatesProxy of any other VM.
$tag:whonix-updatevm $anyvm deny

# Upgrade all StandaloneVm tagged as standaloneVM through sys-net
$tag:standaloneVM $default allow,target=sys-net
# Deny StandaloneVMs tagged as standaloneVM using UpdatesProxy of any other VM.
$tag:standaloneVM $anyvm deny

# Default rule for all TemplateVMs - direct the connection to sys-net
$type:TemplateVM $default allow,target=sys-net
$type:TemplateVM $anyvm deny

$anyvm $anyvm deny
```

The two lines in the middle of the configuration file are giving access to any qube tagged as _standaloneVM_ to the _sys-net_ qube where _Qubes updates proxy_ runs by default.

Then I enable the service updates-proxy-setup for vault (still in dom0):

``` bash
qvm-service --enable vault updates-proxy-setup
```

The list of enabled services for _vault_ can be displayed with this commmand:

```
qvm-service --list vault
```

The output in my case is:

``` bash
updates-proxy-setup on
qubes-update-check  off
cups                off
```

I keep the _qubes-update-check_ service disabled because somehow it doesn't work for me. Not a big deal.

Now I turn on _vault_ (or reboot if it was on) and I check that the global var PROXY_ADDR is populated:

``` bash
echo ${PROXY_ADDR}
```

It should print:

``` bash
    http://127.0.0.1:8082
```

Also `/run/qubes-service/updates-proxy-setup` should be present (and empty).

This means that the _updates-proxy-setup_ service was started correctly.

This step should be optional but I run it to be sure:

``` bash
sudo /usr/lib/qubes/update-proxy-configs
```

This populates some configurations files in `/etc/` . I check that they were populated correctly:

``` bash
cat /etc/apt/apt.d/01qubes-proxy
cat /etc/PackageKit/PackageKit.conf
```

They should both refer to the proxy server `http://127.0.0.1:8082`

Finally I run `sudo apt update` and everything works as expected.

Now I'm able to install and update debian packages but I'm not able to download anything that doesn't exist in the standard Debian repository so anytime I need something outside the standard repositories I download the additional software in a different network-connected qube (_joe-devq_) and pass it to _vault_ via `qvm-copy`.

## Generic software

I like to install these general purpose packages:

``` bash
sudo apt install apt-file wipe coreutils locate tree pwgen git
```

## Software for secrecy

This some security-focused software:

``` bash
sudo apt update
sudo apt install cryptsetup pass gnupg2 qubes-gpg-split e2fsprogs steghide pass xclip libsodium18 libtinfo5
```

> WARNING: the GPG version must be >= 2.1 `apt-cache show gnupg2 | grep -i version`

I install `e2fsprogs` because I need `chattr`.

I install `xclip` so that I can copy passphrases and passwords in the clipboard. It's optionally used by `pass` and a requirement for `risks`.

`libsodium18` and `libtinfo5` are required for compiling `mpw` later on.

> Different Debian versions use different libraries so the final number might change.

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

## Disable bash history

Bash keeps a very comfortable log (called bash history) of the commands typed by the user in any terminal.

In _vault_ though I prefer to disable bash history so that it's not so obvious to see the commands I'm using.

``` bash
echo 'unset HISTFILE' >> .bashrc
source .bashrc
wipe -f .bash_history
```
With this configuration bash remembers only the commands typed in the current session. They are lost when the qube is restarted.

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
sudo sh -c "sed 's/bin\/sh/bin\/bash/g' -i /rw/config/rc.local"
sudo sh -c 'echo "swapoff -a" >> /rw/config/rc.local'
```

## Install tomb

Tomb requires some additional packages:

```bash
sudo apt-get install pinentry-curses zsh
```

Tomb doesn't have a debian package but it's just a collection of bash scripts and so the installation is smooth but I have to download the _tar.gz_ package from _joe-devq_:

```bash
cd /tmp
wget -c https://files.dyne.org/tomb/Tomb-2.5.tar.gz
wget -c https://files.dyne.org/tomb/Tomb-2.5.tar.gz.sha
sha256sum -c Tomb-2.5.tar.gz.sha
```
Output:

> Tomb-2.5.tar.gz: OK

Then I have to copy it to _vault_ with `qvm-copy`:

``` bash
qvm-copy Tomb-2.5.tar.gz  # I choose x-vault as target
```

Then from _vault_:

```bash
cd ~/QubesIncoming/joe-devq/
tar xvfz Tomb-2.5.tar.gz
cd Tomb-2.5
sudo make install
cd ..
rm -fR Tomb-2.5
```

## Risks-scripts repository

Now it's time to download the _risks-script_ repository, still in _joe-devq_:

``` bash
git clone https://github.com/19hundreds/risks-scripts.git
qvm-copy risks-scripts/vault/risks # I choose x-vault as target
```

Then from _vault_ copy `risks` to some `${PATH}`

``` bash
sudo cp ~/QubesIncoming/joe-devq/risks /usr/local/bin/
sudo chmod +x /usr/local/bin/
```


## Vault global vars

There are some global variables for _vault_ used to configure several scripts involved in R.I.S.K.S. and it's convenient to declare them already.

So, from _vault_ terminal:

``` bash
echo '
#!/bin/bash

# Hush partition: where the keys are store
export SDCARD_ENC_PART="/dev/hush"

# Mapper for hush partition
export SDCARD_ENC_PART_MAPPER="hush"

#Mount sound enabled: 0 / disabled: 1
export SDCARD_QUIET=0

# Keys mount point: where the hush partition is mounted
export HUSH_DIR="${HOME}/.hush"

# Data directory: where coffin-files and tomb-files are stored
export GRAVEYARD="${HOME}/.graveyard"

# PASS
export PASSWORD_STORE_ENABLE_EXTENSIONS=true
export PASSWORD_STORE_GENERATED_LENGTH=12

# GPG SPLIT configuration: QUBES_GPG_ACCEPT must be set in /etc/profile.d/qubes-gpg.sh
' >> ~/.bashrc

source ~/.bashrc
```

# joe-fsq and joe-devq configuration

Both the _joe-fsq_ and _joe-devq_ qubes are Debian 10 AppVMs generated from a Debian 10 Template which has no special features.

They could be Debian 9 AppVMs (it would be better from the security standpoint) but the most common applications (like browser, file manager ... ) wouldn't be much up to date. From R.I.S.K.S. standpoint there would be no difference, all the commands and packages are the same.
