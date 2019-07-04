---
layout: post
title: Nmap Cheatsheet
icon: fa-map
order: 1
permalink: /nmap-cheatsheet/
categories: cheatsheets
---

Scan Targets
------------------

Single (IP)
{% highlight bash %}
nmap 192.168.0.1
{% endhighlight %}

Single (Domain)
{% highlight bash %}
nmap example.com
{% endhighlight %}

Multiple
{% highlight bash %}
nmap 192.168.0.1 192.168.0.2
{% endhighlight %}

Range
{% highlight bash %}
nmap 192.168.0.1-254
{% endhighlight %}

Exclude individual IPs
{% highlight bash %}
nmap 192.168.0.1-254 --exclude 192.168.0.100
{% endhighlight %}

Scan hosts from a file
{% highlight bash %}
nmap -iL filename
{% endhighlight %}

Exclude IPs from a file
{% highlight bash %}
nmap 192.168.0.1-254 --excludefile filename
{% endhighlight %}

Scan Ports
------------------
Single port
{% highlight bash %}
nmap 192.168.0.1-254 -p 80
{% endhighlight %}

Multiple ports
{% highlight bash %}
nmap 192.168.0.1-254 -p 22,80,443,
{% endhighlight %}

Port range
{% highlight bash %}
nmap 192.168.0.1-254 -p 1-100
{% endhighlight %}

All ports
{% highlight bash %}
nmap 192.168.0.1-254 -p-
{% endhighlight %}

Only the top 100 most common ports
{% highlight bash %}
nmap 192.168.0.1-254 -F
{% endhighlight %}



Scan Types
------------------
SYN "Stealth" Scan
{% highlight bash %}
nmap 192.168.0.1 -sS 
{% endhighlight %}

TCP Connect Scan
{% highlight bash %}
nmap 192.168.0.1 -sT
{% endhighlight %}

UDP Scan
{% highlight bash %}
nmap 192.168.0.1 -sU
{% endhighlight %}

Ping Scan (won't scan ports)
{% highlight bash %}
nmap 192.168.0.1 -sn
{% endhighlight %}

Services and Versions
------------------
Identifying service versions
{% highlight bash %}
nmap 192.168.0.1 -sV 
{% endhighlight %}

Identifying Operating Systems
{% highlight bash %}
nmap 192.168.0.1 -O
{% endhighlight %}

Timing
------------------
"Paranoid"
{% highlight bash %}
nmap 192.168.0.1 -T0
{% endhighlight %}

"Sneaky"
{% highlight bash %}
nmap 192.168.0.1 -T1
{% endhighlight %}

"Polite"
{% highlight bash %}
nmap 192.168.0.1 -T2
{% endhighlight %}

Default
{% highlight bash %}
nmap 192.168.0.1 -T3
{% endhighlight %}

"Aggressive"
{% highlight bash %}
nmap 192.168.0.1 -T4
{% endhighlight %}

"Insane"
{% highlight bash %}
nmap 192.168.0.1 -T5
{% endhighlight %}



Scan Output
------------------
Default output
{% highlight bash %}
nmap 192.168.0.1 -oN filename
{% endhighlight %}

Greppable output
{% highlight bash %}
nmap 192.168.0.1 -oG filename
{% endhighlight %}

XML output
{% highlight bash %}
nmap 192.168.0.1 -oX filename
{% endhighlight %}

All above outputs
{% highlight bash %}
nmap 192.168.0.1 -oA filename
{% endhighlight %}


Scan Display
------------------
Increase scan output verbosity
{% highlight bash %}
nmap 192.168.0.1 -v
{% endhighlight %}

Increase scan output verbosity (more)
{% highlight bash %}
nmap 192.168.0.1 -vv
{% endhighlight %}


Scripts
------------------

Default scripts
{% highlight bash %}
nmap 192.168.0.1 -sC
{% endhighlight %}

Individual script
{% highlight bash %}
nmap 192.168.0.1 -script scriptname
{% endhighlight %}

Individual script with arguments
{% highlight bash %}
nmap 192.168.0.1 -script scriptname --script-args scriptname.argumentname=value
{% endhighlight %}


Multiple scripts with same start of filename
{% highlight bash %}
nmap 192.168.0.1 -script smb-vuln-*
{% endhighlight %}

List scripts
{% highlight bash %}
ls /usr/share/nmap/scripts
{% endhighlight %}

