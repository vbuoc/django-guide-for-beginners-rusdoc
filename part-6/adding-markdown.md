#### Adding Markdown

Let’s improve the user experience by adding Markdown to our text areas. You will see it’s very easy and simple.

First, let’s install a library called **Python-Markdown**:

```

    pip install markdown

```

We can add a new method to the **Post** model:

**boards/models.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/caa24fcf2b66903617ebbb41337d3d3d#file-models-py-L46)</small>

```

    from django.db import models
    from django.utils.html import mark_safe
    from markdown import markdown

    class Post(models.Model):
        # ...

        def get_message_as_markdown(self):
            return mark_safe(markdown(self.message, safe_mode='escape'))

```

Here we are dealing with user input, so we must take care. When using the `markdown` function, we are instructing it to escape the special characters first and then parse the markdown tags. After that, we mark the output string as safe to be used in the template.

Now in the templates **topic_posts.html** and **reply_topic.html** just change from:

```

    {{ post.message }}

```

To:

```

    {{ post.get_message_as_markdown }}

```

From now on the users can already use markdown in the posts:

![Markdown](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/markdown-1.png)

![Markdown](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/markdown-2.png)

##### Markdown Editor

We can also add a very cool Markdown editor called [SimpleMD](https://simplemde.com).

Either download the JavaScript library or use their CDN:

```

    <link rel="stylesheet" href="https://cdn.jsdelivr.net/simplemde/latest/simplemde.min.css">
    <script src="https://cdn.jsdelivr.net/simplemde/latest/simplemde.min.js"></script>

```

Now edit the **base.html** to make space for extra JavaScripts:

**templates/base.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/5a7ad8e7eae88d64f62fec82d037b168#file-base-html-L57)</small>

```

        <script src="{% static 'js/jquery-3.2.1.min.js' %}"></script>
        <script src="{% static 'js/popper.min.js' %}"></script>
        <script src="{% static 'js/bootstrap.min.js' %}"></script>
        {% block javascript %}{% endblock %}  <!-- Add this empty block here! -->

```

First edit the **reply_topic.html** template:

**templates/reply_topic.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/fb63bb7530690d62787c3ed8b7e15241)</small>

```

    {% extends 'base.html' %}

    {% load static %}

    {% block title %}Post a reply{% endblock %}

    {% block stylesheet %}
      <link rel="stylesheet" href="{% static 'css/simplemde.min.css' %}">
    {% endblock %}

    {% block javascript %}
      <script src="{% static 'js/simplemde.min.js' %}"></script>
      <script>
        var simplemde = new SimpleMDE();
      </script>
    {% endblock %}

```

By default, this plugin will transform the first text area it finds into a markdown editor. So just that code should be enough:

![Editor](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/editor-1.png)

Now do the same thing with the **edit_post.html** template:

**templates/edit_post.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/ee9d8c91888b0bc60013b8f037bae7bb)</small>

```

    {% extends 'base.html' %}

    {% load static %}

    {% block title %}Edit post{% endblock %}

    {% block stylesheet %}
      <link rel="stylesheet" href="{% static 'css/simplemde.min.css' %}">
    {% endblock %}

    {% block javascript %}
      <script src="{% static 'js/simplemde.min.js' %}"></script>
      <script>
        var simplemde = new SimpleMDE();
      </script>
    {% endblock %}

```

![Editor](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/editor-2.png)

* * *