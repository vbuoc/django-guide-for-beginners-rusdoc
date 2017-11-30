Until now we’ve been copying and pasting HTML repeating several parts of the HTML document, which is not very sustainable in the long run. It’s also a bad practice.

In this section we are going to refactor our HTML templates, creating a **master page** and only adding the unique part for each template.

Create a new file named **base.html** in the **templates** folder:

**templates/base.html**

```

    {% load static %}<!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>{% block title %}Django Boards{% endblock %}</title>
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
      </head>
      <body>
        <div class="container">
          <ol class="breadcrumb my-4">
            {% block breadcrumb %}
            {% endblock %}
          </ol>
          {% block content %}
          {% endblock %}
        </div>
      </body>
    </html>

```

This is going to be our master page. Every template we create, is going to **extend** this special template. Observe now we introduced the `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` tag. It is used to reserve a space in the template, which a “child” template (which extends the master page) can insert code and HTML within that space.

In the case of the `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">title</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` we are also setting a default value, which is “Django Boards.” It will be used if we don’t set a value for the `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">title</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` in a child template.

Now let’s refactor our two templates: **home.html** and **topics.html**.

**templates/home.html**

```

    {% extends 'base.html' %}

    {% block breadcrumb %}
      <li class="breadcrumb-item active">Boards</li>
    {% endblock %}

    {% block content %}
      <table class="table">
        <thead class="thead-inverse">
          <tr>
            <th>Board</th>
            <th>Posts</th>
            <th>Topics</th>
            <th>Last Post</th>
          </tr>
        </thead>
        <tbody>
          {% for board in boards %}
            <tr>
              <td>
                <a href="{% url 'board_topics' board.pk %}">{{ board.name }}</a>
                <small class="text-muted d-block">{{ board.description }}</small>
              </td>
              <td class="align-middle">0</td>
              <td class="align-middle">0</td>
              <td></td>
            </tr>
          {% endfor %}
        </tbody>
      </table>
    {% endblock %}

```

The first line in the **home.html** template is `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">extends</span> <span class="w"></span> <span class="err">'base.html'</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>`. This tag is telling Django to use the **base.html** template as a master page. After that, we are using the the _blocks_ to put the unique content of the page.

**templates/topics.html**

```

    {% extends 'base.html' %}

    {% block title %}
      {{ board.name }} - {{ block.super }}
    {% endblock %}

    {% block breadcrumb %}
      <li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
      <li class="breadcrumb-item active">{{ board.name }}</li>
    {% endblock %}

    {% block content %}
        <!-- just leaving it empty for now. we will add core here soon. -->
    {% endblock %}

```

In the **topics.html** template, we are changing the `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">title</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` default value. Notice that we can reuse the default value of the block by calling `<span class="p">{</span><span class="err">{</span> <span class="w"></span> <span class="err">block.super</span> <span class="w"></span> <span class="p">}</span><span class="err">}</span>`. So here we are playing with the website title, which we defined in the **base.html** as “Django Boards.” So for the “Python” board page, the title will be “Python - Django Boards,” for the “Random” board the title will be “Random - Django Boards.”

Now let’s run the tests and see we didn’t break anything:

```

    python manage.py test

```

```

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    .......
    ----------------------------------------------------------------------
    Ran 7 tests in 0.067s

    OK
    Destroying test database for alias 'default'...

```

Great! Everything is looking good.

Now that we have the **base.html** template, we can easily add a top bar with a menu:

**templates/base.html**

```

    {% load static %}<!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>{% block title %}Django Boards{% endblock %}</title>
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
      </head>
      <body>

        <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
          <div class="container">
            <a class="navbar-brand" href="{% url 'home' %}">Django Boards</a>
          </div>
        </nav>

        <div class="container">
          <ol class="breadcrumb my-4">
            {% block breadcrumb %}
            {% endblock %}
          </ol>
          {% block content %}
          {% endblock %}
        </div>
      </body>
    </html>

```

![Django Boards Header](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/django-boards-header-1.png)

![Django Boards Header](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/django-boards-header-2.png)

The HTML I used is part of the [Bootstrap 4 Navbar Component](https://getbootstrap.com/docs/4.0/components/navbar/).

A nice touch I like to add is to change the font in the “logo” (`.navbar-brand`) of the page.

Go to [fonts.google.com](https://fonts.google.com/), type “Django Boards” or whatever name you gave to your project then click on **apply to all fonts**. Browse a bit, find one that you like.

![Google Fonts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/google-fonts.png)

Add the font in the **base.html** template:

```

    {% load static %}<!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>{% block title %}Django Boards{% endblock %}</title>
        <link href="https://fonts.googleapis.com/css?family=Peralta" rel="stylesheet">
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
        <link rel="stylesheet" href="{% static 'css/app.css' %}">
      </head>
      <body>
        <!-- code suppressed for brevity -->
      </body>
    </html>

```

Now create a new CSS file named **app.css** inside the **static/css** folder:

**static/css/app.css**

```

    .navbar-brand {
      font-family: 'Peralta', cursive;
    }

```

![Django Boards Logo](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/boards-logo.png)

* * *