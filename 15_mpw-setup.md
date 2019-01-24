---
title: MPW setup
permalink: mpw-setup
---

---
<!-- TOC -->

- [Get the code](#get-the-code)
- [Template configuration](#template-configuration)
- [Vault setup](#vault-setup)
- [At last](#at-last)
- [Usage](#usage)
    - [From vault](#from-vault)
    - [From AppVM](#from-appvm)

<!-- /TOC -->
---

With the installation of [mpw](masterpassword.app), `pass` becomes the most amazing credential manager I can think of.

# Get the code

There is no debian package ready for `mpw` so it must be compiled.

From _joe-devq_ I download the code:

``` bash
git clone https://github.com/Lyndir/MasterPassword.git
```

I copy the source code to _debian-10-dev_ (the template). So I start it and then from _joe-devq_:

``` bash
qvm-copy MasterPassword # I choose debian-10-dev as target
```

Then I do the same with _vault_:

``` bash
qvm-copy MasterPassword # I choose vault as target
```

I turn off the AppVM:

``` bash
sudo shutdown -P 0
```

# Template configuration

In _debian-10-dev_ I install two required libraries:

``` bash
# this is for Debian 10
sudo apt install libsodium23 libtinfo6

# this is for Debian 9
sudo apt install libsodium18 libtinfo5
```

I also install (see later why)

``` bash
sudo apt install xclip
```

then I compile the source code:

``` bash
cd MasterPassword/platform-indipendent/c/
./build
```

Output:

``` bash
Building target: mpw...
INFO:     Enabled mpw_sodium (libsodium).
INFO:     Enabled mpw_color (libtinfo).
WARNING:  mpw_json was enabled but is missing json-c library.  Will continue with mpw_json disabled!
done!  You can now run ./mpw-cli-tests, ./install or use ./mpw
```

> mpw_json is not necessary

If all goes well, the `mpw` binary is created and I can test it:

``` bash
./mpw
```

If the command doesn't throw errors, I move the binary:

``` bash
sudo mv mpw /usr/bin/
sudo chmod +x /usr/bin/mpw
```
> don't move it to /usr/local/bin or it won't show up in the _joe-devq_

I don't shutdown the template because I still need it.

# Vault setup

_vault_ should be a Debian 9 and the libraries are already installed (from the _Get started_ chapter)

I just need to compile `mpw` in the exact same way as above and move it to `/usr/bin/`

Additionally I need to copy `risq` to _debian-10-dev_:

``` bash
qvm-copy ${HOME}/risks-scripts/appvm/risq # I choose joe-devq as target
```

# At last

I power off the template (_debian-10-dev_):

``` bash
sudo shutdown -P 0
```

and I start _joe-devq_

# Usage

`mpw` is an algorithm for generating password. It takes some input arguments, the master password and it generates a random password.

This allows me to remember just one password for all my credentials.

Instead of saving my passwords in the first line of a pass-file (as by default)I write, instead, the entire `mpw` command-string used to generate the password.

Example:

``` bash
mpw -q -t l -c 1 -a 3 -u joe@foo.bar www.reddit.com
---
domain: www.reddit.com
username: joe@foo.bar
url: https://www.reddit.com
```
Of course I don't write the master password inside the file.

## From vault

When I want to retrieve the password for reddit.com I do:

``` bash
risks open identity ${IDENTITY}
$(pass reddit.com | head -n 1)
```

`mpw` prompts for the master password and I get the password displayed in the terminal.

## From AppVM

I prefer to obtain my credentials directly in the terminal of the AppVM that I'm using and I use `risq` for this.

> this works only if the AppVM is configured for pass-split!

``` bash
risq pass reddit.com
```
it asks for the master password (and the GPG passphrase if the access has timeout) and this the output:

``` bash
domain: www.reddit.com
username: joe@foo.bar
url: https://www.reddit.com

Password has been saved in clipboard
Press CTRL+V to use the content in this qube
Press CTRL+SHIFT+C to share the clipboard with another qube
In the other qube, press CTRL+SHIFT+v and then CTRL+V to use the clipboard content
Local clipboard will be erased is 45 seconds
```

The nice part is that I get the password copied in the clipboard and not displayed on the screen.
