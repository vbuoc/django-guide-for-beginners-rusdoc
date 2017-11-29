#### Gravatar

A very easy way to add user profile pictures is by using [Gravatar](https://gravatar.com).

Inside the **boards/templatetags** folder, create a new file named **gravatar.py**:

**boards/templatetags/gravatar.py**

<figure class="highlight">

    import hashlib
    from urllib.parse import urlencode

    from django import template
    from django.conf import settings

    register = template.Library()

    @register.filter
    def gravatar(user):
        email = user.email.lower().encode('utf-8')
        default = 'mm'
        size = 256
        url = 'https://www.gravatar.com/avatar/{md5}?{params}'.format(
            md5=hashlib.md5(email).hexdigest(),
            params=urlencode({'d': default, 's': str(size)})
        )
        return url

</figure>

Basically I’m using the [code snippet they provide](https://fi.gravatar.com/site/implement/images/python/). I just adapted it to work with Python 3.

Great, now we can load it in our template, just like we did with the Humanize template filter:

**templates/topic_posts.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/23d5c5bc9e6c7ac94506a2660a61012c)</small>

<figure class="highlight">

    {% extends 'base.html' %}

    {% load gravatar %}

    {% block content %}
      <!-- code suppressed -->

      <img src="{{ post.created_by|gravatar }}" alt="{{ post.created_by.username }}" class="w-100 rounded">

      <!-- code suppressed -->
    {% endblock %}

</figure>

![Gravatar](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/gravatar.png)

* * *