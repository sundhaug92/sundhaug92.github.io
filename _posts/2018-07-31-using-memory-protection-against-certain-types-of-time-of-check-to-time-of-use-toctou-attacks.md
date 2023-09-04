---
id: 238
title: 'Using memory-protection against certain types of time of check to time of use (TOCTOU)-attacks'
date: '2018-07-31T09:09:06+02:00'
author: Martin
layout: post
guid: 'https://www.sundhaug.com/?p=238'
permalink: /using-memory-protection-against-certain-types-of-time-of-check-to-time-of-use-toctou-attacks/
categories:
    - Hacking
    - Uncategorized
---

Time of check to time of use is a class of vulnerabilities that depend on it taking some amount of time between the time a certain resource is validated and the time the resource is used. For further information, please consult the [Wikipedia-article](https://en.wikipedia.org/wiki/Time_of_check_to_time_of_use). In this post, I’ll propose using memory-protection to try and stop some TOCTOU-attacks where the resource to be validated and used is a region of memory. This is a way of stopping TOCTOU by making the memory-region immutable (read <https://www.sans.org/reading-room/whitepapers/securecode/tour-tocttous-1049>)

Basic user-mode pseudo-code:

{% highlight c %}
void be_vulnerable() {
    void* res=load_resource("some-resource");
    if(!validate_resource(res)) {
        printf("Something bad happened");
        exit(-1);
    }
    use_resource(res);
}
{% endhighlight %}

1. validate\_resource is going to start off with basic validation before it goes off to more complex validation
2. validate\_resource is going to take some amount of time
3. There’s some other, in-process, not pictured, thread that can access res while be\_vulnerable is running

If the other thread manages to access res while validate\_resource is running, then be\_vulnerable could end up in a situation where the validation does not match the value of res.

One way to avoid this is to use [mprotect (2)](http://man7.org/linux/man-pages/man2/mprotect.2.html), like this:

{% highlight c linenos %}
void be_vulnerable() {
    void* res=load_resource("some-resource");
    mprotect(res, len, PROT_READ); // len not defined, to simplify example
    if(!validate_resource(res)) {
        printf("Something bad happened");
        exit(-1);
    }
    use_resource(res);
}
{% endhighlight %}

This way, you can ensure that after line 3, you’re looking at a definitive truth. Now, there are some limitations to this, sharing res with other processes makes things a bit more difficult (you have to use O\_RDONLY when calling [shm\_open(3)](https://linux.die.net/man/3/shm_open)) or have res be placed in shared memory, copying the data doesn’t copy the protection. Fun fact: If you want to work on private data, you can create a code-section that directly accesses the data, then map that code-section PROT\_EXEC (without PROT\_READ), this is used in Safari on iOS.

Yeah Martin, that’s fine and dandy, but my problem is with code running on devices, not on the main CPU (yes, this has actually been a problem on some consoles).

Well, dear reader, I might still have a solution for you, it’s called an IOMMU. Yes, it’s an MMU, for IO (devices). Now, for security purposes, the IOMMU can only be controlled from the kernel-mode (or hypervisor if you’re using one), so if you’re in user-mode you’re out of luck (unless OSes add support outside of VMs). With the IOMMU, you can carve out regions of memory you want or don’t want a given domain to access, then assign devices to said domain. For simplicity, you can still have the available regions 1:1-mapped for the domains.

Final fun fact: IOMMUs are used on macOS and Windows to stop plugged in devices from accessing memory bef

Sources:  
<https://stackoverflow.com/questions/7216870/how-to-protect-the-memory-shared-between-processes-in-linux>  
<http://man7.org/linux/man-pages/man2/mprotect.2.html>  
[https://linux.die.net/man/3/shm\_open](https://linux.die.net/man/3/shm_open)