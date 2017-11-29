# Продвинутые концепции

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/18/a-complete-beginners-guide-to-django-part-3.html

In this tutorial, we are going to dive deep into two fundamental concepts: URLs and Forms. In the process, we are going to explore many other concepts like creating reusable templates and installing third-party libraries. We are also going to write plenty of unit tests.

If you are following this tutorial series since the first part, coding your project and following the tutorial step by step, you may need to update your models.py before starting:

boards/models.py

```python
class Topic(models.Model):
    # other fields...
    # Add `auto_now_add=True` to the `last_updated` field
    last_updated = models.DateTimeField(auto_now_add=True)

class Post(models.Model):
    # other fields...
    # Add `null=True` to the `updated_by` field
    updated_by = models.ForeignKey(User, null=True, related_name='+')
```

Now run the commands with the virtualenv activated:

python manage.py makemigrations
python manage.py migrate
If you already have null=True in the updated_by field and the auto_now_add=True in the last_updated field, you can safely ignore the instructions above.

If you prefer to use my source code as a starting point, you can grab it on GitHub.

The current state of the project can be found under the release tag v0.2-lw. The link below will take you to the right place:

https://github.com/sibtc/django-beginners-guide/tree/v0.2-lw

The development will follow from here.

URLs

Proceeding with the development of our application, now we have to implement a new page to list all the topics that belong to a given Board. Just to recap, below you can see the wireframe we drew in the previous tutorial:

Wireframe Topics

Figure 1: Boards project wireframe listing all topics in the Django board.

We will start by editing the urls.py inside the myproject folder:

myproject/urls.py

```python
from django.conf.urls import url
from django.contrib import admin

from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^boards/(?P<pk>\d+)/$', views.board_topics, name='board_topics'),
    url(r'^admin/', admin.site.urls),
]
```

This time let’s take a moment and analyze the urlpatterns and url.

The URL dispatcher and URLconf (URL configuration) are fundamental parts of a Django application. In the beginning, it can look confusing; I remember having a hard time when I first started developing with Django.

In fact, right now the Django Developers are working on a proposal to make simplified routing syntax. But for now, as per the version 1.11, that’s what we have. So let’s try to understand how it works.

A project can have many urls.py distributed among the apps. But Django needs a url.py to use as a starting point. This special urls.py is called root URLconf. It’s defined in the settings.py file.

myproject/settings.py

```python
ROOT_URLCONF = 'myproject.urls'
```

It already comes configured, so you don’t need to change anything here.

When Django receives a request, it starts searching for a match in the project’s URLconf. It starts with the first entry of the urlpatterns variable, and test the requested URL against each url entry.

If Django finds a match, it will pass the request to the view function, which is the second parameter of the url. The order in the urlpatterns matters, because Django will stop searching as soon as it finds a match. Now, if Django doesn’t find a match in the URLconf, it will raise a 404 exception, which is the error code for Page Not Found.

This is the anatomy of the url function:

def url(regex, view, kwargs=None, name=None):
    # ...
regex: A regular expression for matching URL patterns in strings. Note that these regular expressions do not search GET or POST parameters. In a request to http://127.0.0.1:8000/boards/?page=2 only /boards/ will be processed.
view: A view function used to process the user request for a matched URL. It also accepts the return of the django.conf.urls.include function, which is used to reference an external urls.py file. You can, for example, use it to define a set of app specific URLs, and include it in the root URLconf using a prefix. We will explore more on this concept later on.
kwargs: Arbitrary keyword arguments that’s passed to the target view. It is normally used to do some simple customization on reusable views. We don’t use it very often.
name: A unique identifier for a given URL. This is a very important feature. Always remember to name your URLs. With this, you can change a specific URL in the whole project by just changing the regex. So it’s important to never hard code URLs in the views or templates, and always refer to the URLs by its name.
Matching URL patterns

Basic URLs

Basic URLs are very simple to create. It’s just a matter of matching strings. For example, let’s say we wanted to create an “about” page, it could be defined like this:

```python
from django.conf.urls import url
from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^about/$', views.about, name='about'),
]
```

We can also create deeper URL structures:

```python
from django.conf.urls import url
from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^about/$', views.about, name='about'),
    url(r'^about/company/$', views.about_company, name='about_company'),
    url(r'^about/author/$', views.about_author, name='about_author'),
    url(r'^about/author/vitor/$', views.about_vitor, name='about_vitor'),
    url(r'^about/author/erica/$', views.about_erica, name='about_erica'),
    url(r'^privacy/$', views.privacy_policy, name='privacy_policy'),
]
```

Those are some examples of simple URL routing. For all the examples above, the view function will follow this structure:

```python
def about(request):
    # do something...
    return render(request, 'about.html')

def about_company(request):
    # do something else...
    # return some data along with the view...
    return render(request, 'about_company.html', {'company_name': 'Simple Complex'})
```

