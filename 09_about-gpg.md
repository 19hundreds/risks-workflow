---
title: About GPG
permalink: about-gpg
---

---

- [Trust at risk](#trust-at-risk)
- [Cabal's risk mitigation](#cabals-risk-mitigation)

---

R.I.S.K.S. relies on GPG for identifying an identity and for encrypting data. Many other applications and the web of trust rely on it.

From [GPG Privacy Handbook](https://www.gnupg.org/gph/en/manual.html):

> GnuPG uses public-key cryptography so that users may communicate securely. In a public-key system, each user has a pair of keys consisting of a private key and a public key. A user's private key is kept secret; never reveal it to anyone. The public key may be given to anyone with whom the user wants to communicate. GnuPG uses a somewhat more sophisticated scheme in which a user has a primary key-pair and then zero or more additional subordinate key-pairs. The primary and subordinate key-pairs are bundled to facilitate key management and the bundle can often be considered simply as one key-pair.


# Trust at risk

Unfortunately the default GPG setup exposes the private key to unnecessary danger. This can lead to unfortunate situations.

From [wikipedia about web of trust](https://en.wikipedia.org/wiki/Web_of_trust):

> There are two keys pertaining to a person: a public key which is shared openly and a private key that is withheld by the owner. The owner's private key will decrypt any information encrypted with its public key. In the web of trust, each user has a ring with a group of people's public keys.

> GnuPG is able to create several different types of key-pairs, but **a primary key must be capable of making signatures**. In all cases it is possible to later add additional subkeys for encryption and signing.

From the [unattended GPG key generation manual](https://gnupg.org/documentation/manuals/gnupg-2.0/Unattended-GPG-key-generation.html):

> Allowed values _for the each key-pair flag_ are: ‘encrypt’, ‘sign’, and ‘auth’.

> GPG requires that all primary keys are capable of certification, so no matter what usage is given here, the ‘cert’ flag will be on.

By default, at the time of creating a new key-pair, GPG creates one signing key-pair, which handles the owner's identity, and one encryption subkey, which handles the decryption of messages intended for the owner.

These keys are crucial. If compromised or lost it's game over and the owner faces two kind of issues: secrets loss and impersonification.

* **secrets loss**, meaning: an attacker in possession of the encryption key-pair can decrypt all the messages intended for the owner. This is secrets stealing.

* **impersonification**, meaning that the attacker can perform activities and talk to others in the owner's behalf. This is identity stealing.

When the private key is out of control, there is no solution against secrets loss. Period.

The impact of impersonification can be mitigated by revoking the key (assuming that the owner still has a revoke certificate for the compromised identity) but revoking the primary key-pair also means losing years of signatures and destroying the trust ring built around that key-pair. It's a dramatic situation which creates a massive inconvenience to the owner and to all the people trusting that key-pair.

# Cabal's risk mitigation

Accordingly to [Alex Cabal's guidelines](https://alexcabal.com/creating-the-perfect-gpg-keypair/), part of the answer sits in the concept of subkeys: they can’t prevent secret-loss but they can mitigate the damage on the trust ring.

In this approach (validated by many others, debian included) three key-pairs are created: one primary key-pair and two subkey-pair belonging to the primary key-pair.

* The primary key-pair has the ‘sign’ (consequently the ‘cert’ too) flag ON. [SC]
* The other subkey has the ‘encrypt’ ON. [E]
* The last subkey has the ‘sign’ and ‘cert’ flag ON. [SC]

After these 3 key-pairs are created, the primary key-pair is removed from the `.gnupg` folder, kept secure and used the least possible (or never) only in the most secure environment so that it won't ever be exposed to treats. This will make the primary key-pair an **offline keypair**. In the end, only the two subkeys will live in the computer.

Holding the primary key safely means that the owner can revoke any subkey any time and, because the primary key-pair must not be revoked, the owner doesn’t have to create a new primary key-pair and go through the hassle of getting people to sign it again.

The owner still has to revoke the stolen subkeys and the attacker can still use the encryption subkey to decrypt any message that the key can decrypt but the damage won’t be as catastrophic.

From [debian-wiki](https://wiki.debian.org/Subkeys?action=show&redirect=subkeys):

> You will need to use the master keys only in exceptional circumstances, namely when you want to modify your own or someone else's key. More specifically, you need the master private key when you:
>
> * sign someone else's key or revoke an existing signature
> * add a new UID or mark an existing UID as primary
> * create a new subkey
> * revoke an existing UID or subkey
> * change the preferences (e.g., with setpref) on a UID
> * change the expiration date on your master key or any of its subkey
> * revoke or generate a revocation certificate for the complete key
>
> (Because each of these operation is done by adding a new self- or revocation signatures from the private master key.)
>
> Since each link of the Web of Trust is an endorsement of the binding between a public key and a user ID, OpenPGP certification signatures (from the signer's private master key) are relative to a UID and are irrelevant for subkeys. In particular, subkey creation or revocation does not affect the reputation of the master key. So in case your subkey gets stolen while your master key remains safe, you can revoke the compromised subkey and replace it with a new subkey without having to rebuild your reputation and without reducing reputation of other people's keys signed with your master key.
