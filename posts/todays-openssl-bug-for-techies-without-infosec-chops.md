<!--
.. title: Today's OpenSSL bug (for techies without infosec chops)
.. slug: todays-openssl-bug-for-techies-without-infosec-chops
.. date: 2015-07-09 08:26:58 UTC-07:00
.. tags: security
.. category:
.. link:
.. description:
.. type: text
-->

# What happened?

OpenSSL 1.0.1n+ and 1.0.2b+ had a new feature that allows finding an
alternative certificate chain when the first one fails. The logic in
that feature had a bug in it, such that it didn't properly verify if
the certificates in the alternative chain had the appropriate
permissions; specifically, it didn't check if those certificates are
certificate authorities.

Specifically, this means that an attacker who has a valid certificate
for any domain, can use that certificate to produce new
certificates. Those normally wouldn't work, but the algorithm for
finding the alternative trust chain doesn't check if the valid
certificate is allowed to act as a certificate authority.

# How big is the window?

1.0.1n and 1.0.2b were both released on 11 Jun 2015. The fixes, 1.0.1p
and 1.0.2d, were released today, on 9 Jul 2015.

The "good news" is that that isn't too long ago. Most people who have
an affected version will be updating regularly, so the number of
people affected is fairly small.

The following distros are affected at present (non-exhaustive list):

* OS X is by default not affected, because they still ship 0.9.8 by
  default. All bets are off if you got your OpenSSL from other places
  like Homebrew; the current stable version shioped by homebrew is
  1.0.2c, which is affected.
* [Ubuntu is mostly not affected][ubuntu]. The only affected version
  is the unreleased 15.10 (Wily), for which an update has already been
  released.
* Fedora is mostly not affected (20, 21, 22). Rawhide is affected, and
  [so is backports][fedora-bp].
* Debian stable is not affected, but [testing and unstable are][debian].
* [ArchLinux testing][arch] is affected.

[ubuntu]: http://people.canonical.com/~ubuntu-security/cve/2015/CVE-2015-1793.html
[arch]: https://www.archlinux.org/packages/?sort=-last_update
[debian]: https://security-tracker.debian.org/tracker/CVE-2015-1793s=openssl
[fedora-bp]: https://bugzilla.redhat.com/show_bug.cgi?id=1238619

# What's a certificate (chain)?

A certificate is a bit like an ID card: it has some information about
you (like your name), and is authenticated by a certificate authority
(in the case of an ID, usually your government).

# What's a certificate authority?

A certificate authority is an entity that's allowed to authenticate
certificates. Your computer typically ships with the identity of those
certificate authorities, so it knows how to recognize certificates
authorized by them.

In the ID analogy, your computer knows how to recognize photo IDs
issued by e.g. California.

The issue here is that in some cases, OpenSSL was willing to accept
entities that should not have been allowed to act as a certificate
authority. In the analogy, it would mean that it accepted CostCo
cards, too.

# Why did they say it wouldn't affect most users?

This basically means "we're assuming most users are using OpenSSL for
vanilla servers", which is probably true. Most servers do use OpenSSL,
and most clients (browsers) don't.

The bug affects anyone trying to authenticate their peer. That
includes regular clients, and servers doing client
authentication. Regular servers aren't affected, because they don't
authenticate their peer.

Servers doing client authentication are fairly rare. The biggest
concern is with clients. While browsers typically don't use OpenSSL, a
lot of API clients do. For those few people affected by the bug and
with clients that use OpenSSL, the bug is catastrophic.

# Why wasn't this found by automated testing?

I'm not sure. I wish automated testing this stuff was easier. Since
I'm both a user and a big fan of client authentication, which is a
pretty rare feature, I hope to spend more time in the future creating
easy-to-use automated testing tools for this kind of scenario.

# In conclusion

The bug is really bad, but affects few people. If you're running
stable versions of your operating system, you're almost certainly
safe.

The biggest concern, in my opinion, is for software developers using
OS X machines. Using OS X for HTTPS REST APIs is fairly common. OS X
comes with 0.9.8zf by default now, which is a recent revision of a
very old branch. Therefore, people have a strong motivation to get
their OpenSSL from a third-party source. The most popular source is
Homebrew, which currently ships ships 1.0.2c, and is affected.