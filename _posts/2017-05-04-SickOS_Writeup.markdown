---
layout: post
title:  "SickOS 1.1 writeup [DONE]"
date:   2017-05-04 13:33:27 +0100
categories: jekyll update
---

[Download this box on VulnHub](https://www.vulnhub.com/entry/sickos-11,132/)

Nmap
![Nmap screenshot]({{ site.url }}/assets/sickos/sickos11/nmap.PNG)

SSH on port 22.
HTTP proxy on 3128.

I added the proxy to my firefox settings then loaded up this:

![Site screenshot]({{ site.url }}/assets/sickos/sickos11/site.PNG)

Running nikto through proxy:
![Nikto screenshot]({{ site.url }}/assets/sickos/sickos11/nikto.PNG)

It finds tons of stuff, including possible shellshock vulnerability, but lets keep going.

Checking robots reveals wolfcms running on site.
![Robots screenshot]({{ site.url }}/assets/sickos/sickos11/robotstxt.PNG)

Checking wolfcm and source, then tried /admin/ for login.
![Wolf cms screenshot]({{ site.url }}/assets/sickos/sickos11/wolfcms.PNG)

Tried common combinations. admin/admin works, no bruteforce needed.
![Wolf cms screenshot]({{ site.url }}/assets/sickos/sickos11/wolfcmsadmin.PNG)

I generate a php reverse shell in msfvenom, then upload it using the file upload option within wolfcms. Netcat doesn't seem to catch the reverse shell correctly however.
![reverse shell screenshot]({{ site.url }}/assets/sickos/sickos11/reverseshell.png)

I load up reverse tcp in metasploit instead and reload the page, it connects, I've got a working meterpreter shell now.
![meterpreter screenshot]({{ site.url }}/assets/sickos/sickos11/meterpreter.PNG)

Finding some accounts in /etc/passwd
![etcpasswd screenshot]({{ site.url }}/assets/sickos/sickos11/etcpasswd.PNG)

I search around the box for anything interesting and find some credentials in config.php. I was going to attempt to look for the mysql service but I thought I'd try SSHing with those credentials.
![configphp screenshot]({{ site.url }}/assets/sickos/sickos11/configphp.PNG)

Trying some of the accounts from earlier with the password found in config.php. SSHing with sickos account works.
![SSH screenshot]({{ site.url }}/assets/sickos/sickos11/ssh.PNG)

I sudo su, it works, checking /root/
![Rooted screenshot]({{ site.url }}/assets/sickos/sickos11/rooted.PNG)
All done.

I think there's probably a few ways to get into this box, nikto found tons of possible vectors, so maybe I will attempt this without the wolfcms stuff at another time.

Takeaways: learned how to proxy some common pentest tools.


