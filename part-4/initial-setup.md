#### Initial Setup

To manage all this information, we can break it down in a different app. In the project root, in the same page where the **manage.py** script is, run the following command to start a new app:

<figure class="highlight">

    django-admin startapp accounts

</figure>

The project structure should like this right now:

<figure class="highlight">

    myproject/
     |-- myproject/
     |    |-- accounts/     <-- our new django app!
     |    |-- boards/
     |    |-- myproject/
     |    |-- static/
     |    |-- templates/
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

</figure>

Next step, include the **accounts** app to the `INSTALLED_APPS` in the **settings.py** file:

<figure class="highlight">

    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',

        'widget_tweaks',

        'accounts',
        'boards',
    ]

</figure>

From now on, we will be working on the **accounts** app.

* * *