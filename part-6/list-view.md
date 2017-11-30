#### List View

We could refactor some of our existing views to take advantage of the CBV capabilities. Take the home page for example. We are just grabbing all the boards from the database and listing it in the HTML:

**boards/views.py**

```

    from django.shortcuts import render
    from .models import Board

    def home(request):
        boards = Board.objects.all()
        return render(request, 'home.html', {'boards': boards})

```

Here is how we could rewrite it using a GCBV for models listing:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/5e248d9a4499e2a796c6ffee9cbb1125#file-views-py-L12)</small>

```

    from django.views.generic import ListView
    from .models import Board

    class BoardListView(ListView):
        model = Board
        context_object_name = 'boards'
        template_name = 'home.html'

```

Then we have to change the reference in the **urls.py** module:

**myproject/urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/a0a826233d95a4cc53a75afc441db1e9#file-urls-py-L10)</small>

```

    from django.conf.urls import url
    from boards import views

    urlpatterns = [
        url(r'^/figure>, views.BoardListView.as_view(), name='home'),
        # ...
    ]

```

If we check the homepage we will see that nothing really changed, everything is working as expected. But we have to tweak our tests a little bit because now we are dealing with a class-based view:

**boards/tests/test_view_home.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/e17216ce3d92110cf7e005ce3288c587)</small>

```

    from django.test import TestCase
    from django.urls import resolve
    from ..views import BoardListView

    class HomeTests(TestCase):
        # ...
        def test_home_url_resolves_home_view(self):
            view = resolve('/')
            self.assertEquals(view.func.view_class, BoardListView)

```

##### Pagination

We can very easily implement pagination with class-based views. But first I wanted to do a pagination by hand, so that we can explore better the mechanics behind it, so it doesn’t look like magic.

It wouldn’t really make sense to paginate the boards listing view because we do not expect to have many boards. But definitely the topics listing and the posts listing need some pagination.

From now on, we will be working on the **board_topics** view.

First, let’s add some volume of posts. We could just use the application’s user interface and add several posts, or open the Python shell and write a small script to do it for us:

```

    python manage.py shell

```

```

    from django.contrib.auth.models import User
    from boards.models import Board, Topic, Post

    user = User.objects.first()

    board = Board.objects.get(name='Django')

    for i in range(100):
        subject = 'Topic test #{}'.format(i)
        topic = Topic.objects.create(subject=subject, board=board, starter=user)
        Post.objects.create(message='Lorem ipsum...', topic=topic, created_by=user)

```

![Topics](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/topics.png)

Good, now we have some data to play with.

Before we jump into the code, let’s experiment a little bit more with the Python shell:

```

    python manage.py shell

```

```

    from boards.models import Topic

    # All the topics in the app
    Topic.objects.count()
    107

    # Just the topics in the Django board
    Topic.objects.filter(board__name='Django').count()
    104

    # Let's save this queryset into a variable to paginate it
    queryset = Topic.objects.filter(board__name='Django').order_by('-last_updated')

```

It’s very important always define an ordering to a **QuerySet** you are going to paginate! Otherwise, it can give you inconsistent results.

Now let’s import the **Paginator** utility:

```

    from django.core.paginator import Paginator

    paginator = Paginator(queryset, 20)

```

Here we are telling Django to paginate our **QuerySet** in pages of 20 each. Now let’s explore some of the paginator properties:

```

    # count the number of elements in the paginator
    paginator.count
    104

    # total number of pages
    # 104 elements, paginating 20 per page gives you 6 pages
    # where the last page will have only 4 elements
    paginator.num_pages
    6

    # range of pages that can be used to iterate and create the
    # links to the pages in the template
    paginator.page_range
    range(1, 7)

    # returns a Page instance
    paginator.page(2)
    <Page 2 of 6>

    page = paginator.page(2)

    type(page)
    django.core.paginator.Page

    type(paginator)
    django.core.paginator.Paginator

```

