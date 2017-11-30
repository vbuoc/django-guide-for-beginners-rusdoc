#### Update View

Let’s get back to the implementation of our project. This time we are going to use a GCBV to implement the **edit post** view:

![Edit post](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/edit.png)

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/376657085570a3dedf7e5f6e6fffc5e3#file-views-py-L66)</small>

```

    from django.shortcuts import redirect
    from django.views.generic import UpdateView
    from django.utils import timezone

    class PostUpdateView(UpdateView):
        model = Post
        fields = ('message', )
        template_name = 'edit_post.html'
        pk_url_kwarg = 'post_pk'
        context_object_name = 'post'

        def form_valid(self, form):
            post = form.save(commit=False)
            post.updated_by = self.request.user
            post.updated_at = timezone.now()
            post.save()
            return redirect('topic_posts', pk=post.topic.board.pk, topic_pk=post.topic.pk)

```

With the **UpdateView** and the **CreateView**, we have the option to either define **form_class** or the **fields** attribute. In the example above we are using the **fields** attribute to create a model form on-the-fly. Internally, Django will use a model form factory to compose a form of the **Post** model. Since it’s a very simple form with just the **message** field, we can afford to work like this. But for complex form definitions, it’s better to define a model form externally and refer to it here.

The **pk_url_kwarg** will be used to identify the name of the keyword argument used to retrieve the **Post** object. It’s the same as we define in the **urls.py**.

If we don’t set the **context_object_name** attribute, the **Post** object will be available in the template as “object.” So, here we are using the **context_object_name** to rename it to **post** instead. You will see how we are using it in the template below.

In this particular example, we had to override the **form_valid()** method so as to set some extra fields such as the **updated_by** and **updated_at**. You can see what the base **form_valid()** method looks like here: [UpdateView#form_valid](https://ccbv.co.uk/projects/Django/1.11/django.views.generic.edit/UpdateView/#form_valid).

**myproject/urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/376657085570a3dedf7e5f6e6fffc5e3#file-urls-py-L40)</small>

```

    from django.conf.urls import url
    from boards import views

    urlpatterns = [
        # ...
        url(r'^boards/(?P<pk>\d+)/topics/(?P<topic_pk>\d+)/posts/(?P<post_pk>\d+)/edit//figure>,
            views.PostUpdateView.as_view(), name='edit_post'),
    ]

```

Now we can add the link to the edit page:

**templates/topic_posts.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/589d31af9d06c5b21ec9b1623be0c357#file-topic_posts-html-L40)</small>

```

    {% if post.created_by == user %}
      <div class="mt-3">
        <a href="{% url 'edit_post' post.topic.board.pk post.topic.pk post.pk %}"
           class="btn btn-primary btn-sm"
           role="button">Edit</a>
      </div>
    {% endif %}

```

**templates/edit_post.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/376657085570a3dedf7e5f6e6fffc5e3#file-edit_post-html)</small>

```

    {% extends 'base.html' %}

    {% block title %}Edit post{% endblock %}

    {% block breadcrumb %}
      <li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
      <li class="breadcrumb-item"><a href="{% url 'board_topics' post.topic.board.pk %}">{{ post.topic.board.name }}</a></li>
      <li class="breadcrumb-item"><a href="{% url 'topic_posts' post.topic.board.pk post.topic.pk %}">{{ post.topic.subject }}</a></li>
      <li class="breadcrumb-item active">Edit post</li>
    {% endblock %}

    {% block content %}
      <form method="post" class="mb-4" novalidate>
        {% csrf_token %}
        {% include 'includes/form.html' %}
        <button type="submit" class="btn btn-success">Save changes</button>
        <a href="{% url 'topic_posts' post.topic.board.pk post.topic.pk %}" class="btn btn-outline-secondary" role="button">Cancel</a>
      </form>
    {% endblock %}

```

Observe now how we are navigating through the post object: **post.topic.board.pk**. If we didn’t set the **context_object_name** to **post**, it would be available as: **object.topic.board.pk**. Got it?

![Edit post](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/edit-1.png)

##### Testing The Update View

Create a new test file named **test_view_edit_post.py** inside the **boards/tests** folder. Clicking on the link below you will see many routine tests, just like we have been doing in this tutorial. So I will just highlight the new parts here:

**boards/tests/test_view_edit_post.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/286917ce31687732eba4e545fc3cbdea)</small>

```

    from django.contrib.auth.models import User
    from django.test import TestCase
    from django.urls import reverse
    from ..models import Board, Post, Topic
    from ..views import PostUpdateView

    class PostUpdateViewTestCase(TestCase):
        '''
        Base test case to be used in all `PostUpdateView` view tests
        '''
        def setUp(self):
            self.board = Board.objects.create(name='Django', description='Django board.')
            self.username = 'john'
            self.password = '123'
            user = User.objects.create_user(username=self.username, email='john@doe.com', password=self.password)
            self.topic = Topic.objects.create(subject='Hello, world', board=self.board, starter=user)
            self.post = Post.objects.create(message='Lorem ipsum dolor sit amet', topic=self.topic, created_by=user)
            self.url = reverse('edit_post', kwargs={
                'pk': self.board.pk,
                'topic_pk': self.topic.pk,
                'post_pk': self.post.pk
            })

    class LoginRequiredPostUpdateViewTests(PostUpdateViewTestCase):
        def test_redirection(self):
            '''
            Test if only logged in users can edit the posts
            '''
            login_url = reverse('login')
            response = self.client.get(self.url)
            self.assertRedirects(response, '{login_url}?next={url}'.format(login_url=login_url, url=self.url))

    class UnauthorizedPostUpdateViewTests(PostUpdateViewTestCase):
        def setUp(self):
            '''
            Create a new user different from the one who posted
            '''
            super().setUp()
            username = 'jane'
            password = '321'
            user = User.objects.create_user(username=username, email='jane@doe.com', password=password)
            self.client.login(username=username, password=password)
            self.response = self.client.get(self.url)

        def test_status_code(self):
            '''
            A topic should be edited only by the owner.
            Unauthorized users should get a 404 response (Page Not Found)
            '''
            self.assertEquals(self.response.status_code, 404)

    class PostUpdateViewTests(PostUpdateViewTestCase):
        # ...

    class SuccessfulPostUpdateViewTests(PostUpdateViewTestCase):
        # ...

    class InvalidPostUpdateViewTests(PostUpdateViewTestCase):
        # ...

```

