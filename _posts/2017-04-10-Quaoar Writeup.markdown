---
layout: post
title:  "Quaoar writeup"
date:   2017-04-12 23:33:27 +0100
categories: jekyll update
---
Let's start with running {% highlight bash %}
netdiscover 
{% endhighlight %}
![Netdiscover screenshot]({{ site.url }}/assets/netdiscover.png)

to find the box we want to target, in this case 192.168.56.101.

Then we'll take a look at what is running on the box with 
{% highlight bash %}
nmap -sV -A 192.168.0.101 
{% endhighlight %}
![Nmap screenshot]({{ site.url }}/assets/nmap.png)

This gives us a pretty good place to start now, we've got an SSH service running on port 22.Tried brute forcing it briefly with hydra and rockyou.txt but no luck as expected (but worth trying I guess).
We've got a web server running on port 80, this is going to be our main area to focus on I think.
Then some final stuff like the kernel version, bit irrelevant now but might be useful place to look for priv escalation later on.

Anyway spinning up http:/192.168.56.101 in the browser, you can see this:
![Web screenshot]({{ site.url }}/assets/web.png)

Nothing special in the source code that I can see, so lets click.
![Web screenshot]({{ site.url }}/assets/web1.png)

Nice. I can't see anything standing out to me so now let's try running 
{% highlight bash %}
nikto -host http://192.168.56.101
{% endhighlight %}
to get some more info about the webserver. 
![Nikto screenshot]({{ site.url }}/assets/nikto.png)

Robots.txt will be worth checking out. Apache is quite outdated, but there doesn't seem to be any CVEs that will help us gain a shell. Also notice wordpress, that is definetly a soft target here.
Before we do anything else let's check out robots.txt:

![Robots screenshot]({{ site.url }}/assets/robots.png)

Nice, and it seems to hint us toward wordpress too.

For now lets keep going gathering a bit of info, running
{% highlight bash %}
dirb 
{% endhighlight %}
![Dirb screenshot]({{ site.url }}/assets/dirb.png)

spits out tons of directories, nearly all of them seem related to wordpress in one way or another. I noticed /upload/ too, thinking this may be a possible vector but seems to just be related to wordpress again.

Think we've enough info to try something now, so lets try
{% highlight bash %}
wpscan 
{% endhighlight %}
![Wpscan screenshot]({{ site.url }}/assets/wpscan.png)

Which is a great tool. It notices a few vulns. Lets try enumerating users, we get two results. Then lets try bruteforcing admin to start, it  takes seconds.
![Wpscan1 screenshot]({{ site.url }}/assets/wpscan1.png)

Seems like they've got the default password "admin" still set, great, should have probably tried that one to start with manually. We're in:

![WP screenshot]({{ site.url }}/assets/wp.png)

Lets have a poke around themes and plugins, good place to see if we can insert a php reverse shell. I'm pretty sure there's a metasploit module that would do it automatically right now, but I want to try and not rely on it too much.
![WP screenshot]({{ site.url }}/assets/wp1.png)

I generate a PHP reverse shell in 
{% highlight bash %}
msfvenom 
{% endhighlight %}
but then spent quite a bit of time trying to get it to work by putting it in the theme and loading a few of the pages referenced. I was probably missing something obvious but after giving up on fiddling around with the themes I got it to work by pasting it in the Mail Masta plugin (after enabling it), saving and loading a couple of times.
![WP screenshot]({{ site.url }}/assets/wp2.png)

By "working" I meant sort of working, running 
{% highlight bash %}
nv -lvp 4444 
{% endhighlight %} (where 4444 is the port the reverse shell is connecting via) does connect to my netcat instance, but it seems to hang/die/just generally not work.
![Netcat screenshot]({{ site.url }}/assets/netcat.png)
 This is the bit where I was completely stuck and had to cheat, I saw one of the walkthroughs suggesting a similar issue and tried to get a meterpreter 
session with the reverse_tcp module in metasploit. This worked first time!

![Meterpreter screenshot]({{ site.url }}/assets/meterpreter.png)

We have a shell. 
Lets take a look where we are:

![Meterpreter screenshot]({{ site.url }}/assets/meterpreter1.png)

Eventually after poking around for a few minutes I find a wp-config.php file with some tasty credentials:
![Meterpreter screenshot]({{ site.url }}/assets/meterpreter2.png)

I had a look at what was running, trying to work out if there were any processes I could try to use to gain root, but I gave up pretty quickly, luckily root ssh was enabled so lets try with the credentials we found:

![SSH screenshot]({{ site.url }}/assets/ssh.png)

Oh yes.

![SSH screenshot]({{ site.url }}/assets/ssh1.png)

And a flag for good luck.

Then after more fiddling around through files before stumbling upon the other flag. (I really should have just searched for grep flag or something..

![SSH screenshot]({{ site.url }}/assets/ssh2.png)

Anyway this was a pretty fun box to to start writing up and I'd recommend it to other beginners. Take-aways for me are probably to try obvious things straight away rather than trying to over think the situation (and figure out why netcat hates me)

