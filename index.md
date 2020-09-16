---
layout: default
---

# $ cat about.txt
{:id="about"}

This site is under construction

I like bad movies and all things *sec. Pentester/noob

You can find me hanging around on various infosec IRC/Discord channels.

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

