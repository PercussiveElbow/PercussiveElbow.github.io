---
layout: default
---

# $ cat about.txt
{:id="about"}

I like bad movies and all things *sec. Pentester/noob

You can find me hanging around on various infosec IRC/Discord channels.

<ul>
<li><a href="https://www.github.com/percussiveelbow">Github</a></li>
<li><a href="https://www.hackthebox.eu/profile/55538">HackTheBox</a></li>
<li><a href="mailto:PercussiveElbow@protonmail.com">Email</a></li>
</ul>

Big shout out to [LampiaoSec](https://github.com/lampiaosec) for the Jekyll theme and saving your eyes from my web design skills.

# $ cat projects.txt
 
{:id="projects"}

I have a terrible habit of starting projects and not finishing them. I do try to open source these though.
<ul>
{% for project in site.categories.projects %}
<li><a href="{{ project.link }}">{{ project.title }}</a> - {{ project.description }} <i>{{project.description2}}</i> </li>
{% endfor %}
</ul>

# cat tutorials.txt
{:id="tutorials"}

<ul>
{% for tutorial in site.categories.tutorials %}

<li><a href="{{ tutorial.url }}" title="{{ tutorial.description }}">{{ tutorial.title }}</a></li>

{% endfor %}
</ul>

# $ ls -R /posts
{:id="posts"}

<ul>
{% for post in site.categories.posts %}

<li><a href="{{ post.url }}" title="{{ post.description }}">/{{post.date | date: '20%y/%B'}}/{{ post.title }}</a></li>

{% endfor %}
</ul>

# $ ls -R /ctf_walkthroughs
{:id="walkthroughs"}

<ul>
{% for walkthrough in site.categories.walkthroughs %}

<li><a href="{{ walkthrough.url }}" title="{{ walkthrough.description }}">{{walkthrough.description}}{{ walkthrough.title }}</a></li>

{% endfor %}
</ul>


# $ ls /cheatsheets
{:id="cheatsheets"}
Like man pages, but worse.

<ul>
{% for cheatsheet in site.categories.cheatsheets %}

<li><a href="{{ cheatsheet.url }}" title="{{ cheatsheet.description }}">{{ cheatsheet.title }}</a></li>

{% endfor %}
</ul>
