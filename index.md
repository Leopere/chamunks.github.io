---
layout: page
title: Chamunks
tagline: chamunks.github.com
---

{% include JB/setup %}

# Recent Posts
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

# My repositories
- [**Dotfiles**](https://chamunks.github.io/.dotfiles/): Configuration files from my main computer.
- [**GNOME Shell Edge-Flipping**](http://chamunks.github.com/gnome-shell-edge-flipping/): GNOME Shell extension that provides reactive screen edges to switch between workspaces.
- [**MultiBoot USB**](https://chamunks.github.io/multibootusb/): Grub2 loopback multiboot pen drive to boot from ISO.
