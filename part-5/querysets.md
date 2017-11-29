#### QuerySets

Let’s take the time now to explore some of the models’ API functionalities a little bit. First, let’s improve the home view:

![Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/boards-1.png)

We have three tasks here:

*   Display the posts count of the board;
*   Display the topics count of the board;
*   Display the last user who posted something and the date and time.

Let’s play with the Python terminal first, before we jump into the implementation.

Since we are going to try things out in the Python terminal, it’s a good idea to define a `__str__` method for all our models.

**boards/models.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/9524eb42005697fbb79836285b50b1f4)</small>

<figure class="highlight">

    from django.db import models
    from django.utils.text import Truncator

    class Board(models.Model):
        # ...
        def __str__(self):
            return self.name

    class Topic(models.Model):
        # ...
        def __str__(self):
            return self.subject

    class Post(models.Model):
        # ...
        def __str__(self):
            truncated_message = Truncator(self.message)
            return truncated_message.chars(30)

</figure>

In the Post model we are using the **Truncator** utility class. It’s a convenient way to truncate long strings into an arbitrary string size (here we are using 30).

Now let’s open the Python shell terminal:

<figure class="highlight">

    python manage.py shell

    # First get a board instance from the database
    board = Board.objects.get(name='Django')

</figure>

The easiest of the three tasks is to get the current topics count, because the Topic and Board are directly related:

<figure class="highlight">

    board.topics.all()
    <QuerySet [<Topic: Hello everyone!>, <Topic: Test>, <Topic: Testing a new post>, <Topic: Hi>]>

    board.topics.count()
    4

</figure>

That’s about it.

Now the number of _posts_ within a _board_ is a little bit trickier because Post is not directly related to Board.

<figure class="highlight">

    from boards.models import Post

    Post.objects.all()
    <QuerySet [<Post: This is my first topic.. :-)>, <Post: test.>, <Post: Hi everyone!>,
      <Post: New test here!>, <Post: Testing the new reply feature!>, <Post: Lorem ipsum dolor sit amet,...>,
      <Post: hi there>, <Post: test>, <Post: Testing..>, <Post: some reply>, <Post: Random random.>
    ]>

    Post.objects.count()
    11

</figure>

Here we have 11 posts. But not all of them belongs to the “Django” board.

Here is how we can filter it:

<figure class="highlight">

    from boards.models import Board, Post

    board = Board.objects.get(name='Django')

    Post.objects.filter(topic__board=board)
    <QuerySet [<Post: This is my first topic.. :-)>, <Post: test.>, <Post: hi there>,
      <Post: Hi everyone!>, <Post: Lorem ipsum dolor sit amet,...>, <Post: New test here!>,
      <Post: Testing the new reply feature!>
    ]>

    Post.objects.filter(topic__board=board).count()
    7

</figure>

The double underscores `topic__board` is used to navigate through the models’ relationships. Under the hoods, Django builds the bridge between the Board - Topic - Post, and build a SQL query to retrieve just the posts that belong to a specific board.

Now our last mission is to identify the last post.

<figure class="highlight">

    # order by the `created_at` field, getting the most recent first
    Post.objects.filter(topic__board=board).order_by('-created_at')
    <QuerySet [<Post: testing>, <Post: new post>, <Post: hi there>, <Post: Lorem ipsum dolor sit amet,...>,
      <Post: Testing the new reply feature!>, <Post: New test here!>, <Post: Hi everyone!>,
      <Post: test.>, <Post: This is my first topic.. :-)>
    ]>

    # we can use the `first()` method to just grab the result that interest us
    Post.objects.filter(topic__board=board).order_by('-created_at').first()
    <Post: testing>

</figure>

Sweet. Now we can implement it.

**boards/models.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/74077336decd75292082752eb8405ad3)</small>

<figure class="highlight">

    from django.db import models

    class Board(models.Model):
        name = models.CharField(max_length=30, unique=True)
        description = models.CharField(max_length=100)

        def __str__(self):
            return self.name

        def get_posts_count(self):
            return Post.objects.filter(topic__board=self).count()

        def get_last_post(self):
            return Post.objects.filter(topic__board=self).order_by('-created_at').first()

</figure>

Observe that we are using `self`, because this method will be used by a Board instance. So that means we are using this instance to filter the QuerySet.

Now we can improve the home HTML template to display this brand new information:

**templates/home.html**

<figure class="highlight">

    {% extends 'base.html' %}

    {% block breadcrumb %}
      <li class="breadcrumb-item active">Boards</li>
    {% endblock %}

    {% block content %}
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
                <a href="{% url 'board_topics' board.pk %}">{{ board.name }}</a>
                <small class="text-muted d-block">{{ board.description }}</small>
              </td>
              <td class="align-middle">
                {{ board.get_posts_count }}
              </td>
              <td class="align-middle">
                {{ board.topics.count }}
              </td>
              <td class="align-middle">
                {% with post=board.get_last_post %}
                  <small>
                    <a href="{% url 'topic_posts' board.pk post.topic.pk %}">
                      By {{ post.created_by.username }} at {{ post.created_at }}
                    </a>
                  </small>
                {% endwith %}
              </td>
            </tr>
          {% endfor %}
        </tbody>
      </table>
    {% endblock %}

