---
id: 205
title: 'D-Link DIR-100D1 Authentification-bypass'
date: '2015-10-23T23:59:49+02:00'
author: Martin
layout: post
guid: 'http://www.sundhaug.com/?p=205'
permalink: /d-link-dir-100d1-authentification-bypass/
categories:
    - Hacking
tags:
    - dlink
---

(post recovered after backup-issues, thanks to Archive.org)  
After the previous set of bugs was discovered ([read my previous blog-post about it here](http://www.sundhaug.com/hacking-the-dir-100-rev-d1/)), D-Link implemented access controls. This made most commands and variables protected. So for example, an unauthentificated can’t ask for the administrative password in B13 and B15 nor can he do other naughty stuff an ill-behaved unauthentificated user may want to do (I’m considering writing more on that in a later blog-post) but he can ask for the current system language, that’s not too dangerous, now is it?

An inquisitive mind, will note that multiple cliget-commands can be batched together in one HTTP-request. If the unauthentificated user asks for say /cliget.cgi?cmd=$sys\_user1%;$poe\_user%;$poe\_pass, the router will tell him rightfully of, those are all protected variables and this is the expected result. However, let’s say the first variable isn’t a protected variable, let’s say the request is for /cliget.cgi?cmd=$sys\_language%;$sys\_user1%;$poe\_user%;$poe\_pass. As you’ll notice, this just asks for the current language in addition to all the previously mentioned hush-hush-stuff, the correct response would be for the router to tell him off again but unless it runs on a firmware I’m about to link you to, it’ll say “he asked for the language, the admin-credentials and the poe-credentials, language seems good go right ahead then” and let the unauthentificated user read the administrative password, amongst other things.

This should be seen as a good lesson for the importance of correct array usage; if you take an array, validate all elements before taking them for granted, don’t just check the first and be done with it.

This issue has been fixed in 4.03 B13\_fam2 ([ftp://dlinktemp:dlinktemp2015@194.117.170.198/Router/DIR-100/RevD/DIR100D1\_FW403WWB13\_fam2.bin](ftp://dlinktemp:dlinktemp2015@194.117.170.198/Router/DIR-100/RevD/DIR100D1_FW403WWB13_fam2.bin)) Yes, I know downloading from IPs, especially IPs not registred in a block owned by the device manufacturer seems shady, I don’t like it but that’s the link I got so that’s what I can give you, the firmwares don’t seem to be neither signed nor encrypted nor do any public hashes/hashfiles from a reputable source exist so you choose if you trust me or not, though the official D-Link twitter-account has apparently refered to it before (<https://twitter.com/dlink/status/181876200338231297>).

For the record, I don’t hate the DIR-100D1, I’ve grown rather fond of mucking about with it and if there’s interest, I’ll write more about what I’ve done to the poor thing and learned from mucking about.