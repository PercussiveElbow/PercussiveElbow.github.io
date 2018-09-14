---
layout: page
title: Linux Privilege Escalation
icon: fa-angle-up
order: 1
permalink: /linux-privesc/
---

This post will serve as an introduction to Linux escalation techniques, mainly focusing on file/process permissions, but along with some other stuff too. 

You've gained some access to a machine and you need that root shell, but you don't wanna run the risk of crashing the box through running whatever the latest privilege escalation kernel exploit that Tavis Ormandy released this morning. So what do you do?

Intro to File Permissions
-------------------------
On Linux each file is associated with a set of permissions.

I'm logged in as user I have created for these examples, "low". 
I've created a file with the _touch_ command, and then simply listed the contents of the directory.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example1.png)

Okay, so what does all this information mean? Well, lets work our way from right to left.

* Starting off we obviously have the file name. ![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example2.png)

* Next we have the date/time the file was last modified, again straightforward. ![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example3.png)

* Now we have the size of the file. I haven't added any contents to this file, so it's empty (0). ![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example4.png)

### Ownership
Okay, now we're getting to the important stuff. ![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example5.png)

What this section shows is the is ownership of the file. The value on the left is the user owner, while the right is the group owner.

Users on Linux can be part of multiple groups. By default you will be in a group of your own username. ![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example6.png)

As a note, to change file user/group ownership we can simply use _chown_. The syntax for this is:
{% highlight bash %}
chown user:group filename
{% endhighlight %}
If you only need to change the group owner, _changegrp_ is avaliable.
{% highlight bash %}
chngrp group filename
{% endhighlight %}
If you only need to change the user owner, just use chown without the second half.
{% highlight bash %}
chown user filename
{% endhighlight %}

### Permissions
Finally let's look at the leftmost section. These are the permissions. ![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example7.png)


Linux file permissions are divided into three parts
1. **Read (r)**
2. **Write (w)**
3. **Execute (x)**

If none of the above apply, it will simply be displayed as (-).


So this section shows us if we have read, write, execute permissions. So why are these three values repeated three times?

This is because each file has r/w/x permissions associated for it's user(u),group(g) and finally other(o), which is everyone else.



To set these permissions we have a few options. 

To add individual permissions to a user we can use the following syntax. This command adds execute permissions(x) to the user that owns the file.
{% highlight bash %}
chmod u+x filename
{% endhighlight %}
To remove this permission, replace the + with -.
{% highlight bash %}
chmod u-x filename
{% endhighlight %}

If you want to edit the read(r) or write(w) permissions, just replace the x value in the above example with the appropriate character. Similarily, if you want to edit the permission for the group(g) or other(o), just replace the u value.

Alternatively we can do this numerically.
Think of each rwx as three binary bits 

e.g. rw- is 110, which is 6 in decimal.

Using _chmod_ again, we can set the exact rwx permissions for each category (u/g/o) at once. For example to add rw- permissions to user (u), and r-- permissions to group(g) and other(o), use the following command. 

{% highlight bash %}
chmod 644 filename
{% endhighlight %}

To calculate this: 

rw- r-- r--

110 100 100

6   4   4

### More Permissions

The final character on the end will represent whether the item is:
* **a file (-)**
* **directory (d)**
* **symbolic link (l)** 

Meanwhile the presence of a **t** indicates a sticky bit. This means only the file owner, the owner of the directory, or root can rename/delete that file.



There is another type of permission: the SUID/GUID bit (aka SetUID and SetGID permissions). 

These bits tell the system to run the executable as either the owner or group owner of the file. This means we can allow a regular user to run a command as either another user, or even root. If we can exploit this we would be able to escalate privileges.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/example8.png)

We can set these like earlier:
{% highlight bash %}
chmod u+s filename
{% endhighlight %}

SetUID is usually ignored on files containing shebangs (an example being #!/bin/bash you often see at the top of a file) for sensible security reasons, so generally it only works with binaries. For this reason you may see binary files acting as wrappers to call shell commands to work around this.

I'll discuss the exploitation fo these SUID/GUID bits more in the next section. 

Exploiting SUID/GUID
-------------------------

As we now know, these type of files should be very useful for escalating privileges. Before we can attempt to exploit SUID, we need to find some targets via some quick enumeration.

To locate SUID files
{% highlight bash %}
find / -perm -u=s -type f 2>/dev/null
{% endhighlight %}


To locate GUID files
{% highlight bash %}
find / -perm -g=s -type f 2>/dev/null
{% endhighlight %}

Now I'll list some common techniques for exploiting SUID binaries to escalate privileges.


Exploiting SUID/GUID with writable files
-------------------------

Self explanatory: If a SUID bit is set on a file and it's world writable, or at least writable by our user, we can insert whatever code we want into it and use it to escalate privileges.

Exploiting SUID/GUID with Command Injection
-------------------------

If we can inject commands on a binary with SUID enabled we can escalate privileges. This is pretty much the same as regular command injection other than the aspect that we're running code with higher privileges. For this I'm going to use an example from [OWASP's Command Injection Page](https://www.owasp.org/index.php/Command_Injection)

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid_command_injection_1.png)

This binary simply calls the system cat command, but allows us to supply an argument. 

By performing some basic command injection, we can execute the cat command and then another command of our choosing.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid_command_injection_2.png)

