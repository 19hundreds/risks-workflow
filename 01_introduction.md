---
title: Introduction
permalink: /introduction
---

---

- [About](#about)
- [Audience](#audience)
- [What to expect from implementing R.I.S.K.S.](#what-to-expect-from-implementing-risks)
- [Disclaimer](#disclaimer)
- [Contribute](#contribute)
- [Why Qubes OS?](#why-qubes-os)
- [Why not Linux?](#why-not-linux)
- [Why not Windows or MacOS ?](#why-not-windows-or-macos-)
- [Can R.I.S.K.S. be adopted only in Qubes OS?](#can-risks-be-adopted-only-in-qubes-os)

---

# About

I'm very concerned by the monitoring and the profiling performed today and even more worried about tomorrow.

The implications on freedom frighten me.

This is my trigger. It has pushed me to investigate solutions and R.I.S.K.S. sums up part of what I've learned along the road.

# Audience

I hope and think that R.I.S.K.S. can help who has little security background and is concerned about it. That's exactly how I started with R.I.S.K.S.

I'm targeting journalists, lawyers, whistleblowers, activists, crypto-users, anyone involved in sensitive matters but also regular people. Anyone who believes that freedom is worth some sweating.

# What to expect from implementing R.I.S.K.S.

Setting up RISKS and following its workflow results in:

* just four (4) passphares to memorize to manage all the secrets (doesn't matter how many identities or how many secrets)
* an encrypted sdcard configured for hosting private keys
* an isolated qube (virtual machine) configured for:
    * managing GPG configurations for multiple identities
    * managing credentials using [pass](https://www.passwordstore.org/)
    * managing SSH configurations for multiple identities
    * managing multiple encrypted filesystems (embedded in a file) for multiple identities
    * providing a secure access to GPG and SSH keys from any other qube
* a set of scripts to manage the workflow comfortably
* disaster recovery and backup procedures

# Disclaimer

Despite the efforts, R.I.S.K.S. can have flaws.

It also relies on encryption which in some areas of the world is forbidden and brutally persecuted.

Use this guide at your own risk and be wise. Good luck.

# Contribute

Please share your doubts and **criticize** R.I.S.K.S.

Tear it apart without mercy but, please, make an effort to make R.I.S.K.S. better and vital to someone.

Correct what you think is wrong or suggest what can be improved.

Feel free to open an issue on [Github](https://github.com/19hundreds/password-management-workflow/issues) and to send a PR.

# Why Qubes OS?

[Qubes OS](https://www.qubes-os.org/) is a "reasonably secure operating system for personal computer". It's based on the Xen hypervisor and offers better security by **compartmentalizing** the user's activities in isolated Linux virtual machines called _qubes_.

The basic concept is that, by separating the daily activities and performing them in separated environments, the attack surface is smaller and minimizes the damages in case of accident.

Compartmentalization also goes along very well with the use of multiple identities resulting in enhanced user's privacy.

More security and more privacy are default achievements right out the box for a Qubes OS user, even for those with shallow technical background.

Don't trust me on this: see what the [experts](https://www.qubes-os.org/experts/) say about Qubes OS.

Unfortunately the learning curve can be quite steep (especially for those with no experience in linux) but its [documentation](https://www.qubes-os.org/doc/) is awesome.

# Why not Linux?

Qubes OS relies on Linux and all its qubes are based on Fedora. Who doesn't feel comfortable with non-debian based distributions _Debian_ qubes are available as well. So Linux is good part of Qubes.

A regular Linux installation, out of the box, can't grant the level of security and privacy provided by Qubes OS.

# Why not Windows or MacOS ?

They are closed source operating systems and this means that they can't be trusted. And none should (in my opinion).

# Can R.I.S.K.S. be adopted only in Qubes OS?

No, it can be implemented also in Linux but the user has to do the extra mile to fit R.I.S.K.S. in the chosen Linux box.