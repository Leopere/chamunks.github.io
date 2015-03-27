---
layout: page
title: Agus Lopez
tagline: aguslr.github.com
---
{% include JB/setup %}

## Recent Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## My repositories

* [**GNOME Shell Edge-Flipping**](http://aguslr.github.com/gnome-shell-edge-flipping/): GNOME Shell extension that provides reactive screen edges to switch between workspaces.
* [**MultiBoot USB**](https://aguslr.github.io/multibootusb/): Grub2 loopback multiboot pen drive to boot from ISO.
* [**Dotfiles**](https://github.com/aguslr/.dotfiles): Configuration files from my main computer.
