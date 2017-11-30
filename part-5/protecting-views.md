#### Protecting Views

We have to start protecting our views against non-authorized users. So far we have the following view to start new posts:

![Topics view not logged in](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-1.png)

In the picture above the user is not logged in, and even though they can see the page and the form.

Django has a built-in _view decorator_ to avoid that issue:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/4d3334a0daa9e7a872653a22ff39320a#file-models-py-L19)</small>

```

    from django.contrib.auth.decorators import login_required

    @login_required
    def new_topic(request, pk):
        # ...

```

From now on, if the user is not authenticated they will be redirected to the login page:

![Login required redirect](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/login-required.png)

Notice the query string **?next=/boards/1/new/**. We can improve the log in template to make use of the **next** variable and improve the user experience.

##### Configuring Login Next Redirect

**templates/login.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/1ab597fe18e2dc56028f7aa8c3b588b3#file-login-html-L13)</small>

```

    <form method="post" novalidate>
      {% csrf_token %}
      <input type="hidden" name="next" value="{{ next }}">
      {% include 'includes/form.html' %}
      <button type="submit" class="btn btn-primary btn-block">Log in</button>
    </form>

```

Then if we try to log in now, the application will direct us back to where we were.

![Magic](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/Pixton_Comic_Magic.png)

So the **next** parameter is part of a built-in functionality.

##### Login Required Tests

Let’s now add a test case to make sure this view is protected by the `@login_required` decorator. But first, let’s do some refactoring in the **boards/tests/test_views.py** file.

Let’s split the **test_views.py** into three files:

*   **test_view_home.py** will include the **HomeTests** class <small>[(view complete file contents)](https://gist.github.com/vitorfs/6ac3aad244c856d418f18890efcb4a7e#file-test_view_home-py)</small>
*   **test_view_board_topics.py** will include the **BoardTopicsTests** class <small>[(view complete file contents)](https://gist.github.com/vitorfs/6ac3aad244c856d418f18890efcb4a7e#file-test_view_board_topics-py)</small>
*   **test_view_new_topic.py** will include the **NewTopicTests** class <small>[(view complete file contents)](https://gist.github.com/vitorfs/6ac3aad244c856d418f18890efcb4a7e#file-test_view_new_topic-py)</small>

```

    myproject/
     |-- myproject/
     |    |-- accounts/
     |    |-- boards/
     |    |    |-- migrations/
     |    |    |-- templatetags/
     |    |    |-- tests/
     |    |    |    |-- __init__.py
     |    |    |    |-- test_templatetags.py
     |    |    |    |-- test_view_home.py          <-- here
     |    |    |    |-- test_view_board_topics.py  <-- here
     |    |    |    +-- test_view_new_topic.py     <-- and here
     |    |    |-- __init__.py
     |    |    |-- admin.py
     |    |    |-- apps.py
     |    |    |-- models.py
     |    |    +-- views.py
     |    |-- myproject/
     |    |-- static/
     |    |-- templates/
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

```

Run the tests to make sure everything is working.

New let’s add a new test case in the **test_view_new_topic.py** to check if the view is decorated with `@login_required`:

**boards/tests/test_view_new_topic.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/13e75451396d76354b476edaefadbdab#file-test_view_new_topic-py-L84)</small>

```

    from django.test import TestCase
    from django.urls import reverse
    from ..models import Board

    class LoginRequiredNewTopicTests(TestCase):
        def setUp(self):
            Board.objects.create(name='Django', description='Django board.')
            self.url = reverse('new_topic', kwargs={'pk': 1})
            self.response = self.client.get(self.url)

        def test_redirection(self):
            login_url = reverse('login')
            self.assertRedirects(self.response, '{login_url}?next={url}'.format(login_url=login_url, url=self.url))

```

In the test case above we are trying to make a request to the **new topic** view without being authenticated. The expected result is for the request be redirected to the login view.

* * *