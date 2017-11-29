#### Humanize

I just thought it would be a nice touch to add the built-in **humanize** package. It’s a set of utility functions to add a “human touch” to data.

For example, we can use it to display date and time fields more naturally. Instead of showing the whole date, we can simply show: “2 minutes ago”.

Let’s do it. First, add the `django.contrib.humanize` to the `INSTALLED_APPS`.

**myproject/settings.py**

<figure class="highlight">

    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'django.contrib.humanize',  # <- here

        'widget_tweaks',

        'accounts',
        'boards',
    ]

</figure>

Now we can use it in the templates. First, let’s edit the **topics.html** template:

**templates/topics.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/45521a289677a1a406b4fb743e8141ee)</small>

<figure class="highlight">

    {% extends 'base.html' %}

    {% load humanize %}

    {% block content %}
      <!-- code suppressed -->

      <td>{{ topic.last_updated|naturaltime }}</td>

      <!-- code suppressed -->
    {% endblock %}

</figure>

All we have to do is to load the template tags using `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">load</span> <span class="w"></span> <span class="err">humanize</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` and then apply the template filter: `<span class="p">{</span><span class="err">{</span> <span class="w"></span> <span class="err">topic.last_updated|naturaltime</span> <span class="w"></span> <span class="p">}</span><span class="err">}</span>`

![Humanize](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/humanize.png)

You may add it to other places you like.

* * *