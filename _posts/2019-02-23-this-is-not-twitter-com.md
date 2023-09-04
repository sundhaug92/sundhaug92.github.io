---
id: 308
title: 'This is not twitter.com'
date: '2019-02-23T01:11:50+01:00'
author: Martin
layout: post
guid: 'https://www.sundhaug.com/?p=308'
permalink: /this-is-not-twitter-com/
categories:
    - Uncategorized
---

<figure class="wp-block-embed-twitter wp-block-embed is-type-rich is-provider-twitter"><div class="wp-block-embed__wrapper">> seriously? [pic.twitter.com/yk6yh0CZy2](https://t.co/yk6yh0CZy2)
> 
> — Saleem Rashid (@spudowiar) [February 22, 2019](https://twitter.com/spudowiar/status/1098933359466692608?ref_src=twsrc%5Etfw)

<script async="" charset="utf-8" src="https://platform.twitter.com/widgets.js"></script></div><figcaption>Yup, seriously</figcaption></figure>Online identity is hard to do right, no matter if you ask the authentication folks or the cryptography folks either agree on this point, but this is not how you do it. Ignoring that the terminal period within the quotation marks should be outside the quotation marks, the description given there only proves that whoever sent the email gave enough fucks to turn setup TLS, which is free and dirt easy (Seriously, with many hosting providers, it’s a switch you flip).

The padlock-bit does not require the site to be secure neither in the “uses HTTPS”-way, nor in the “uses HTTPS securely”-way, nor the “does not have vulnerabilities”-way, a site filled to the brim with vulnerabilities, could still have the browser show a padlock icon (note, the description doesn’t say where to find it, and even if it did the site could use HTTPS and a favicon of a padlock, or an emoji somewhere in the <g class="gr_ gr_3 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling ins-del" data-gr-id="3" id="3">address-bar</g> 🔒.

As for the twitter.com-bit? It doesn’t say that that has to be the entire domain, nor even part of the domain, both https://twitter.com.sundhaug.com/ (not actually <g class="gr_ gr_133 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling ins-del" data-gr-id="133" id="133">setup</g>), and <https://www.sundhaug.com/this-is-not-twitter.com/> are valid links to prove an email is from Twitter, which is obviously absurd.

Now, of course, Twitter could change this text to specify that the link has to start with https://twitter.com/, but even then, anyone could send an email with a link to something at <g class="gr_ gr_480 gr-alert gr_gramm gr_inline_cards gr_run_anim Punctuation only-del replaceWithoutSep" data-gr-id="480" id="480">twitter.com,</g> and oftentimes do, without that making their emails “from” Twitter, and a phisher could expand upon the list of domains that are supposedly valid, because there’s no way for the user to verify that part of the email was from Twitter (and even if there was, why not just have a way to verify the entire email, even if most if not all users will never do so manually and you’d probably have to find a way to do it automatically).