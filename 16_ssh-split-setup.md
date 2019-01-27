---
title: SSH Split setup
permalink: ssh-split-setup
---

---

<!-- TOC -->

- [About SSH-split](#about-ssh-split)
- [Tomb file for ssh](#tomb-file-for-ssh)
- [Setup Qubes for SSH split](#setup-qubes-for-ssh-split)
    - [dom0 configuration](#dom0-configuration)
    - [vault configuration](#vault-configuration)
    - [template configuration](#template-configuration)
    - [joe-devq configuration](#joe-devq-configuration)
    - [Test](#test)

<!-- /TOC -->

---

# About SSH-split

The SSH-split configuration is especially interesting for IT professional like developers, devops and system administrators but, more in general, for anyone who has a remote server to manage via ssh.

It works the same way as GPG-split: _vault_ is the provider of the ssh-keys for other qubes which don't host the ssh-keys on their hard drive resulting in additional protection against malware and unauthorized access.

# Tomb file for ssh

I follow the standard procedure to create a tomb file (20MB) in _vault_ to host the keys of my `~/.ssh` directory:

``` bash
IDENTITY="joe"
RECIPIENT="joe@foo.bar"
LABEL="ssh"
TOMBID="${IDENTITY}-${LABEL}"
risks open gpg ${IDENTITY}
tomb dig -s 20 ${GRAVEYARD}/${TOMBID}.tomb
```
> don't change `${TOMBID}` or `risks` and `risq` will not work as expected

From now on I can open or close the ssh tomb-file with :

``` bash
risks open ssh ${IDENTITY}
risks close ssh ${IDENTITY}
```

If I have already some ssh keys for `${IDENTITY}` then I copy them inside the tomb:

``` bash
risks open ssh ${IDENTITY}
cp <my-old-keys-directory>/* ${HOME}/.ssh
```

Otherwise I create a new pair:

``` bash
risks open ssh ${IDENTITY}
ssh-keygen
```

The interactive shell opens and I hit _enter_ 3 times, accepting the defaults.

I check that the new keys have been created:

``` bash
ll ~/.ssh/
total 3
-rw------- 1 user user 1679 Jan 15 20:38 id_rsa
-rw-r--r-- 1 user user  394 Jan 15 20:38 id_rsa.pub
```

In both the cases I make the keys immutable:

``` bash
sudo chatt  +i ~/.ssh/id_rsa.*
risks close ssh ${IDENTITY}
```

# Setup Qubes for SSH split

This configuration grants _joe-devq_ to use the SSH configuration present in _vault_ without storing the ssh-files.

## dom0 configuration

In dom0 terminal:

``` bash
sudo sh -c 'echo "$anyvm $anyvm ask" > /etc/qubes-rpc/policy/qubes.SshAgent'
```

Eventually I can trust _joe-devq_ to access _vault_ ssh-agent without confirmation prompt.

``` bash
sudo sh -c 'echo "joe-devq vault allow" > /etc/qubes-rpc/policy/qubes.SshAgent'
sudo sh -c 'echo "$anyvm $anyvm ask" >> /etc/qubes-rpc/policy/qubes.SshAgent'
```
The `qubes.SshAgent` file becomes then something like this:

``` bash
joe-devq  vault allow
$anyvm $anyvm ask
```
## vault configuration

_vault_ needs just a couple of packages and to have ssh-add started at boot.

``` bash
sudo apt update
sudo apt install nmap ssh-askpass
mkdir ~/.config/autostart
echo '
[Desktop Entry]
Name=ssh-add
Exec=ssh-add
Type=Application
' > ${HOME}/.config/autostart/ssh-add.desktop
```

## template configuration

I turn off _joe-devq_, start _debian-10-dev_ and from its terminal:

``` bash
sudo apt update
sudo apt install nmap
sudo shutdown -P 0
```

## joe-devq configuration

I turn on _joe-devq_ and from its terminal I add these lines to `/rw/config/rc.local`:

``` bash
SSH_VAULT_VM="vault"
export SSH_SOCK="/home/user/.SSH_AGENT_$SSH_VAULT_VM"
rm -f "$SSH_SOCK"
sudo -u user /bin/sh -c "umask 177 && ncat -k -l -U '$SSH_SOCK' -c 'qrexec-client-vm $SSH_VAULT_VM qubes.SshAgent' &"
```

Then I add this to `${HOME}/.bashrc`

``` bash
SSH_VAULT_VM="vault"
export SSH_AUTH_SOCK=${HOME}/.SSH_AGENT_$SSH_VAULT_VM
```

## Test

* I stop _vault_
* I stop _joe-devq_
* I open a dom0 terminal
* I start _vault_ and open a terminal
* I start _joe-devq_ and open a terminal
* I attach the _hush partition_ to _vault_

From a dom0 terminal:

``` bash
qvm-shutdown _vault_ && qvm-run _vault_ gnome-terminal
qvm-shutdown _joe-devq_ && qvm-run _joe-devq_ gnome-terminal
attach_hush_to vault
```

From _vault_ terminal:

``` bash
risks mount sdcard
IDENTITY="joe"
risks open ssh ${IDENTITY}
ssh-add -L
```

The output is something like:

``` bash
ssh-rsa AAAAF3Pza01yc2EAAAADAQA3AAA3AQDPrqi5HAv6NLg+WJVjS14nGTpx+Jr/si/O0RYSMs21ran0K1xYAZ+h0sVrtUkt+JJHr3E38GHm6Dpah4err0by4uPfk+x3e15ZfZy4RsxttUXbkYmsto3byulUfOyN0dOYSL+jHt7i4qdvUNffTiqL0s/eaDb5q20ytg8g35WEhxchSa3y9PYRSPQ35dqjJU35DhON0yc9H36uhT4d0JtOjhpDL79JH3oi8+c0t4h+p1G3RvwLPnwkH
```

From _joe-devq_ terminal:

``` bash
ssh-add -L
```

The usual Qubes OS prompt-window pops up and I select _vault_.

Then, in _joe-devq_ terminal, I should see the exact same output displayed in _vault_. This proves that everything is working as expected.

I can now try to access any SSH remote server from _joe-devq_ with:

``` bash
ssh ${user}@${server_ip}
```