Django Apps

In the Django philosophy we have two important concepts:

app: is a Web application that does something. An app usually is composed of a set of models (database tables), views, templates, tests.
project: is a collection of configurations and apps. One project can be composed of multiple apps, or a single app.
It’s important to note that you can’t run a Django app without a project. Simple websites like a blog can be written entirely inside a single app, which could be named blog or weblog for example.

Django Apps

It’s a way to organize the source code. In the beginning, it’s not very trivial to determine what is an app or what is not. How to organize the code and so on. But don’t worry much about that right now! Let’s first get comfortable with Django’s API and the fundamentals.

Alright! So, to illustrate let’s create a simple Web Forum or Discussion Board. To create our first app, go to the directory where the manage.py file is and executes the following command:

django-admin startapp boards
Notice that we used the command startapp this time.

This will give us the following directory structure:

myproject/
 |-- myproject/
 |    |-- boards/                <-- our new django app!
 |    |    |-- migrations/
 |    |    |    +-- __init__.py
 |    |    |-- __init__.py
 |    |    |-- admin.py
 |    |    |-- apps.py
 |    |    |-- models.py
 |    |    |-- tests.py
 |    |    +-- views.py
 |    |-- myproject/
 |    |    |-- __init__.py
 |    |    |-- settings.py
 |    |    |-- urls.py
 |    |    |-- wsgi.py
 |    +-- manage.py
 +-- venv/
So, let’s first explore what each file does:

migrations/: here Django store some files to keep track of the changes you create in the models.py file, so to keep the database and the models.py synchronized.
admin.py: this is a configuration file for a built-in Django app called Django Admin.
apps.py: this is a configuration file of the app itself.
models.py: here is where we define the entities of our Web application. The models are translated automatically by Django into database tables.
tests.py: this file is used to write unit tests for the app.
views.py: this is the file where we handle the request/response cycle of our Web application.
Now that we created our first app, let’s configure our project to use it.

To do that, open the settings.py and try to find the INSTALLED_APPS variable:

settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
As you can see, Django already come with 6 built-in apps installed. They offer common functionalities that most Web applications need, like authentication, sessions, static files management (images, javascripts, css, etc.) and so on.

We will explore those apps as we progress in this tutorial series. But for now, let them be and just add our boards app to the list of INSTALLED_APPS:

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'boards',
]
Using the analogy of the square and circles from the previous comic, the yellow circle would be our boards app, and the django.contrib.admin, django.contrib.auth, etc, would be the red circles.