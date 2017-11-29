# Начиная новый проект

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/04/a-complete-beginners-guide-to-django-part-1.html

To start a new Django project, run the command below:

django-admin startproject myproject
The command-line utility django-admin is automatically installed with Django.

After we run the command above, it will generate the base folder structure for a Django project.

Right now, our myproject directory looks like this:

myproject/                  <-- higher level folder
 |-- myproject/             <-- django project folder
 |    |-- myproject/
 |    |    |-- __init__.py
 |    |    |-- settings.py
 |    |    |-- urls.py
 |    |    |-- wsgi.py
 |    +-- manage.py
 +-- venv/                  <-- virtual environment folder
Our initial project structure is composed of five files:

manage.py: a shortcut to use the django-admin command-line utility. It’s used to run management commands related to our project. We will use it to run the development server, run tests, create migrations and much more.
__init__.py: this empty file tells Python that this folder is a Python package.
settings.py: this file contains all the project’s configuration. We will refer to this file all the time!
urls.py: this file is responsible for mapping the routes and paths in our project. For example, if you want to show something in the URL /about/, you have to map it here first.
wsgi.py: this file is a simple gateway interface used for deployment. You don’t have to bother about it. Just let it be for now.
Django comes with a simple web server installed. It’s very convenient during the development, so we don’t have to install anything else to run the project locally. We can test it by executing the command:

python manage.py runserver
For now, you can ignore the migration errors; we will get to that later.

Now open the following URL in a Web browser: http://127.0.0.1:8000 and you should see the following page:

It worked!

Hit Control + C to stop the development server.




