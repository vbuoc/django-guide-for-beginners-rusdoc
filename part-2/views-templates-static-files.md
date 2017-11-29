Views, Templates, and Static Files

At the moment we already have a view named home displaying “Hello, World!” in the homepage of our application.

```python
myproject/urls.py

from django.conf.urls import url
from django.contrib import admin

from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^admin/', admin.site.urls),
]
boards/views.py

from django.http import HttpResponse

def home(request):
    return HttpResponse('Hello, World!')
```

We can use this as our starting point. If you recall our wireframes, the Figure 5 showed how the homepage should look like. What we want to do is display a list of boards in a table alongside with some other information.

The first thing to do is import the Board model and list all the existing boards:

boards/views.py

```python
from django.http import HttpResponse
from .models import Board

def home(request):
    boards = Board.objects.all()
    boards_names = list()

    for board in boards:
        boards_names.append(board.name)

    response_html = '<br>'.join(boards_names)

    return HttpResponse(response_html)
```

And the result would be this simple HTML page:

Boards Homepage HttpResponse

But let’s stop right here. We are not going very far rendering HTML like this. For this simple view, all we need is a list of boards; then the rendering part is a job for the Django Template Engine.

Django Template Engine Setup

Create a new folder named templates alongside with the boards and mysite folders:

myproject/
 |-- myproject/
 |    |-- boards/
 |    |-- myproject/
 |    |-- templates/   <-- here!
 |    +-- manage.py
 +-- venv/
Now within the templates folder, create an HTML file named home.html:

templates/home.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Boards</title>
  </head>
  <body>
    <h1>Boards</h1>

    {% for board in boards %}
      {{ board.name }} <br>
    {% endfor %}

  </body>
</html>
```

In the example above we are mixing raw HTML with some special tags `{% for ... in ... %}` and `{{ variable }}`. They are part of the Django Template Language. The example above shows how to iterate over a list of objects using a for. The {{ board.name }} renders the name of the board in the HTML template, generating a dynamic HTML document.

Before we can use this HTML page, we have to tell Django where to find our application’s templates.

Open the settings.py inside the myproject directory and search for the TEMPLATES variable and set the DIRS key to os.path.join(BASE_DIR, 'templates'):

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(BASE_DIR, 'templates')
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Basically what this line is doing is finding the full path of your project directory and appending “/templates” to it.

We can debug this using the Python shell:

python manage.py shell
from django.conf import settings

settings.BASE_DIR
'/Users/vitorfs/Development/myproject'

import os

os.path.join(settings.BASE_DIR, 'templates')
'/Users/vitorfs/Development/myproject/templates'
See? It’s just pointing to the templates folder we created in the previous steps.

Now we can update our home view:

boards/views.py

from django.shortcuts import render
from .models import Board

def home(request):
    boards = Board.objects.all()
    return render(request, 'home.html', {'boards': boards})
The resulting HTML:

Boards Homepage render

We can improve the HTML template to use a table instead:

templates/home.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Boards</title>
  </head>
  <body>
    <h1>Boards</h1>

    <table border="1">
      <thead>
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
              {{ board.name }}<br>
              <small style="color: #888">{{ board.description }}</small>
            </td>
            <td>0</td>
            <td>0</td>
            <td></td>
          </tr>
        {% endfor %}
      </tbody>
    </table>
  </body>
</html>
```

Boards Homepage render

Testing the Homepage

Testing Comic

This is going to be a recurrent subject, and we are going to explore together different concepts and strategies throughout the whole tutorial series.

Let’s write our first test. For now, we will be working in the tests.py file inside the boards app:

boards/tests.py

from django.core.urlresolvers import reverse
from django.test import TestCase

class HomeTests(TestCase):
    def test_home_view_status_code(self):
        url = reverse('home')
        response = self.client.get(url)
        self.assertEquals(response.status_code, 200)
This is a very simple test case but extremely useful. We are testing the status code of the response. The status code 200 means success.

We can check the status code of the response in the console:

Response 200

If there were an uncaught exception, syntax error, or anything, Django would return a status code 500 instead, which means Internal Server Error. Now, imagine our application has 100 views. If we wrote just this simple test for all our views, with just one command, we would be able to test if all views are returning a success code, so the user does not see any error message anywhere. Without automate tests, we would need to check each page, one by one.

To execute the Django’s test suite:

```bash
python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.041s

OK
Destroying test database for alias 'default'...
```

Now we can test if Django returned the correct view function for the requested URL. This is also a useful test because as we progress with the development, you will see that the urls.py module can get very big and complex. The URL conf is all about resolving regex. There are some cases where we have a very permissive URL, so Django can end up returning the wrong view function.

Here’s how we do it:

boards/tests.py

```python
from django.core.urlresolvers import reverse
from django.urls import resolve
from django.test import TestCase
from .views import home

class HomeTests(TestCase):
    def test_home_view_status_code(self):
        url = reverse('home')
        response = self.client.get(url)
        self.assertEquals(response.status_code, 200)

    def test_home_url_resolves_home_view(self):
        view = resolve('/')
        self.assertEquals(view.func, home)
```