Here we have to pay attention because if we try to get a page that doesn’t exist, the Paginator will throw an exception:

```

    paginator.page(7)
    EmptyPage: That page contains no results

```

Or if we try to pass an arbitrary parameter, which is not a page number:

```

    paginator.page('abc')
    PageNotAnInteger: That page number is not an integer

```

We have to keep those details in mind when designing the user interface.

Now let’s explore the attributes and methods offered by the **Page** class a little bit:

```

    page = paginator.page(1)

    # Check if there is another page after this one
    page.has_next()
    True

    # If there is no previous page, that means this one is the first page
    page.has_previous()
    False

    page.has_other_pages()
    True

    page.next_page_number()
    2

    # Take care here, since there is no previous page,
    # if we call the method `previous_page_number() we will get an exception:
    page.previous_page_number()
    EmptyPage: That page number is less than 1

```

##### FBV Pagination

Here is how we do it using regular function-based views:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/16f0ac257439245fa6645af259d8846f#file-views-py-L19)</small>

```

    from django.db.models import Count
    from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
    from django.shortcuts import get_object_or_404, render
    from django.views.generic import ListView
    from .models import Board

    def board_topics(request, pk):
        board = get_object_or_404(Board, pk=pk)
        queryset = board.topics.order_by('-last_updated').annotate(replies=Count('posts') - 1)
        page = request.GET.get('page', 1)

        paginator = Paginator(queryset, 20)

        try:
            topics = paginator.page(page)
        except PageNotAnInteger:
            # fallback to the first page
            topics = paginator.page(1)
        except EmptyPage:
            # probably the user tried to add a page number
            # in the url, so we fallback to the last page
            topics = paginator.page(paginator.num_pages)

        return render(request, 'topics.html', {'board': board, 'topics': topics})

```

Now the trick part is to render the pages correctly using the Bootstrap 4 pagination component. But take the time to read the code and see if it makes sense for you. We are using here all the methods we played with before. And here in that context, `topics` is no longer a QuerySet but a `paginator.Page` instance.

Right after the topics HTML table, we can render the pagination component:

**templates/topics.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/3101a1bd72125aeb45829659a5532bc6)</small>

```

    {% if topics.has_other_pages %}
      <nav aria-label="Topics pagination" class="mb-4">
        <ul class="pagination">
          {% if topics.has_previous %}
            <li class="page-item">
              <a class="page-link" href="?page={{ topics.previous_page_number }}">Previous</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">Previous</span>
            </li>
          {% endif %}

          {% for page_num in topics.paginator.page_range %}
            {% if topics.number == page_num %}
              <li class="page-item active">
                <span class="page-link">
                  {{ page_num }}
                  <span class="sr-only">(current)</span>
                </span>
              </li>
            {% else %}
              <li class="page-item">
                <a class="page-link" href="?page={{ page_num }}">{{ page_num }}</a>
              </li>
            {% endif %}
          {% endfor %}

          {% if topics.has_next %}
            <li class="page-item">
              <a class="page-link" href="?page={{ topics.next_page_number }}">Next</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">Next</span>
            </li>
          {% endif %}
        </ul>
      </nav>
    {% endif %}

```

![Pagination](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/paginate.png)

##### GCBV Pagination

Below, the same implementation but this time using the **ListView**.

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/323173bc56dec48fd83caf983d459421#file-views-py-L19)</small>

```

    class TopicListView(ListView):
        model = Topic
        context_object_name = 'topics'
        template_name = 'topics.html'
        paginate_by = 20

        def get_context_data(self, **kwargs):
            kwargs['board'] = self.board
            return super().get_context_data(**kwargs)

        def get_queryset(self):
            self.board = get_object_or_404(Board, pk=self.kwargs.get('pk'))
            queryset = self.board.topics.order_by('-last_updated').annotate(replies=Count('posts') - 1)
            return queryset

