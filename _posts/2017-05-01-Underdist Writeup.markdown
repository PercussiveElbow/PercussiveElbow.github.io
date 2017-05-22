---
layout: post
title:  "Underdist3 writeup [IN PROGRESS]"
date:   2017-05-01 13:33:27 +0100
categories: jekyll update
---

[Download this box on VulnHub](https://www.vulnhub.com/entry/underdist-3,108/)

Starting with running netdiscover to find the box IP.
![Netdiscover screenshot]({{ site.url }}/assets/underdist/netdiscover.PNG)

Then running an nmap scap. Things of note:
Webserver on port 80.
SSH on 22.
SMTP on 25 (Interesting.)

![Nmap screenshot]({{ site.url }}/assets/underdist/nmap.PNG)

Taking a look at the site:

![site screenshot]({{ site.url }}/assets/underdist/site.PNG)

Site Source:

![sitesource screenshot]({{ site.url }}/assets/underdist/sitesource.PNG)

That php looks very interesting. First I'll run Dirb though:

![Dirb screenshot]({{ site.url }}/assets/underdist/dirb.PNG)

I also checked apache version, no CVEs that will be useful.

That string after php?a= looks like base64 to me, so lets try decoding.
![Decode screenshot]({{ site.url }}/assets/underdist/decode.PNG)

Seems to refer to a text file. 
I tried LFI, then realised I'd need to base64 it. After some fumbling around trying to work out the directory structure, I moved up enough dirs to get /etc/passwd.

![LFI screenshot]({{ site.url }}/assets/underdist/LFI.PNG)

I paused here for quite a while trying to work out how to get further, before going back to the start and realising I'd missed attempting anything with the SMTP. I loaded up metasploit and ran smtp_enum, then ran it again with a new worldlist consisting of the users I found in /etc/passwd. I got some results:

![SMTP ENUM screenshot]({{ site.url }}/assets/underdist/smtp_enum.PNG)

After that I tried mailing some php code to the box and opening it with the LFI to see if I can browse dir structure a little easier and possibly set up reverse shell
tbc..