In the second test, we are making use of the resolve function. Django uses it to match a requested URL with a list of URLs listed in the urls.py module. This test will make sure the URL /, which is the root URL, is returning the home view.

Test it again:

```bash
python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.027s

OK
Destroying test database for alias 'default'...
```

To see more detail about the test execution, set the verbosity to a higher level:

```bash
python manage.py test --verbosity=2
Creating test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
Operations to perform:
  Synchronize unmigrated apps: messages, staticfiles
  Apply all migrations: admin, auth, boards, contenttypes, sessions
Synchronizing apps without migrations:
  Creating tables...
    Running deferred SQL...
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying boards.0001_initial... OK
  Applying sessions.0001_initial... OK
System check identified no issues (0 silenced).
test_home_url_resolves_home_view (boards.tests.HomeTests) ... ok
test_home_view_status_code (boards.tests.HomeTests) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.017s

OK
Destroying test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
```

Verbosity determines the amount of notification and debug information that will be printed to the console; 0 is no output, 1 is normal output, and 2 is verbose output.

Static Files Setup

Static files are the CSS, JavaScripts, Fonts, Images, or any other resources we may use to compose the user interface.

As it is, Django doesn’t serve those files. Except during the development process, so to make our lives easier. But Django provides some features to help us manage the static files. Those features are available in the django.contrib.staticfiles application already listed in the INSTALLED_APPS configuration.

With so many front-end component libraries available, there’s no reason for us keep rendering basic HTML documents. We can easily add Bootstrap 4 to our project. Bootstrap is an open source toolkit for developing with HTML, CSS, and JavaScript.

In the project root directory, alongside with the boards, templates, and myproject folders, create a new folder named static, and within the static folder create another one named css:

myproject/
 |-- myproject/
 |    |-- boards/
 |    |-- myproject/
 |    |-- templates/
 |    |-- static/       <-- here
 |    |    +-- css/     <-- and here
 |    +-- manage.py
 +-- venv/
Go to getbootstrap.com and download the latest version:

Bootstrap Download

Download the Compiled CSS and JS version.

In your computer, extract the bootstrap-4.0.0-beta-dist.zip file you downloaded from the Bootstrap website, copy the file css/bootstrap.min.css to our project’s css folder:

myproject/
 |-- myproject/
 |    |-- boards/
 |    |-- myproject/
 |    |-- templates/
 |    |-- static/
 |    |    +-- css/
 |    |         +-- bootstrap.min.css    <-- here
 |    +-- manage.py
 +-- venv/
The next step is to instruct Django where to find the static files. Open the settings.py, scroll to the bottom of the file and just after the STATIC_URL, add the following:

```python
STATIC_URL = '/static/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
```

Same thing as the TEMPLATES directory, remember?

Now we have to load the static files (the Bootstrap CSS file) in our template:

templates/home.html

```html
{% load static %}<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Boards</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
  </head>
  <body>
    <!-- body suppressed for brevity ... -->
  </body>
</html>
```

First we load the Static Files App template tags by using the `{% load static %}` in the beginning of the template.

The template tag `{% static %}` is used to compose the URL where the resource lives. In this case, the `{% static 'css/bootstrap.min.css' %}` will return /static/css/bootstrap.min.css, which is equivalent to http://127.0.0.1:8000/static/css/bootstrap.min.css.

The `{% static %}` template tag uses the STATIC_URL configuration in the settings.py to compose the final URL. For example, if you hosted your static files in a subdomain like https://static.example.com/, we would set the STATIC_URL=https://static.example.com/ then the `{% static 'css/bootstrap.min.css' %}` would return https://static.example.com/css/bootstrap.min.css.

If none of this makes sense for you at the moment, don’t worry. Just remember to use the `{% static %}` whenever you need to refer to a CSS, JavaScript or image file. Later on, when we start working with Deployment, we will discuss more it. For now, we are all set up.

Refreshing the page 127.0.0.1:8000 we can see it worked:

Boards Homepage Bootstrap

Now we can edit the template so to take advantage of the Bootstrap CSS:

```html
{% load static %}<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Boards</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
  </head>
  <body>
    <div class="container">
      <ol class="breadcrumb my-4">
        <li class="breadcrumb-item active">Boards</li>
      </ol>
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
                {{ board.name }}
                <small class="text-muted d-block">{{ board.description }}</small>
              </td>
              <td class="align-middle">0</td>
              <td class="align-middle">0</td>
              <td></td>
            </tr>
          {% endfor %}
        </tbody>
      </table>
    </div>
  </body>
</html>
```
The result now:

Boards Homepage Bootstrap

So far we are adding new boards using the interactive console (python manage.py shell). But we need a better way to do it. In the next section, we are going to implement an admin interface for the website administrator manage it.