</figure>

And that’s the result for now:

![Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/boards-2.png)

Run the tests:

<figure class="highlight">

    python manage.py test

</figure>

<figure class="highlight">

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    .......................................................EEE......................
    ======================================================================
    ERROR: test_home_url_resolves_home_view (boards.tests.test_view_home.HomeTests)
    ----------------------------------------------------------------------
    django.urls.exceptions.NoReverseMatch: Reverse for 'topic_posts' with arguments '(1, '')' not found. 1 pattern(s) tried: ['boards/(?P<pk>\\d+)/topics/(?P<topic_pk>\\d+)//figure>]

    ======================================================================
    ERROR: test_home_view_contains_link_to_topics_page (boards.tests.test_view_home.HomeTests)
    ----------------------------------------------------------------------
    django.urls.exceptions.NoReverseMatch: Reverse for 'topic_posts' with arguments '(1, '')' not found. 1 pattern(s) tried: ['boards/(?P<pk>\\d+)/topics/(?P<topic_pk>\\d+)//figure>]

    ======================================================================
    ERROR: test_home_view_status_code (boards.tests.test_view_home.HomeTests)
    ----------------------------------------------------------------------
    django.urls.exceptions.NoReverseMatch: Reverse for 'topic_posts' with arguments '(1, '')' not found. 1 pattern(s) tried: ['boards/(?P<pk>\\d+)/topics/(?P<topic_pk>\\d+)//figure>]

    ----------------------------------------------------------------------
    Ran 80 tests in 5.663s

    FAILED (errors=3)
    Destroying test database for alias 'default'...

</figure>

It seems like we have a problem with our implementation here. The application is crashing if there are no posts.

**templates/home.html**

<figure class="highlight">

    {% with post=board.get_last_post %}
      {% if post %}
        <small>
          <a href="{% url 'topic_posts' board.pk post.topic.pk %}">
            By {{ post.created_by.username }} at {{ post.created_at }}
          </a>
        </small>
      {% else %}
        <small class="text-muted">
          <em>No posts yet.</em>
        </small>
      {% endif %}
    {% endwith %}

</figure>

Run the tests again:

<figure class="highlight">

    python manage.py test

</figure>

<figure class="highlight">

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    ................................................................................
    ----------------------------------------------------------------------
    Ran 80 tests in 5.630s

    OK
    Destroying test database for alias 'default'...

</figure>

I added a new board with no messages just to check the “empty message”:

![Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/boards-3.png)

Now it’s time to improve the topics listing view.

![Topics](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-3.png)

I will show you another way to include the count, this time to the number of replies, in a more effective way.

As usual, let’s try first with the Python shell:

<figure class="highlight">

    python manage.py shell

</figure>

<figure class="highlight">

    from django.db.models import Count
    from boards.models import Board

    board = Board.objects.get(name='Django')

    topics = board.topics.order_by('-last_updated').annotate(replies=Count('posts'))

    for topic in topics:
        print(topic.replies)

    2
    4
    2
    1

</figure>

Here we are using the `annotate` QuerySet method to generate a new “column” on the fly. This new column, which will be translated into a property, accessible via `topic.replies` contain the count of posts a given topic has.

We can do just a minor fix because the replies should not consider the starter topic (which is also a Post instance).

So here is how we do it:

<figure class="highlight">

    topics = board.topics.order_by('-last_updated').annotate(replies=Count('posts') - 1)

    for topic in topics:
        print(topic.replies)

    1
    3
    1
    0

</figure>

Cool, right?

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/f22b493b3e076aba9351c9d98f547f5e#file-views-py-L14)</small>

<figure class="highlight">

    from django.db.models import Count
    from django.shortcuts import get_object_or_404, render
    from .models import Board

    def board_topics(request, pk):
        board = get_object_or_404(Board, pk=pk)
        topics = board.topics.order_by('-last_updated').annotate(replies=Count('posts') - 1)
        return render(request, 'topics.html', {'board': board, 'topics': topics})

</figure>

**templates/topics.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/1a2235f05f436c92025dc86028c22fc4#file-topics-html-L28)</small>

<figure class="highlight">

    {% for topic in topics %}
      <tr>
        <td><a href="{% url 'topic_posts' board.pk topic.pk %}">{{ topic.subject }}</a></td>
        <td>{{ topic.starter.username }}</td>
        <td>{{ topic.replies }}</td>
        <td>0</td>
        <td>{{ topic.last_updated }}</td>
      </tr>
    {% endfor %}

</figure>

![Topics](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-4.png)

Next step now is to fix the _views_ count. But for that, we will need to create a new field.

* * *