Advanced URLs

A more advanced usage of URL routing is achieved by taking advantage of the regex to match certain types of data and create dynamic URLs.

For example, to create a profile page, like many services do like github.com/vitorfs or twitter.com/vitorfs, where “vitorfs” is my username, we can do the following:

```python
from django.conf.urls import url
from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^(?P<username>[\w.@+-]+)/$', views.user_profile, name='user_profile'),
]
```

This will match all valid usernames for a Django User model.

Now observe that the example above is a very permissive URL. That means it will match lots of URL patterns because it is defined in the root of the URL, with no prefix like /profile/<username>/. In this case, if we wanted to define a URL named /about/, we would have do define it before the username URL pattern:

```python
from django.conf.urls import url
from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^about/$', views.about, name='about'),
    url(r'^(?P<username>[\w.@+-]+)/$', views.user_profile, name='user_profile'),
]
```

If the “about” page was defined after the username URL pattern, Django would never find it, because the word “about” would match the username regex, and the view user_profile would be processed instead of the about view function.

There are some side effects to that. For example, from now on, we would have to treat “about” as a forbidden username, because if a user picked “about” as their username, this person would never see their profile page.

URL routing order matters

 Sidenote: If you want to design cool URLs for user profiles, the easiest solution to avoid URL collision is by adding a prefix like /u/vitorfs/, or like Medium does /@vitorfs/, where "@" is the prefix.

If you want no prefix at all, consider using a list of forbidden names like this: github.com/shouldbee/reserved-usernames. Or another example is an application I developed when I was learning Django; I created my list at the time: github.com/vitorfs/parsifal/.

Those collisions are very common. Take GitHub for example; they have this URL to list all the repositories you are currently watching: github.com/watching. Someone registered a username on GitHub with the name "watching," so this person can't see his profile page. We can see a user with this username exists by trying this URL: github.com/watching/repositories which was supposed to list the user's repositories, like mine for example github.com/vitorfs/repositories.
The whole idea of this kind of URL routing is to create dynamic pages where part of the URL will be used as an identifier for a certain resource, that will be used to compose a page. This identifier can be an integer ID or a string for example.

Initially, we will be working with the Board ID to create a dynamic page for the Topics. Let’s read again the example I gave at the beginning of the URLs section:

url(r'^boards/(?P<pk>\d+)/$', views.board_topics, name='board_topics')
The regex \d+ will match an integer of arbitrary size. This integer will be used to retrieve the Board from the database. Now observe that we wrote the regex as (?P<pk>\d+), this is telling Django to capture the value into a keyword argument named pk.

Here is how we write a view function for it:

```python
def board_topics(request, pk):
    # do something...
```

Because we used the (?P<pk>\d+) regex, the keyword argument in the board_topics must be named pk.

If we wanted to use any name, we could do it like this:

url(r'^boards/(\d+)/$', views.board_topics, name='board_topics')
Then the view function could be defined like this:

def board_topics(request, board_id):
    # do something...
Or like this:

def board_topics(request, id):
    # do something...
The name wouldn’t matter. But it’s a good practice to use named parameters because when we start composing bigger URLs capturing multiple IDs and variables, it will be easier to read.

 Sidenote: PK or ID?

PK stands for Primary Key. It's a shortcut for accessing a model's primary key. All Django models have this attribute.

For the most cases, using the pk property is the same as id. That's because if we don't define a primary key for a model, Django will automatically create an AutoField named id, which will be its primary key.

If you defined a different primary key for a model, for example, let's say the field email is your primary key. To access it you could either use obj.email or obj.pk.
Using the URLs API

It’s time to write some code. Let’s implement the topic listing page (see Figure 1) I mentioned at the beginning of the URLs section.

First, edit the urls.py adding our new URL route:

myproject/urls.py

```python
from django.conf.urls import url
from django.contrib import admin

from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^boards/(?P<pk>\d+)/$', views.board_topics, name='board_topics'),
    url(r'^admin/', admin.site.urls),
]
```

Now let’s create the view function board_topics:

boards/views.py

```python
from django.shortcuts import render
from .models import Board

def home(request):
    # code suppressed for brevity

def board_topics(request, pk):
    board = Board.objects.get(pk=pk)
    return render(request, 'topics.html', {'board': board})
```

In the templates folder, create a new template named topics.html:

templates/topics.html

```html
{% load static %}<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ board.name }}</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
  </head>
  <body>
    <div class="container">
      <ol class="breadcrumb my-4">
        <li class="breadcrumb-item">Boards</li>
        <li class="breadcrumb-item active">{{ board.name }}</li>
      </ol>
    </div>
  </body>
</html>
```
> Note: For now we are simply creating new HTML templates. No worries, in the following section I will show you how to create reusable templates.
Now check the URL http://127.0.0.1:8000/boards/1/ in a web browser. The result should be the following page:

Topics Page

Time to write some tests! Edit the tests.py file and add the following tests in the bottom of the file:

boards/tests.py

```python
from django.core.urlresolvers import reverse
from django.urls import resolve
from django.test import TestCase
from .views import home, board_topics
from .models import Board

class HomeTests(TestCase):
    # ...

class BoardTopicsTests(TestCase):
    def setUp(self):
        Board.objects.create(name='Django', description='Django board.')

    def test_board_topics_view_success_status_code(self):
        url = reverse('board_topics', kwargs={'pk': 1})
        response = self.client.get(url)
        self.assertEquals(response.status_code, 200)

    def test_board_topics_view_not_found_status_code(self):
        url = reverse('board_topics', kwargs={'pk': 99})
        response = self.client.get(url)
        self.assertEquals(response.status_code, 404)

    def test_board_topics_url_resolves_board_topics_view(self):
        view = resolve('/boards/1/')
        self.assertEquals(view.func, board_topics)
```

A few things to note here. This time we used the setUp method. In the setup method, we created a Board instance to use in the tests. We have to do that because the Django testing suite doesn’t run your tests against the current database. To run the tests Django creates a new database on the fly, applies all the model migrations, runs the tests, and when done, destroys the testing database.

So in the setUp method, we prepare the environment to run the tests, so to simulate a scenario.

The test_board_topics_view_success_status_code method: is testing if Django is returning a status code 200 (success) for an existing Board.
The test_board_topics_view_not_found_status_code method: is testing if Django is returning a status code 404 (page not found) for a Board that doesn’t exist in the database.
The test_board_topics_url_resolves_board_topics_view method: is testing if Django is using the correct view function to render the topics.
Now it’s time to run the tests:

python manage.py test
And the output:

```bash
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.E...
======================================================================
ERROR: test_board_topics_view_not_found_status_code (boards.tests.BoardTopicsTests)
----------------------------------------------------------------------
Traceback (most recent call last):
# ...
boards.models.DoesNotExist: Board matching query does not exist.

----------------------------------------------------------------------
Ran 5 tests in 0.093s

FAILED (errors=1)
Destroying test database for alias 'default'...
```

The test test_board_topics_view_not_found_status_code failed. We can see in the Traceback it returned an exception “boards.models.DoesNotExist: Board matching query does not exist.”

Topics Error 500 Page

In production with DEBUG=False, the visitor would see a 500 Internal Server Error page. But that’s not the behavior we want.

We want to show a 404 Page Not Found. So let’s refactor our view:

boards/views.py

```python
from django.shortcuts import render
from django.http import Http404
from .models import Board

def home(request):
    # code suppressed for brevity

def board_topics(request, pk):
    try:
        board = Board.objects.get(pk=pk)
    except Board.DoesNotExist:
        raise Http404
    return render(request, 'topics.html', {'board': board})
Let’s test again:

python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.....
----------------------------------------------------------------------
Ran 5 tests in 0.042s

OK
Destroying test database for alias 'default'...
```

Yay! Now it’s working as expected.

Topics Error 404 Page

This is the default page Django show while with DEBUG=False. Later on, we can customize the 404 page to show something else.

Now that’s a very common use case. In fact, Django has a shortcut to try to get an object, or return a 404 with the object does not exist.

So let’s refactor the board_topics view again:

```python
from django.shortcuts import render, get_object_or_404
from .models import Board

def home(request):
    # code suppressed for brevity

def board_topics(request, pk):
    board = get_object_or_404(Board, pk=pk)
    return render(request, 'topics.html', {'board': board})
```

Changed the code? Test it.

```bash
python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.....
----------------------------------------------------------------------
Ran 5 tests in 0.052s

OK
Destroying test database for alias 'default'...
```

Didn’t break anything. We can proceed with the development.

The next step now is to create the navigation links in the screens. The homepage should have a link to take the visitor to the topics page of a given Board. Similarly, the topics page should have a link back to the homepage.

Wireframe Links

We can start by writing some tests for the HomeTests class:

boards/tests.py

```python
class HomeTests(TestCase):
    def setUp(self):
        self.board = Board.objects.create(name='Django', description='Django board.')
        url = reverse('home')
        self.response = self.client.get(url)

    def test_home_view_status_code(self):
        self.assertEquals(self.response.status_code, 200)

    def test_home_url_resolves_home_view(self):
        view = resolve('/')
        self.assertEquals(view.func, home)

    def test_home_view_contains_link_to_topics_page(self):
        board_topics_url = reverse('board_topics', kwargs={'pk': self.board.pk})
        self.assertContains(self.response, 'href="{0}"'.format(board_topics_url))
```