If we get command injection like this in a SUID binary, escalating is easy.

Exploiting SUID/GUID with PATH
-------------------------

Lets say we've found a SUID binary. We can see that it's calling the system shell. In this situation we weren't able to exploit it by supplying an argument for command injection because our argument doesn't get passed down to the call to the shell, so what is another method to exploit this?

This binary calls the harmless _id_ command. One thing that this binary doesn't do however, is list the full path of _id_. This is *great* for priv esc, because if we have the ability to edit our own path, we can redirect the execution to a binary we supply ourselves. (Note: this works for any binary really, id is just an example.)

As we can see in this example file, we have a SUID binary that calls the system shell with the "id" command. 

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid1.png)

I've provided the source code above, but in the case we can often identify the command from the binary with _strings filename_ or some reverse engineering.

Running the compiled file runs the id command as expected, showing us it's running as root.
![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid2.png)

I quickly make an exploit file and compile it.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid5.png)


Now remember that the call to id does not list the full path. I can exploit this by editing my own path, to point to another directory. When the shell calls for a file like this, it looks through all the locations on the PATH. When it obtains the first instance of a file matching the name, it will attempt to run it.

Therefore by setting my own path to point to the directory I'm currently in first (/home/low/suid) and then renaming my exploit binary to _id_, execution can be redirected to the evil _id_ rather than the real one.
![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid3.png)

As you can see calling the binary runs the evil id and the exploit code within.


![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid4.png)


Running _which id_ after we edit the path shows us which binary we'll run if we call _id_, this is the evil one.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid6.png)

Before editing the path we can see _id_ points to the correct location _/usr/bin/id_.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/suid7.png)


Exploiting SUID/GUID with Environment Variables
-------------------------
This is the same logic as exploiting using the PATH method. Just set the environment variable instead of your path

Exploiting File Capabilities
-------------------------

Following on from what we now know about SUID/GUID files on Linux, we can see how enabling a file to have one of those bits provides a **lot** of power, and so we need to carefully consider where we use this feature. If a file has a SUID bit to run as root, it has the power to do everything that root can.

So you may be thinking something along the lines of this; my script/program *needs* root to do something specific that can't be fixed by adding it to a group and giving it appropriate file permissions. What if I need to bind to a port under 1024 or set the system time? Do I really need to have to grant my script root privileges, allowing it to do _anything_ if it were to be exploited?

Well the answer is no. Over the past few years work has been implemented within the kernel to try to break down the need to run something as a fully privileged process. This feature is called [Capabilities](http://man7.org/linux/man-pages/man7/capabilities.7.html).

Think of your smartphone, unless you're a luddite, amish or Richard Stallman*. When you install an application from the App/Play store, you're granted a list of permissions before installing it. You may even be prompted when you're using it for some specific intrusive permission, such as location detection. 

This is a little bit like what capabilities are for. Rather than granting a file full root privileges when ran with a SUID bit or sudo, we can instead give it specific powers.

So following on from an example earlier, I want to bind a program, let's imagine it's some janky custom webapp I've created to a port under 1024, in this case it would likely be 80. My choices are: 
1. Run this program as root to bind the port correctly.
2. Use the _CAP_NET_BIND_SERVICE_ capability to grant the specific ability to bind to the port, while not providing any other root powers.

So let's play with an example.
I'll call Python's inbuilt SimpleHTTPServer on port 80.

When I run that with my non root user though, we get a permission denied error.
![Example]({{ site.url }}/assets/images/posts/linux_file_perms/cap1.png)

To add the capability to bind to this port, use the following command. You of course need root permissions for this.
{% highlight bash %}
setcap cap_net_bind_service=+ep filename
{% endhighlight %}
![Example]({{ site.url }}/assets/images/posts/linux_file_perms/cap2.png)

As we can see, we can now bind to port 80 successfully.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/cap3.png)

And use the following command to view capabilities.
{% highlight bash %}
getcap filename
{% endhighlight %}
![Example]({{ site.url }}/assets/images/posts/linux_file_perms/cap4.png)

Now that we understand what capabilities are, how can we use them to our advantage?

