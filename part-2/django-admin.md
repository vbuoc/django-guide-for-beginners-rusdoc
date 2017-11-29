
Introduction to Django Admin

When we start a new project, Django already comes configured with the Django Admin listed in the INSTALLED_APPS.

Django Admin Comic

A good use case of the Django Admin is for example in a blog; it can be used by the authors to write and publish articles. Another example is an e-commerce website, where the staff members can create, edit, delete products.

For now, we are going to configure the Django Admin to maintain our application’s boards.

Let’s start by creating an administrator account:

python manage.py createsuperuser
Follow the instructions:

Username (leave blank to use 'vitorfs'): admin
Email address: admin@example.com
Password:
Password (again):
Superuser created successfully.
Now open the URL in a web browser: http://127.0.0.1:8000/admin/

Django Admin Login

Enter the username and password to log into the administration interface:

Django Admin

It already comes with some features configured. Here we can add Users and Groups to manage permissions. We will explore more of those concepts later on.

To add the Board model is very straightforward. Open the admin.py file in the boards directory, and add the following code:

boards/admin.py

from django.contrib import admin
from .models import Board

admin.site.register(Board)
Save the admin.py file, and refresh the page on your web browser:

Django Admin Boards

And that’s it! It’s ready to be used. Click on the Boards link to see the list of existing boards:

Django Admin Boards List

We can add a new board by clicking on the Add Board button:

Django Admin Boards Add

Click on the save button:

Django Admin Boards List

We can check if everything is working be opening the http://127.0.0.1:8000 URL:

Boards Homepage

Conclusions

In this tutorial, we explored many new concepts. We defined some requirements for our project, created the first models, migrated the database, started playing with the Models API. We created our very first view and wrote some unit tests. We also configured the Django Template Engine, Static Files, and added the Bootstrap 4 library to the project. Finally, we had a very brief introduction the Django Admin interface.

I hope you enjoyed the second part of this tutorial series! The third part is coming out next week, on Sep 18, 2017. In the next part, we are going to explore Django’s URL routing, the forms API, reusable templates, and more testing. If you would like to get notified when the third part is out, you can subscribe to our mailing list.

The source code of the project is available on GitHub. The current state of the project can be found under the release tag v0.2-lw. The link below will take you to the right place:

https://github.com/sibtc/django-beginners-guide/tree/v0.2-lw

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/04/a-complete-beginners-guide-to-django-part-1.html