Observe that now we added a setUp method for the HomeTests as well. That’s because now we are going to need a Board instance and also we moved the url and response to the setUp, so we can reuse the same response in the new test.

The new test here is the test_home_view_contains_link_to_topics_page. Here we are using the assertContains method to test if the response body contains a given text. The text we are using in the test, is the href part of an a tag. So basically we are testing if the response body has the text href="/boards/1/".

Let’s run the tests:

```bash
python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
....F.
======================================================================
FAIL: test_home_view_contains_link_to_topics_page (boards.tests.HomeTests)
----------------------------------------------------------------------
# ...

AssertionError: False is not true : Couldn't find 'href="/boards/1/"' in response

----------------------------------------------------------------------
Ran 6 tests in 0.034s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

Now we can write the code that will make this test pass.

Edit the home.html template:

templates/home.html

```html
<!-- code suppressed for brevity -->
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
<!-- code suppressed for brevity -->
```

So basically we changed the line:

`{{ board.name }}`
To:

`<a href="{% url 'board_topics' board.pk %}">{{ board.name }}</a>`

Always use the {% url %} template tag to compose the applications URLs. The first parameter is the name of the URL (defined in the URLconf, i.e., the urls.py), then you can pass an arbitrary number of arguments as needed.

If it were a simple URL, like the homepage, it would be just {% url 'home' %}.

Save the file and run the tests again:

```bash
python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
......
----------------------------------------------------------------------
Ran 6 tests in 0.037s

OK
Destroying test database for alias 'default'...
```

Good! Now we can check how it looks in the web browser:

Boards with Link

Now the link back. We can write the test first:

boards/tests.py

```python
class BoardTopicsTests(TestCase):
    # code suppressed for brevity...

    def test_board_topics_view_contains_link_back_to_homepage(self):
        board_topics_url = reverse('board_topics', kwargs={'pk': 1})
        response = self.client.get(board_topics_url)
        homepage_url = reverse('home')
        self.assertContains(response, 'href="{0}"'.format(homepage_url))
```

Run the tests:

```bash
python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.F.....
======================================================================
FAIL: test_board_topics_view_contains_link_back_to_homepage (boards.tests.BoardTopicsTests)
----------------------------------------------------------------------
Traceback (most recent call last):
# ...

AssertionError: False is not true : Couldn't find 'href="/"' in response

----------------------------------------------------------------------
Ran 7 tests in 0.054s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

Update the board topics template:

templates/topics.html

```html
{% load static %}<!DOCTYPE html>
<html>
  <head><!-- code suppressed for brevity --></head>
  <body>
    <div class="container">
      <ol class="breadcrumb my-4">
        <li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
        <li class="breadcrumb-item active">{{ board.name }}</li>
      </ol>
    </div>
  </body>
</html>
```

Run the tests:

```bash
python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.......
----------------------------------------------------------------------
Ran 7 tests in 0.061s

OK
Destroying test database for alias 'default'...
```

Board Topics with Link

As I mentioned before, URL routing is a fundamental part of a web application. With this knowledge, we should be able to proceed with the development. Next, to complete the section about URLs, you will find a summary of useful URL patterns.

List of Useful URL Patterns

The trick part is the regex. So I prepared a list of the most used URL patterns. You can always refer to this list when you need a specific URL.

Primary Key AutoField
Regex	(?P<pk>\d+)
Example	url(r'^questions/(?P<pk>\d+)/$', views.question, name='question')
Valid URL	/questions/934/
Captures	{'pk': '934'}
Slug Field
Regex	(?P<slug>[-\w]+)
Example	url(r'^posts/(?P<slug>[-\w]+)/$', views.post, name='post')
Valid URL	/posts/hello-world/
Captures	{'slug': 'hello-world'}
Slug Field with Primary Key
Regex	(?P<slug>[-\w]+)-(?P<pk>\d+)
Example	url(r'^blog/(?P<slug>[-\w]+)-(?P<pk>\d+)/$', views.blog_post, name='blog_post')
Valid URL	/blog/hello-world-159/
Captures	{'slug': 'hello-world', 'pk': '159'}
Django User Username
Regex	(?P<username>[\w.@+-]+)
Example	url(r'^profile/(?P<username>[\w.@+-]+)/$', views.user_profile, name='user_profile')
Valid URL	/profile/vitorfs/
Captures	{'username': 'vitorfs'}
Year
Regex	(?P<year>[0-9]{4})
Example	url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive, name='year')
Valid URL	/articles/2016/
Captures	{'year': '2016'}
Year / Month
Regex	(?P<year>[0-9]{4})/(?P<month>[0-9]{2})
Example	url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive, name='month')
Valid URL	/articles/2016/01/
Captures	{'year': '2016', 'month': '01'}
You can find more details about those patterns in this post: List of Useful URL Patterns.

> [Переиспользуемые шаблоны](part-3/reusable-templates.md)
