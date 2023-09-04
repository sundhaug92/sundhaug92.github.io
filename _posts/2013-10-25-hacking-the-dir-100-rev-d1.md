---
id: 31
title: 'Hacking the DIR-100 rev. D1'
date: '2013-10-25T05:55:35+02:00'
author: Martin
layout: post
guid: 'http://sundhaug.com/?p=31'
permalink: /hacking-the-dir-100-rev-d1/
dsq_thread_id:
    - '1897614903'
categories:
    - Hacking
tags:
    - dlink
---

So, let’s say you have this router, the DIR-100 revision D1, I did. Being bored, I went the login screen but since the password was not the default I was unable login. So I cranked open Chromes fantastic development tools.

Now, I sent random values to try to capture some traffic and lo and behold there was a several requests made to /cliget.cgi?cmd=$… . The client-side code was loading configuration-data using XHR, including a partial MD5 of the password! Trying more values I discovered that if I requested /cliget.cgi?cmd=$, I would get the entire configuration file, **including the username and password**! The router was giving away admin access to everyone with access to the login page (LAN, or if remote admin is enabled LAN+WAN).

Later, revisiting the router I discovered that the thing didn’t check the HTTP Origin header, meaning that a basic CSRF attack could successfully perform a login, given a known username and password. I also discovered the /cli.cgi?cmd= API that is used by the client-side code to change the configuration setting, which also proved unable to hinder basic CSRF attacks. **Using a simple link and i bit of JavaScript a hacker could take control of someones network including but not limited to redirecting traffic.**

After watching a bit of WarGames (the 80’s flick that thaught us that every system has a help function) I got this funky idea, that would happen if I requested help from the router? Apparently, a lot, you get a list of every command in the system, including amongst other dhcps (lists all DHCP-clients), mem, reboot and GDB (supposedly not GNU GDB). One of the first thing i tried executing, was gdb which crashed the system, it might be that it triggers a debugger Connected to the internal 3-pin serial port. mem however is quite a different beast, it’s a memory monitor like debug.com of DOS-past. That’s right: you can **read and write memory on the device without authentification**<span style="text-decoration: underline;">. </span>

In addition to all this, I have it from credible sources that the router is plagued by the wide-spread UPnP portmap-from-wan bug that was detected, which means that an attacker may use the router for connection-bouncing and/or presumably use the UPnP bug to gain access to the internal admin-panel.

I will however remark that there has been some mention of the cliget.cgi and cli.cgi APIs before [http://forum.nag.ru/forum/index.php?showtopic=58050&amp;st=20](http://forum.nag.ru/forum/index.php?showtopic=58050&st=20).

So, as any responsible man would do, I attempted to contact D-Link, to no avail. D-Link has also made it difficult to track down firmware upgrades for the router.

I ask that anyone handling this information does so with care, and that D-Link makes sufficient arrangements for their unprotected customers.