---
layout: page
title: A blog about technical stuff
tagline:
---

# Hi!

Well, you've found me. I'm Leo Mendoza, and I do things on computers and also on stage. But you're not here to listen to me
hawk my wares, you're here to find out the stuff I discovered at work. So, hooray for that! Anyway, this stuff is
not only documented for the benefit of *you* the viewer, but also so that I have some sort of digital presence online.
Technology, you are fancy!

Here are the posts on this thing. As a newly indoctrinated Markdown user, forgive me if things don't render as we'd like. Also,
one day I'll style this nicely. Unfortunately, today is not that day.

# Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

