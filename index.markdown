---
layout: default
title: Overview
menus: header
nav_order: 1
has_children: false
---

## Who am I

Discover My experiences and my profile : [Yannick GOBERT](/about/)

## Goals

<ul>
{% for item in site.menus.header %}
  <li class="menu-item-{{ loop.index }}">
    <a href="{{ item.url }}" title="Go to {{ item.title }}">{{ item.title }}</a>
  </li>
{% endfor %}
</ul>

## DevOps platform blue print

![DevOps platform overview](assets/images/devopsplatformblueprint.png)