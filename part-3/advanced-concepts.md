# Продвинутые концепции > Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/18/a-complete-beginners-guide-to-django-part-3.html

#### Introduction

In this tutorial, we are going to dive deep into two fundamental concepts: URLs and Forms. In the process, we are going to explore many other concepts like creating reusable templates and installing third-party libraries. We are also going to write plenty of unit tests.

If you are following this tutorial series since the first part, coding your project and following the tutorial step by step, you may need to update your **models.py** before starting:

**boards/models.py**

```

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

```

    python manage.py makemigrations
    python manage.py migrate

```

If you already have `null=True` in the `updated_by` field and the `auto_now_add=True` in the `last_updated` field, you can safely ignore the instructions above.

If you prefer to use my source code as a starting point, you can grab it on GitHub.

The current state of the project can be found under the release tag **v0.2-lw**. The link below will take you to the right place:

[https://github.com/sibtc/django-beginners-guide/tree/v0.2-lw](https://github.com/sibtc/django-beginners-guide/tree/v0.2-lw)

The development will follow from here.

* * *

h4 id="conclusions">Conclusions

In this tutorial, we focused on URLs, Reusable Templates, and Forms. As usual, we also implement several test cases. That’s how we develop with confidence.

Our tests file is starting to get big, so in the next tutorial, we are going to refactor it to improve the maintainability so to sustain the growth of our code base.

We are also reaching a point where we need to interact with the logged in user. In the next tutorial, we are going to learn everything about authentication and how to protect our views and resources.

I hope you enjoyed the third part of this tutorial series! The fourth part is coming out next week, on Sep 25, 2017. If you would like to get notified when the fourth part is out, you can [subscribe to our mailing list](http://eepurl.com/b0gR51).

The source code of the project is available on GitHub. The current state of the project can be found under the release tag **v0.3-lw**. The link below will take you to the right place:

[https://github.com/sibtc/django-beginners-guide/tree/v0.3-lw](https://github.com/sibtc/django-beginners-guide/tree/v0.3-lw)