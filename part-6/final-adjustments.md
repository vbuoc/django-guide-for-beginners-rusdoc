#### Final Adjustments

Maybe you have already noticed, but there’s a small issue when someone replies to a post. It’s not updating the `last_update` field, so the ordering of the topics is broken right now.

Let’s fix it:

**boards/views.py**

```

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

                topic.last_updated = timezone.now()  # <- here
                topic.save()                         # <- and here

                return redirect('topic_posts', pk=pk, topic_pk=topic_pk)
        else:
            form = PostForm()
        return render(request, 'reply_topic.html', {'topic': topic, 'form': form})

```

Next thing we want to do is try to control the view counting system a little bit more. We don’t want to the same user refreshing the page counting as multiple views. For this we can use sessions:

**boards/views.py**

```

    class PostListView(ListView):
        model = Post
        context_object_name = 'posts'
        template_name = 'topic_posts.html'
        paginate_by = 20

        def get_context_data(self, **kwargs):

            session_key = 'viewed_topic_{}'.format(self.topic.pk)  # <-- here
            if not self.request.session.get(session_key, False):
                self.topic.views += 1
                self.topic.save()
                self.request.session[session_key] = True           # <-- until here

            kwargs['topic'] = self.topic
            return super().get_context_data(**kwargs)

        def get_queryset(self):
            self.topic = get_object_or_404(Topic, board__pk=self.kwargs.get('pk'), pk=self.kwargs.get('topic_pk'))
            queryset = self.topic.posts.order_by('created_at')
            return queryset

```

Now we could provide a better navigation in the topics listing. Currently the only option is for the user to click in the topic title and go to the first page. We could workout something like this:

**boards/models.py**

```

    import math
    from django.db import models

    class Topic(models.Model):
        # ...

        def __str__(self):
            return self.subject

        def get_page_count(self):
            count = self.posts.count()
            pages = count / 20
            return math.ceil(pages)

        def has_many_pages(self, count=None):
            if count is None:
                count = self.get_page_count()
            return count > 6

        def get_page_range(self):
            count = self.get_page_count()
            if self.has_many_pages(count):
                return range(1, 5)
            return range(1, count + 1)

```

Then in the **topics.html** template we could implement something like this:

**templates/topics.html**

```

      <table class="table table-striped mb-4">
        <thead class="thead-inverse">
          <tr>
            <th>Topic</th>
            <th>Starter</th>
            <th>Replies</th>
            <th>Views</th>
            <th>Last Update</th>
          </tr>
        </thead>
        <tbody>
          {% for topic in topics %}
            {% url 'topic_posts' board.pk topic.pk as topic_url %}
            <tr>
              <td>
                <p class="mb-0">
                  <a href="{{ topic_url }}">{{ topic.subject }}</a>
                </p>
                <small class="text-muted">
                  Pages:
                  {% for i in topic.get_page_range %}
                    <a href="{{ topic_url }}?page={{ i }}">{{ i }}</a>
                  {% endfor %}
                  {% if topic.has_many_pages %}
                  ... <a href="{{ topic_url }}?page={{ topic.get_page_count }}">Last Page</a>
                  {% endif %}
                </small>
              </td>
              <td class="align-middle">{{ topic.starter.username }}</td>
              <td class="align-middle">{{ topic.replies }}</td>
              <td class="align-middle">{{ topic.views }}</td>
              <td class="align-middle">{{ topic.last_updated|naturaltime }}</td>
            </tr>
          {% endfor %}
        </tbody>
      </table>

```

Like a tiny pagination for each topic. Note that I also took the time to add the `table-striped` class for a better styling of the table.

![Pages](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/pages.png)

In the reply page, we are currently listing all topic replies. We could limit it to just the last ten posts.

**boards/models.py**

```

    class Topic(models.Model):
        # ...

        def get_last_ten_posts(self):
            return self.posts.order_by('-created_at')[:10]

```

**templates/reply_topic.html**

```

    {% block content %}

      <form method="post" class="mb-4" novalidate>
        {% csrf_token %}
        {% include 'includes/form.html' %}
        <button type="submit" class="btn btn-success">Post a reply</button>
      </form>

      {% for post in topic.get_last_ten_posts %}  <!-- here! -->
        <div class="card mb-2">
          <!-- code suppressed -->
        </div>
      {% endfor %}

    {% endblock %}

```

![Replies](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/replies.png)

Another thing is that when the user replies to a post, we are redirecting the user to the first page again. We could improve it by sending the user to the last page.

We can add an id to the post card:

**templates/topic_posts.html**

```

    {% block content %}

      <div class="mb-4">
        <a href="{% url 'reply_topic' topic.board.pk topic.pk %}" class="btn btn-primary" role="button">Reply</a>
      </div>

      {% for post in posts %}
        <div id="{{ post.pk }}" class="card {% if forloop.last %}mb-4{% else %}mb-2{% endif %} {% if forloop.first %}border-dark{% endif %}">
          <!-- code suppressed -->
        </div>
      {% endfor %}

      {% include 'includes/pagination.html' %}

    {% endblock %}

```

The important bit here is the `<div id="{{ post.pk }}" ...>`.

Then we can play with it like this in the views:

**boards/views.py**

```

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

                topic.last_updated = timezone.now()
                topic.save()

                topic_url = reverse('topic_posts', kwargs={'pk': pk, 'topic_pk': topic_pk})
                topic_post_url = '{url}?page={page}#{id}'.format(
                    url=topic_url,
                    id=post.pk,
                    page=topic.get_page_count()
                )

                return redirect(topic_post_url)
        else:
            form = PostForm()
        return render(request, 'reply_topic.html', {'topic': topic, 'form': form})

```

In the **topic_post_url** we are building a URL with the last page and adding an anchor to the element with id equals to the post ID.

With this, it will required us to update the following test case:

**boards/tests/test_view_reply_topic.py**

```

    class SuccessfulReplyTopicTests(ReplyTopicTestCase):
        # ...

        def test_redirection(self):
            '''
            A valid form submission should redirect the user
            '''
            url = reverse('topic_posts', kwargs={'pk': self.board.pk, 'topic_pk': self.topic.pk})
            topic_posts_url = '{url}?page=1#2'.format(url=url)
            self.assertRedirects(self.response, topic_posts_url)

```

![Replies](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/reply-last-page.png)

Next issue, as you can see in the previous screenshot, is to solve the problem with the pagination when the number of pages is too high.

The easiest way is to tweak the **pagination.html** template:

**templates/includes/pagination.html**

```

    {% if is_paginated %}
      <nav aria-label="Topics pagination" class="mb-4">
        <ul class="pagination">
          {% if page_obj.number > 1 %}
            <li class="page-item">
              <a class="page-link" href="?page=1">First</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">First</span>
            </li>
          {% endif %}

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
            {% elif page_num > page_obj.number|add:'-3' and page_num < page_obj.number|add:'3' %}
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

          {% if page_obj.number != paginator.num_pages %}
            <li class="page-item">
              <a class="page-link" href="?page={{ paginator.num_pages }}">Last</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">Last</span>
            </li>
          {% endif %}
        </ul>
      </nav>
    {% endif %}

```

![Long pages](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/long-pages.png)

* * *