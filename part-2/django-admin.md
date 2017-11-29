Introduction to Django Admin > Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html

When we start a new project, Django already comes configured with the **Django Admin** listed in the `INSTALLED_APPS`.

![Django Admin Comic](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Django_Admin.png)

A good use case of the Django Admin is for example in a blog; it can be used by the authors to write and publish articles. Another example is an e-commerce website, where the staff members can create, edit, delete products.

For now, we are going to configure the Django Admin to maintain our application’s boards.

Let’s start by creating an administrator account:

<figure class="highlight">

    python manage.py createsuperuser

</figure>

Follow the instructions:

<figure class="highlight">

    Username (leave blank to use 'vitorfs'): admin
    Email address: admin@example.com
    Password:
    Password (again):
    Superuser created successfully.

</figure>

Now open the URL in a web browser: **http://127.0.0.1:8000/admin/**

![Django Admin Login](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-login.png)

Enter the **username** and **password** to log into the administration interface:

![Django Admin](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin.png)

It already comes with some features configured. Here we can add **Users** and **Groups** to manage permissions. We will explore more of those concepts later on.

To add the **Board** model is very straightforward. Open the **admin.py** file in the **boards** directory, and add the following code:

**boards/admin.py**

<figure class="highlight">

    from django.contrib import admin
    from .models import Board

    admin.site.register(Board)

</figure>

Save the **admin.py** file, and refresh the page on your web browser:

![Django Admin Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards.png)

And that’s it! It’s ready to be used. Click on the **Boards** link to see the list of existing boards:

![Django Admin Boards List](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-list.png)

We can add a new board by clicking on the **Add Board** button:

![Django Admin Boards Add](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-add.png)

Click on the **save** button:

![Django Admin Boards List](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-list-2.png)

We can check if everything is working be opening the **http://127.0.0.1:8000** URL:

![Boards Homepage](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-bootstrap-3.png)

* * *