```

While using pagination with class-based views, the way we interact with the paginator in the template is a little bit different. It will make available the following variables in the template: **paginator**, **page_obj**, **is_paginated**, **object_list**, and also a variable with the name we defined in the **context_object_name**. In our case this extra variable will be named **topics**, and it will be equivalent to **object_list**.

Now about the whole **get_context_data** thing, well, that’s how we add stuff to the request context when extending a GCBV.

But the main point here is the **paginate_by** attribute. In some cases, just by adding it will be enough.

Remember to update the **urls.py**:

**myproject/urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/61f5345b7cf8b006b2901a61b8f8e348#file-urls-py-L37)</small>

```

    from django.conf.urls import url
    from boards import views

    urlpatterns = [
        # ...
        url(r'^boards/(?P<pk>\d+)//figure>, views.TopicListView.as_view(), name='board_topics'),
    ]

```

Now let’s fix the template:

**templates/topics.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/65095aa3eda78bafd22d5e2f94086d40#file-topics-html-L40)</small>

```

    {% block content %}
      <div class="mb-4">
        <a href="{% url 'new_topic' board.pk %}" class="btn btn-primary">New topic</a>
      </div>

      <table class="table mb-4">
        <!-- table content suppressed -->
      </table>

      {% if is_paginated %}
        <nav aria-label="Topics pagination" class="mb-4">
          <ul class="pagination">
            {% if page_obj.has_previous %}
              <li class="page-item">
                <a class="page-link" href="?page={{ page_obj.previous_page_number }}">Previous</a>
              </li>
            {% else %}
              <li class="page-item disabled">
                <span class="page-link">Previous</span>
              </li>
            {% endif %}

            {% for page_num in paginator.page_range %}
              {% if page_obj.number == page_num %}
                <li class="page-item active">
                  <span class="page-link">
                    {{ page_num }}
                    <span class="sr-only">(current)</span>
                  </span>
                </li>
              {% else %}
                <li class="page-item">
                  <a class="page-link" href="?page={{ page_num }}">{{ page_num }}</a>
                </li>
              {% endif %}
            {% endfor %}

            {% if page_obj.has_next %}
              <li class="page-item">
                <a class="page-link" href="?page={{ page_obj.next_page_number }}">Next</a>
              </li>
            {% else %}
              <li class="page-item disabled">
                <span class="page-link">Next</span>
              </li>
            {% endif %}
          </ul>
        </nav>
      {% endif %}

    {% endblock %}

```

Now take the time to run the tests and fix if needed.

**boards/tests/test_view_board_topics.py**

```

    from django.test import TestCase
    from django.urls import resolve
    from ..views import TopicListView

    class BoardTopicsTests(TestCase):
        # ...
        def test_board_topics_url_resolves_board_topics_view(self):
            view = resolve('/boards/1/')
            self.assertEquals(view.func.view_class, TopicListView)

```

##### Reusable Pagination Template

Just like we did with the **form.html** partial template, we can also create something similar for the pagination HTML snippet.

Let’s paginate the topic posts page, and then find a way to reuse the pagination component.

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/53139ca0fd7c01b8459c2ff62828f963)</small>

```

    class PostListView(ListView):
        model = Post
        context_object_name = 'posts'
        template_name = 'topic_posts.html'
        paginate_by = 2

        def get_context_data(self, **kwargs):
            self.topic.views += 1
            self.topic.save()
            kwargs['topic'] = self.topic
            return super().get_context_data(**kwargs)

        def get_queryset(self):
            self.topic = get_object_or_404(Topic, board__pk=self.kwargs.get('pk'), pk=self.kwargs.get('topic_pk'))
            queryset = self.topic.posts.order_by('created_at')
            return queryset

```

Now update the **urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/428d58dbb61f9c2601bb7434150ea37f)</small>

```

    from django.conf.urls import url
    from boards import views

    urlpatterns = [
        # ...
        url(r'^boards/(?P<pk>\d+)/topics/(?P<topic_pk>\d+)//figure>, views.PostListView.as_view(), name='topic_posts'),
    ]

```

Now we grab that pagination HTML snippet from the **topics.html** template, and create a new file named **pagination.html** inside the **templates/includes** folder, alongside with the **forms.html** file:

```

    myproject/
     |-- myproject/
     |    |-- accounts/
     |    |-- boards/
     |    |-- myproject/
     |    |-- static/
     |    |-- templates/
     |    |    |-- includes/
     |    |    |    |-- form.html
     |    |    |    +-- pagination.html  <-- here!
     |    |    +-- ...
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

```

**templates/includes/pagination.html**

```

    {% if is_paginated %}
      <nav aria-label="Topics pagination" class="mb-4">
        <ul class="pagination">
          {% if page_obj.has_previous %}
            <li class="page-item">
              <a class="page-link" href="?page={{ page_obj.previous_page_number }}">Previous</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">Previous</span>
            </li>
          {% endif %}

          {% for page_num in paginator.page_range %}
            {% if page_obj.number == page_num %}
              <li class="page-item active">
                <span class="page-link">
                  {{ page_num }}
                  <span class="sr-only">(current)</span>
                </span>
              </li>
            {% else %}
              <li class="page-item">
                <a class="page-link" href="?page={{ page_num }}">{{ page_num }}</a>
              </li>
            {% endif %}
          {% endfor %}

          {% if page_obj.has_next %}
            <li class="page-item">
              <a class="page-link" href="?page={{ page_obj.next_page_number }}">Next</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">Next</span>
            </li>
          {% endif %}
        </ul>
      </nav>
    {% endif %}

```

Now in the **topic_posts.html** template we use it:

**templates/topic_posts.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/df5b16bb16c1134ba4e03218dce250d7)</small>

```

    {% block content %}

      <div class="mb-4">
        <a href="{% url 'reply_topic' topic.board.pk topic.pk %}" class="btn btn-primary" role="button">Reply</a>
      </div>

      {% for post in posts %}
        <div class="card {% if forloop.last %}mb-4{% else %}mb-2{% endif %} {% if forloop.first %}border-dark{% endif %}">
          {% if forloop.first %}
            <div class="card-header text-white bg-dark py-2 px-3">{{ topic.subject }}</div>
          {% endif %}
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
                    <a href="{% url 'edit_post' post.topic.board.pk post.topic.pk post.pk %}"
                       class="btn btn-primary btn-sm"
                       role="button">Edit</a>
                  </div>
                {% endif %}
              </div>
            </div>
          </div>
        </div>
      {% endfor %}

      {% include 'includes/pagination.html' %}

    {% endblock %}

```

Don’t forget to change the main forloop to `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">for</span> <span class="w"></span> <span class="err">post</span> <span class="w"></span> <span class="err">in</span> <span class="w"></span> <span class="err">posts</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>`.

We can also update the previous template, the **topics.html** template to use the pagination partial template too:

**templates/topics.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/9198ad8f91cd889f315ade7e4eb62710#file-topics-html-L40)</small>

```

    {% block content %}
      <div class="mb-4">
        <a href="{% url 'new_topic' board.pk %}" class="btn btn-primary">New topic</a>
      </div>

      <table class="table mb-4">
        <!-- table code suppressed -->
      </table>

      {% include 'includes/pagination.html' %}

    {% endblock %}

```

Just for testing purpose, you could just add a few posts (or create some using the Python Shell) and change the **paginate_by** to a low number, say 2, and see how it’s looking like:

![Topics](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/topics-1.png) <small>[(view complete file contents)](https://gist.github.com/vitorfs/6b3cd0769f805ab38626f5bd97b4e5e3)</small>

Update the test cases:

**boards/tests/test_view_topic_posts.py**

```

    from django.test import TestCase
    from django.urls import resolve
    from ..views import PostListView

    class TopicPostsTests(TestCase):
        # ...
        def test_view_function(self):
            view = resolve('/boards/1/topics/1/')
            self.assertEquals(view.func.view_class, PostListView)

```

* * *