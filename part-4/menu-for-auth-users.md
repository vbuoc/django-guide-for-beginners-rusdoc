#### Displaying Menu For Authenticated Users

Now we will need to do some tweaks in our **base.html** template. We have to add a dropdown menu with the logout link.

The Bootstrap 4 dropdown component needs jQuery to work.

First, go to [jquery.com/download/](https://jquery.com/download/) and download the **compressed, production jQuery 3.2.1** version.

![jQuery Download](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/jquery-download.jpg)

Inside the **static** folder, create a new folder named **js**. Copy the **jquery-3.2.1.min.js** file to there.

Bootstrap 4 also needs a library called **Popper** to work. Go to [popper.js.org](https://popper.js.org/) and download the latest version.

Inside the **popper.js-1.12.5** folder, go to **dist/umd** and copy the file **popper.min.js** to our **js** folder. Pay attention here; Bootstrap 4 will only work with the **umd/popper.min.js**. So make sure you are copying the right file.

If you no longer have all the Bootstrap 4 files, download it again from [getbootstrap.com](http://getbootstrap.com/).

Similarly, copy the **bootstrap.min.js** file to our **js** folder as well.

The final result should be:

<figure class="highlight">

    myproject/
     |-- myproject/
     |    |-- accounts/
     |    |-- boards/
     |    |-- myproject/
     |    |-- static/
     |    |    |-- css/
     |    |    +-- js/
     |    |         |-- bootstrap.min.js
     |    |         |-- jquery-3.2.1.min.js
     |    |         +-- popper.min.js
     |    |-- templates/
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

</figure>

In the bottom of the **base.html** file, add the scripts _after_ the `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">endblock</span> <span class="w"></span> <span class="err">body</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>`:

**templates/base.html**

<figure class="highlight">

    {% load static %}<!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>{% block title %}Django Boards{% endblock %}</title>
        <link href="https://fonts.googleapis.com/css?family=Peralta" rel="stylesheet">
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
        <link rel="stylesheet" href="{% static 'css/app.css' %}">
        {% block stylesheet %}{% endblock %}
      </head>
      <body>
        {% block body %}
        <!-- code suppressed for brevity -->
        {% endblock body %}
        <script src="{% static 'js/jquery-3.2.1.min.js' %}"></script>
        <script src="{% static 'js/popper.min.js' %}"></script>
        <script src="{% static 'js/bootstrap.min.js' %}"></script>
      </body>
    </html>

</figure>

If you found the instructions confusing, just download the files using the direct links below:

*   [https://code.jquery.com/jquery-3.2.1.min.js](https://code.jquery.com/jquery-3.2.1.min.js)
*   [https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.11.0/umd/popper.min.js](https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.11.0/umd/popper.min.js)
*   [https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/js/bootstrap.min.js](https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/js/bootstrap.min.js)

Right-click and ***Save link as…**.

Now we can add the Bootstrap 4 dropdown menu:

**templates/base.html**

<figure class="highlight">

    <nav class="navbar navbar-expand-sm navbar-dark bg-dark">
      <div class="container">
        <a class="navbar-brand" href="{% url 'home' %}">Django Boards</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#mainMenu" aria-controls="mainMenu" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="mainMenu">
          <ul class="navbar-nav ml-auto">
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" id="userMenu" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                {{ user.username }}
              </a>
              <div class="dropdown-menu dropdown-menu-right" aria-labelledby="userMenu">
                <a class="dropdown-item" href="#">My account</a>
                <a class="dropdown-item" href="#">Change password</a>
                <div class="dropdown-divider"></div>
                <a class="dropdown-item" href="{% url 'logout' %}">Log out</a>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </nav>

</figure>

![Dropdown menu](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/dropdown.png)

Let’s try it. Click on logout:

![Logout](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/logout.png)

It’s working. But the dropdown is showing regardless of the user being logged in or not. The difference is that now the username is empty, and we can only see an arrow.

We can improve it a little bit:

<figure class="highlight">

    <nav class="navbar navbar-expand-sm navbar-dark bg-dark">
      <div class="container">
        <a class="navbar-brand" href="{% url 'home' %}">Django Boards</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#mainMenu" aria-controls="mainMenu" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="mainMenu">
          {% if user.is_authenticated %}
            <ul class="navbar-nav ml-auto">
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" href="#" id="userMenu" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                  {{ user.username }}
                </a>
                <div class="dropdown-menu dropdown-menu-right" aria-labelledby="userMenu">
                  <a class="dropdown-item" href="#">My account</a>
                  <a class="dropdown-item" href="#">Change password</a>
                  <div class="dropdown-divider"></div>
                  <a class="dropdown-item" href="{% url 'logout' %}">Log out</a>
                </div>
              </li>
            </ul>
          {% else %}
            <form class="form-inline ml-auto">
              <a href="#" class="btn btn-outline-secondary">Log in</a>
              <a href="{% url 'signup' %}" class="btn btn-primary ml-2">Sign up</a>
            </form>
          {% endif %}
        </div>
      </div>
    </nav>

</figure>

Now we are telling Django to show the dropdown menu if the user is logged in, and if not, show the log in and sign up buttons:

![Main Menu](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/mainmenu.png)

* * *