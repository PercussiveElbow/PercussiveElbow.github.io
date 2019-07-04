---
layout: post
title:  "HTB Poison Walkthrough"
date:   2018-09-09 12:00:00 +0100
categories: walkthroughs
description: "/htb/"
permalink: /htb-poison/
---

I've just finished [NoxCTF](https://ctf18.noxale.com/) yesterday so I thought I'd try to do a quick writeup of Poison on HackTheBox. This is a pretty easy box, user in particular is straightforward, although PE can trip you up if you overthink it.

The initial nmap scan revealed four ports opened. The webserver seemed like the most likely target immediately, but a VNC service running was a little bit unusual. I made note of it and continued enumeration of the webserver.

{% highlight c %}
Nmap scan report for 10.10.10.84
Host is up (0.043s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (EdDSA)
80/tcp   open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
5900/tcp open  vnc     VNC (protocol 3.8)
| vnc-info: 
|   Protocol version: 3.8
|   Security types: 
|     VNC Authentication (2)
|     Tight (16)
|   Tight auth subtypes: 
|_    STDV VNCAUTH_ (2)
6000/tcp open  X11     (access denied)
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd
{% endhighlight %}

Visiting the webserver in my browser revealed a page that immediately stuck out as being the likely exploitation path. Using this functionality you can run a few PHP files on the server. I checked these out one by one manually.

![Web]({{ site.url }}/assets/images/HTB/poison/web1.png)

_listfiles.php_ showed a few file names. _pwdbackup.txt_ in particular seemed to be either a bit of a troll or the file we would need to obtain user.

![Web]({{ site.url }}/assets/images/HTB/poison/web2.png)

I noticed by changing the _file_ parameter in the URL that LFI was possible. I retrieved the password backup which consisted of what I thought may be a VNC or SSH password, encoded a ton of times with base64. You can either simply loop base64 decode in in bash/python or if you're really lazy, just mash it in an online decoder a few times.
![Web]({{ site.url }}/assets/images/HTB/poison/web3.png)

The password ends up being:
**_Charix!2#4%6&8(0_**

This is great that we've got a password so quick, but it's kind of useless without knowing the username. However we can change the file we're viewing with LFI to _/etc/passwd_ to get a list of users. The non standard user that stuck out, **"charix"** had access to a shell.

![Web]({{ site.url }}/assets/images/HTB/poison/web4.png)

This was the same username as part of the password, so they are likely to be the correct pair.
We can attempt to use these credentials.

Luckily they work straight away with SSH.

![User]({{ site.url }}/assets/images/HTB/poison/user1.png)

Privilege escalation isn't too hard on this machine either, but if you skip over your very basic first steps enumeration you could miss it for a while.
Checking in Charix's home directory we can see a _"secret.zip"_.

I transferred it back to my attacking machine to take a closer look at it, it was password protected, but the password is just the same one we already found earlier. 

The contents of the extracted "secret" file confused me for a while. I decided to try and use the VNC service from earlier using Charix's password. That didn't work, but then I realized what the file was, it was the VNC password file itself! I wasn't able to use it on the already exposed VNC service, but by reverse forwarding out the correct port from Poison to my localhost via SSH I could access the service on my local machine.

{% highlight c %}
ssh -R 5901:localhost:5901 charix@10.10.10.84
{% endhighlight %}

Note: I forgot to save the exact command I used but I'm pretty sure the above should work.


![PE]({{ site.url }}/assets/images/HTB/poison/pe1.png)

Finally authenticating the (very slow) VNC service we can see that we gained root access to this machine.
![Root]({{ site.url }}/assets/images/HTB/poison/root.png)
