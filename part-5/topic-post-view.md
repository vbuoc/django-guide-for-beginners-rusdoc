#### Topic Posts View

Let’s take the time now to implement the posts listing page, accordingly to the wireframe below:

![Wireframe Posts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/wireframe-posts.png)

First, we need a route:

**myproject/urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/aede6d3b7dc3494cf0df48f796075403#file-urls-py-L38)</small>

<figure class="highlight">

    url(r'^boards/(?P<pk>\d+)/topics/(?P<topic_pk>\d+)//figure>, views.topic_posts, name='topic_posts'),

</figure>

Observe that now we are dealing with two keyword arguments: `pk` which is used to identify the Board, and now we have the `topic_pk` which is used to identify which topic to retrieve from the database.

The matching view would be like this:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/3d73ef25a01eceea07ef3ad8538437cf#file-views-py-L39)</small>

<figure class="highlight">

    from django.shortcuts import get_object_or_404, render
    from .models import Topic

    def topic_posts(request, pk, topic_pk):
        topic = get_object_or_404(Topic, board__pk=pk, pk=topic_pk)
        return render(request, 'topic_posts.html', {'topic': topic})

</figure>

Note that we are indirectly retrieving the current board. Remember that the topic model is related to the board model, so we can access the current board. You will see in the next snippet:

**templates/topic_posts.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/17e583f4f0068850c5929bd307dd436a)</small>

<figure class="highlight">

    {% extends 'base.html' %}

    {% block title %}{{ topic.subject }}{% endblock %}

    {% block breadcrumb %}
      <li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
      <li class="breadcrumb-item"><a href="{% url 'board_topics' topic.board.pk %}">{{ topic.board.name }}</a></li>
      <li class="breadcrumb-item active">{{ topic.subject }}</li>
    {% endblock %}

    {% block content %}

    {% endblock %}

</figure>

Observe that now instead of using `board.name` in the template, we are navigating through the topic properties, using `topic.board.name`.

![Posts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-1.png)

Now let’s create a new test file for the **topic_posts** view:

**boards/tests/test_view_topic_posts.py**

<figure class="highlight">

    from django.contrib.auth.models import User
    from django.test import TestCase
    from django.urls import resolve, reverse

    from ..models import Board, Post, Topic
    from ..views import topic_posts

    class TopicPostsTests(TestCase):
        def setUp(self):
            board = Board.objects.create(name='Django', description='Django board.')
            user = User.objects.create_user(username='john', email='john@doe.com', password='123')
            topic = Topic.objects.create(subject='Hello, world', board=board, starter=user)
            Post.objects.create(message='Lorem ipsum dolor sit amet', topic=topic, created_by=user)
            url = reverse('topic_posts', kwargs={'pk': board.pk, 'topic_pk': topic.pk})
            self.response = self.client.get(url)

        def test_status_code(self):
            self.assertEquals(self.response.status_code, 200)

        def test_view_function(self):
            view = resolve('/boards/1/topics/1/')
            self.assertEquals(view.func, topic_posts)

</figure>

Note that the test setup is starting to get more complex. We can create mixins or an abstract class to reuse the code as needed. We can also use a third party library to setup some test data, to reduce the boilerplate code.

Also, by now we already have a significant amount of tests, and it’s gradually starting to run slower. We can instruct the test suite just to run tests from a given app:

<figure class="highlight">

    python manage.py test boards

</figure>

<figure class="highlight">

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    .......................
    ----------------------------------------------------------------------
    Ran 23 tests in 1.246s

    OK
    Destroying test database for alias 'default'...

</figure>

We could also run only a specific test file:

<figure class="highlight">

    python manage.py test boards.tests.test_view_topic_posts

</figure>

<figure class="highlight">

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    ..
    ----------------------------------------------------------------------
    Ran 2 tests in 0.129s

    OK
    Destroying test database for alias 'default'...

</figure>

Or just a specific test case:

<figure class="highlight">

    python manage.py test boards.tests.test_view_topic_posts.TopicPostsTests.test_status_code

</figure>

<figure class="highlight">

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.100s

    OK
    Destroying test database for alias 'default'...

</figure>

Cool, right?

Let’s keep moving forward.

Inside the **topic_posts.html**, we can create a for loop iterating over the topic posts:

**templates/topic_posts.html**

<figure class="highlight">

    {% extends 'base.html' %}

    {% load static %}

    {% block title %}{{ topic.subject }}{% endblock %}

    {% block breadcrumb %}
      <li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
      <li class="breadcrumb-item"><a href="{% url 'board_topics' topic.board.pk %}">{{ topic.board.name }}</a></li>
      <li class="breadcrumb-item active">{{ topic.subject }}</li>
    {% endblock %}

    {% block content %}

      <div class="mb-4">
        <a href="#" class="btn btn-primary" role="button">Reply</a>
      </div>

      {% for post in topic.posts.all %}
        <div class="card mb-2">
          <div class="card-body p-3">
            <div class="row">
              <div class="col-2">
                <img src="{% static 'img/avatar.svg' %}" alt="{{ post.created_by.username }}" class="w-100">
                <small>Posts: {{ post.created_by.posts.count }}</small>
              </div>
              <div class="col-10">
                <div class="row mb-3">
                  <div class="col-6">
                    <strong class="text-muted">{{ post.created_by.username }}</strong>
                  </div>
                  <div class="col-6 text-right">
                    <small class="text-muted">{{ post.created_at }}</small>
                  </div>
                </div>
                {{ post.message }}
                {% if post.created_by == user %}
                  <div class="mt-3">
                    <a href="#" class="btn btn-primary btn-sm" role="button">Edit</a>
                  </div>
                {% endif %}
              </div>
            </div>
          </div>
        </div>
      {% endfor %}

    {% endblock %}

</figure>

![Posts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-2.png)

Since right now we don’t have a way to upload a user picture, let’s just have an empty image.

I downloaded a free image from [IconFinder](https://www.iconfinder.com/search/?q=user&license=2&price=free) and saved in the **static/img** folder of the project.

We still haven’t really explored Django’s ORM, but the code `<span class="p">{</span><span class="err">{</span> <span class="w"></span> <span class="err">post.created_by.posts.count</span> <span class="w"></span> <span class="p">}</span><span class="err">}</span>` is executing a `select count` in the database. Even though the result is correct, it is a bad approach. Right now it’s causing several unnecessary queries in the database. But hey, don’t worry about that right now. Let’s focus on how we interact with the application. Later on, we are going to improve this code, and how to diagnose heavy queries.

Another interesting point here is that we are testing if the current post belongs to the authenticated user: `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">if</span> <span class="w"></span> <span class="err">post.created_by</span> <span class="w"></span> <span class="err">==</span> <span class="w"></span> <span class="err">user</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>`. And we are only showing the edit button for the owner of the post.

Since we now have the URL route to the topic posts listing, update the **topics.html** template with the link:

**templates/topics.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/cb4b7c9ff382ddeafb4114d0c84b3869)</small>

<figure class="highlight">

    {% for topic in board.topics.all %}
      <tr>
        <td><a href="{% url 'topic_posts' board.pk topic.pk %}">{{ topic.subject }}</a></td>
        <td>{{ topic.starter.username }}</td>
        <td>0</td>
        <td>0</td>
        <td>{{ topic.last_updated }}</td>
      </tr>
    {% endfor %}

</figure>

* * *