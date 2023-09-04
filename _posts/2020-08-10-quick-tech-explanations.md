---
id: 497
title: 'Quick tech explanations'
date: '2020-08-10T00:44:00+02:00'
author: Martin
layout: post
guid: 'https://www.sundhaug.com/?p=497'
permalink: /quick-tech-explanations/
categories:
    - Hacking
---

(Adapted from my Twitter)

What is TLS: Transport Layer Security, the protocol that does the security part of HTTPS. TLS includes “handshaking” so both the client and server can agree on various details on how to verify each other and how to encrypt data.

What is heartbleed: Transport Layer Security (the protocol that makes HTTP HTTPS), has an extension for doing heartbeat. The trouble is that the client can specify how many bytes it’s sending. Heartbleed was the result of servers not verifying how many bytes it had received, and thus could return “random” (other parts of the application, such as keys) data.

What were the issues with HKP (Http Public Key Pinning): It was difficult to use right and things could go very wrong. It has been removed from all major clients (either entirely or by default), so websites are unlikely to use it. In addition, it can be abused for ransom. Imagine if I could tell your users: Hey users, only connect to his site for the next X seconds if it presents my certificate, with X being some significant time-period like a month or year.

How Windows Update does transport-security (hint: It doesn’t): Instead of doing HTTPS, the client downloads files over HTTP, which have to be signed by Microsoft to be executed, which limits the danger. This is kind of like how Debian-based Linux-distros do it (though they use PGP-signed packages, and support independent PGP-keys)

Why is it “referer” and not “referrer” (in HTTP): It’s misspelled in the standard, which is how we got “referer”

How does MSCHAPv2 (MicroSoft Challenge Handshake Authentication Protocol) work (RADIUS, WPA2-Enterprise)? Here’s my ID (username), let’s do TLS, then I challenge you to hash the password-hash you have and some data I send you, and you challenge me by doing the same, we check what the other sent us back and if we agree then we’re good.