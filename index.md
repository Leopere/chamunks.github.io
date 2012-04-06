---
layout: page
title: Agus Lopez
tagline: aguslr.github.com
---
{% include JB/setup %}

## Update Author Attributes

In `_config.yml` remember to specify your own data:
    
    title : My Blog =)
    
    author :
      name : Name Lastname
      email : blah@email.test
      github : username
      twitter : username

The theme should reference these variables whenever needed.
    
## Recent Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## My repositories

* [**Gnome-shell Edge-Flipping**](http://aguslr.github.com/gnome-shell-edge-flipping/): Gnome-shell extension that provides reactive screen edges to switch between different workspaces.
* [**Dotfiles**](https://github.com/aguslr/.dotfiles): Configuration files from my main computer.
* [**Scripts**](https://github.com/aguslr/Scripts): Scripts made by me or tailored to my needs.
