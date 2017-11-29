#### Reply Post View

Let’s implement now the reply post view so that we can add more data and progress with the implementation and tests.

![Reply Wireframes](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/wireframe-reply.png)

New URL route:

**myproject/urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/71a5f9f39202edfbab9bacf11844548b#file-urls-py-L39)</small>

<figure class="highlight">

    url(r'^boards/(?P<pk>\d+)/topics/(?P<topic_pk>\d+)/reply//figure>, views.reply_topic, name='reply_topic'),

</figure>

Create a new form for the post reply:

**boards/forms.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/3dd5ed2b3e27b4c12886e9426acf8fda#file-forms-py-L20)</small>

<figure class="highlight">

    from django import forms
    from .models import Post

    class PostForm(forms.ModelForm):
        class Meta:
            model = Post
            fields = ['message', ]

</figure>

A new view protected by `@login_required` and with a simple form processing logic:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/9e3811d9b11958b4106d99d9243efa71#file-views-py-L45)</small>

<figure class="highlight">

    from django.contrib.auth.decorators import login_required
    from django.shortcuts import get_object_or_404, redirect, render
    from .forms import PostForm
    from .models import Topic

    @login_required
    def reply_topic(request, pk, topic_pk):
        topic = get_object_or_404(Topic, board__pk=pk, pk=topic_pk)
        if request.method == 'POST':
            form = PostForm(request.POST)
            if form.is_valid():
                post = form.save(commit=False)
                post.topic = topic
                post.created_by = request.user
                post.save()
                return redirect('topic_posts', pk=pk, topic_pk=topic_pk)
        else:
            form = PostForm()
        return render(request, 'reply_topic.html', {'topic': topic, 'form': form})

</figure>

Also take the time to update the return redirect of the **new_topic** view function (marked with the comment **# TODO**).

<figure class="highlight">

    @login_required
    def new_topic(request, pk):
        board = get_object_or_404(Board, pk=pk)
        if request.method == 'POST':
            form = NewTopicForm(request.POST)
            if form.is_valid():
                topic = form.save(commit=False)
                # code suppressed ...
                return redirect('topic_posts', pk=pk, topic_pk=topic.pk)  # <- here
        # code suppressed ...

</figure>

Very important: in the view **reply_topic** we are using `topic_pk` because we are referring to the keyword argument of the function, in the view **new_topic** we are using `topic.pk` because a `topic` is an object (Topic model instance) and `.pk` we are accessing the `pk` property of the Topic model instance. Small detail, big difference.

The first version of our template:

**templates/reply_topic.html**

<figure class="highlight">

    {% extends 'base.html' %}

    {% load static %}

    {% block title %}Post a reply{% endblock %}

    {% block breadcrumb %}
      <li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
      <li class="breadcrumb-item"><a href="{% url 'board_topics' topic.board.pk %}">{{ topic.board.name }}</a></li>
      <li class="breadcrumb-item"><a href="{% url 'topic_posts' topic.board.pk topic.pk %}">{{ topic.subject }}</a></li>
      <li class="breadcrumb-item active">Post a reply</li>
    {% endblock %}

    {% block content %}

      <form method="post" class="mb-4">
        {% csrf_token %}
        {% include 'includes/form.html' %}
        <button type="submit" class="btn btn-success">Post a reply</button>
      </form>

      {% for post in topic.posts.all %}
        <div class="card mb-2">
          <div class="card-body p-3">
            <div class="row mb-3">
              <div class="col-6">
                <strong class="text-muted">{{ post.created_by.username }}</strong>
              </div>
              <div class="col-6 text-right">
                <small class="text-muted">{{ post.created_at }}</small>
              </div>
            </div>
            {{ post.message }}
          </div>
        </div>
      {% endfor %}

    {% endblock %}

</figure>

![Reply form](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/reply.png)

Then after posting a reply, the user is redirected back to the topic posts:

![Topic posts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-3.png)

We could now change the starter post, so to give it more emphasis in the page:

**templates/topic_posts.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/3e4ad94ac3ae9d72194af4006d4aeaff#file-topic_posts-html-L20)</small>

<figure class="highlight">

    {% for post in topic.posts.all %}
      <div class="card mb-2 {% if forloop.first %}border-dark{% endif %}">
        {% if forloop.first %}
          <div class="card-header text-white bg-dark py-2 px-3">{{ topic.subject }}</div>
        {% endif %}
        <div class="card-body p-3">
          <!-- code suppressed -->
        </div>
      </div>
    {% endfor %}

</figure>

![Topic posts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-4.png)

Now for the tests, pretty standard, just like we have been doing so far. Create a new file **test_view_reply_topic.py** inside the **boards/tests** folder:

**boards/tests/test_view_reply_topic.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/7148fcb95075fb6641e638214b751cf1)</small>

<figure class="highlight">

    from django.contrib.auth.models import User
    from django.test import TestCase
    from django.urls import reverse
    from ..models import Board, Post, Topic
    from ..views import reply_topic

    class ReplyTopicTestCase(TestCase):
        '''
        Base test case to be used in all `reply_topic` view tests
        '''
        def setUp(self):
            self.board = Board.objects.create(name='Django', description='Django board.')
            self.username = 'john'
            self.password = '123'
            user = User.objects.create_user(username=self.username, email='john@doe.com', password=self.password)
            self.topic = Topic.objects.create(subject='Hello, world', board=self.board, starter=user)
            Post.objects.create(message='Lorem ipsum dolor sit amet', topic=self.topic, created_by=user)
            self.url = reverse('reply_topic', kwargs={'pk': self.board.pk, 'topic_pk': self.topic.pk})

    class LoginRequiredReplyTopicTests(ReplyTopicTestCase):
        # ...

    class ReplyTopicTests(ReplyTopicTestCase):
        # ...

    class SuccessfulReplyTopicTests(ReplyTopicTestCase):
        # ...

    class InvalidReplyTopicTests(ReplyTopicTestCase):
        # ...

</figure>

The essence here is the custom test case class **ReplyTopicTestCase**. Then all the four classes will extend this test case.

First, we test if the view is protected with the `@login_required` decorator, then check the HTML inputs, status code. Finally, we test a valid and an invalid form submission.

* * *