For now, the important parts are: **PostUpdateViewTestCase** is a class we defined to be reused across the other test cases. It’s just the basic setup, creating users, topic, boards, and so on.

The class **LoginRequiredPostUpdateViewTests** will test if the view is protected with the `@login_required` decorator. That is if only authenticated users can access the edit page.

The class **UnauthorizedPostUpdateViewTests** creates a new user, different from the one who posted and tries to access the edit page. The application should only authorize the owner of the post to edit it.

Let’s run the tests:

```

    python manage.py test boards.tests.test_view_edit_post

```

```

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    ..F.......F
    ======================================================================
    FAIL: test_redirection (boards.tests.test_view_edit_post.LoginRequiredPostUpdateViewTests)
    ----------------------------------------------------------------------
    ...
    AssertionError: 200 != 302 : Response didn't redirect as expected: Response code was 200 (expected 302)

    ======================================================================
    FAIL: test_status_code (boards.tests.test_view_edit_post.UnauthorizedPostUpdateViewTests)
    ----------------------------------------------------------------------
    ...
    AssertionError: 200 != 404

    ----------------------------------------------------------------------
    Ran 11 tests in 1.360s

    FAILED (failures=2)
    Destroying test database for alias 'default'...

```

First, let’s fix the problem with the `@login_required` decorator. The way we use view decorators on class-based views is a little bit different. We need an extra import:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/826a6d421ebbeb80a0aee8e1b9b70398#file-views-py-L67)</small>

```

    from django.contrib.auth.decorators import login_required
    from django.shortcuts import redirect
    from django.views.generic import UpdateView
    from django.utils import timezone
    from django.utils.decorators import method_decorator
    from .models import Post

    @method_decorator(login_required, name='dispatch')
    class PostUpdateView(UpdateView):
        model = Post
        fields = ('message', )
        template_name = 'edit_post.html'
        pk_url_kwarg = 'post_pk'
        context_object_name = 'post'

        def form_valid(self, form):
            post = form.save(commit=False)
            post.updated_by = self.request.user
            post.updated_at = timezone.now()
            post.save()
            return redirect('topic_posts', pk=post.topic.board.pk, topic_pk=post.topic.pk)

```

We can’t decorate the class directly with the `@login_required` decorator. We have to use the utility `@method_decorator`, and pass a decorator (or a list of decorators) and tell which method should be decorated. In class-based views it’s common to decorate the **dispatch** method. It’s an internal method Django use (defined inside the View class). All requests pass through this method, so it’s safe to decorate it.

Run the tests:

```

    python manage.py test boards.tests.test_view_edit_post

```

```

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    ..........F
    ======================================================================
    FAIL: test_status_code (boards.tests.test_view_edit_post.UnauthorizedPostUpdateViewTests)
    ----------------------------------------------------------------------
    ...
    AssertionError: 200 != 404

    ----------------------------------------------------------------------
    Ran 11 tests in 1.353s

    FAILED (failures=1)
    Destroying test database for alias 'default'...

```

Okay! We fixed the `@login_required` problem. Now we have to deal with the other users editing any posts problem.

The easiest way to solve this problem is by overriding the `get_queryset` method of the **UpdateView**. You can see what the original method looks like here [UpdateView#get_queryset](https://ccbv.co.uk/projects/Django/1.11/django.views.generic.edit/UpdateView/#get_queryset).

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/667d8439ecf05e58f14fcc74672e48da)</small>

```

    @method_decorator(login_required, name='dispatch')
    class PostUpdateView(UpdateView):
        model = Post
        fields = ('message', )
        template_name = 'edit_post.html'
        pk_url_kwarg = 'post_pk'
        context_object_name = 'post'

        def get_queryset(self):
            queryset = super().get_queryset()
            return queryset.filter(created_by=self.request.user)

        def form_valid(self, form):
            post = form.save(commit=False)
            post.updated_by = self.request.user
            post.updated_at = timezone.now()
            post.save()
            return redirect('topic_posts', pk=post.topic.board.pk, topic_pk=post.topic.pk)

```

With the line `queryset = super().get_queryset()` we are reusing the `get_queryset` method from the parent class, that is, the **UpateView** class. Then, we are adding an extra filter to the queryset, which is filtering the post using the logged in user, available inside the **request** object.

Test it again:

```

    python manage.py test boards.tests.test_view_edit_post

```

```

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    ...........
    ----------------------------------------------------------------------
    Ran 11 tests in 1.321s

    OK
    Destroying test database for alias 'default'...

```

All good!

* * *