#### Migrations

Migration is a fundamental part of Web development with Django. It’s how we evolve our application’s models keeping the models’ files synchronized with the database.

When we first run the command `python manage.py migrate` Django grab all migration files and generate the database schema.

When Django applies a migration, it has a special table called **django_migrations**. In this table, Django registers all the applied migrations.

So if we try to run the command again:

```

    python manage.py migrate

```

```

    Operations to perform:
      Apply all migrations: admin, auth, boards, contenttypes, sessions
    Running migrations:
      No migrations to apply.

```

Django will know there’s nothing to do.

Let’s create a migration by adding a new field to the Topic model:

**boards/models.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/816f47aa4df8e7b157df75e0ff209aac#file-models-py-L25)</small>

```

    class Topic(models.Model):
        subject = models.CharField(max_length=255)
        last_updated = models.DateTimeField(auto_now_add=True)
        board = models.ForeignKey(Board, related_name='topics')
        starter = models.ForeignKey(User, related_name='topics')
        views = models.PositiveIntegerField(default=0)  # <- here

        def __str__(self):
            return self.subject

```

Here we added a `PositiveIntegerField`. Since this field is going to store the number of page views, a negative page view wouldn’t make sense.

Before we can use our new field, we have to update the database schema. Execute the `makemigrations` command:

```

    python manage.py makemigrations

    Migrations for 'boards':
      boards/migrations/0003_topic_views.py
        - Add field views to topic

```

The `makemigrations` command automatically generated the **0003_topic_views.py** file, which will be used to modify the database, adding the **views** field.

Now apply the migration by running the command `migrate`:

```

    python manage.py migrate

    Operations to perform:
      Apply all migrations: admin, auth, boards, contenttypes, sessions
    Running migrations:
      Applying boards.0003_topic_views... OK

```

Now we can use it to keep track of the number of views a given topic is receiving:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/c0c97c1e050204d9152c59b4da2f9305#file-views-py-L41)</small>

```

    from django.shortcuts import get_object_or_404, render
    from .models import Topic

    def topic_posts(request, pk, topic_pk):
        topic = get_object_or_404(Topic, board__pk=pk, pk=topic_pk)
        topic.views += 1
        topic.save()
        return render(request, 'topic_posts.html', {'topic': topic})

```

**templates/topics.html** <small>[(view complete file contents)](https://gist.github.com/vitorfs/70ebb1a06e1044387943ee83bafcd526)</small>

```

    {% for topic in topics %}
      <tr>
        <td><a href="{% url 'topic_posts' board.pk topic.pk %}">{{ topic.subject }}</a></td>
        <td>{{ topic.starter.username }}</td>
        <td>{{ topic.replies }}</td>
        <td>{{ topic.views }}</td>  <!-- here -->
        <td>{{ topic.last_updated }}</td>
      </tr>
    {% endfor %}

```

Now open a topic and refresh the page a few times, and see if it’s counting the page views:

![Posts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-5.png)

* * *