Exploiting programs with capabilities should be the same thought process as that of SUID executables, except it's a little more tricky as we're limited in what we can do. 

I recommended checking if the file you're exploiting first has capabilities with _getcap_, and then checking the link above for what that capability is. If you're lucky with which capabilities are present, and the program is designed in a way where you can exploit that somehow, you might have a path to privilege escalation.

### Example.

The _CAP_DAC_OVERRIDE_ capability bypasses file r/w/x permission checks. If a program has this capability enabled, we can essentially read/write/execute any file when it is ran.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/cap5.png)

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/cap6.png)

Take a look at the above script. This is used to backup a user's work to another location. Currently (without capabilities) it can't overwrite a file _cant_overwrite_me_ that is owned by root. (I know the error says segmentation fault but that's because I'm too lazy to handle errors properly, trust me it's permissions!)

Now let's assume we found the same script but it has the _CAP_DAC_OVERRIDE_ capability added, maybe a sysadmin enabled it without proper consideration to allow the user to backup to a location that they don't have write access to. Can you see how we might exploit it to gain privs?

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/cap7.png)

We essentially have write access to anywhere on the machine. We can do any number of nasty things with this, including adding a user to _/etc/passwd_ by overwriting it, or adding an SSH key to _/etc/ssh_.

Hopefully this can come in useful for you, while usage of capabilities seems to be much rarer than SUID/GUID, it's still a useful technique to keep in mind.


Exploiting Sudo
-------------------------

Before this section I'll just briefly say all the SUID methods detailed before pretty much still apply for sudo usage. So if you find a script you can sudo with command injection, no full path listing or environment variable injection, knock yourself out!

The sudo command allows us to run a command as root. 

We can check if we're able to run any commands with sudo for our current user with
{% highlight bash %}
sudo -l
{% endhighlight %}

This is a *very* good way to escalate privileges, as it's a common misconfiguration. 

As an common example of this, my low user has access to sudo a text editor (vim).

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/sudo1.png)

Using 'sudo vi' will allow us to read/write anywhere arbitrarily, but not only that we can even drop down to a shell with :shell. ![Example]({{ site.url }}/assets/images/posts/linux_file_perms/sudo2.png)

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/sudo3.png)

Many common binaries can be used in similar ways to exploit and gain privileges. 
A few are listed below.

* *cp*,*mv*: You can write anywhere, so simply move or copy over _/etc/passwd_ to add a user or a SSH key. 
* *less*,*more*,*man* Just call _!bash_

If you have sudo access to any language interpreter (such as python, perl or ruby), you've already pretty much won.

If you don't have sudo access to a language interpreter, but you do have access to a language package manager (think Python's Pip, Ruby's Bundler, NodeJS's NPM) etc, you can sudo install a custom package containing your executable code.

As there are too many programs that could be running under sudo, it's good to know how to enumerate them yourself to know if you can abuse it and hopefully execute code. To do so you should check any extra arguments ("program_name -h") or checking the program with man program_name.

Perhaps you find that you have sudo avaliable on a script or binary. Do you have write permissions on that script? Again this is a quick win. If not are you able to supply arguments when running it?




Exploiting Cron Jobs
-------------------------

If we can find any cron jobs that run with higher privileges than us we may be able to escalate privileges through a misconfiguration
{% highlight bash %}
cat /etc/*cron*
{% endhighlight %}

If we have write access to a file run in one of these cron jobs, we can simply inject whatever code we want and escalate.

Exploiting Wildcards
-------------------------
So we've found a SUID executable, a cronjob that runs as root, or a script we have sudo permissions to run. We don't have write access to it. We can't inject commands using it. Binary paths are all properly configured. But we notice that the command being ran has an asterix (*) present in the command. 

Due to the way the shell expands the asterix, we can actually use this to inject arguments. While this isn't a guaranteed way to escalate privileges, if we're lucky the binary being called has an argument that allows us code execution.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/wildcard1.png)

If we don't have access to the source code, we can use _strings_ to see if we can find anything that looks like a shell command. 


![Example]({{ site.url }}/assets/images/posts/linux_file_perms/wildcard2.png)

When running this SUID executable we can see that we're able to back up a file only readable by root.

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/wildcard3.png)

![Example]({{ site.url }}/assets/images/posts/linux_file_perms/wildcard4.png)


![Example]({{ site.url }}/assets/images/posts/linux_file_perms/wildcard5.png)



https://www.exploit-db.com/papers/33930/


Programs running as root
-------------------------
{% highlight bash %}
ps aux | grep root
{% endhighlight %}

If we're able to exploit one of these services running as root we can gain root ourselves. 


Bound network ports
-------------------------
{% highlight bash %}
ps aux | grep root
{% endhighlight %}

*I joke I love you Richard
