Models

The models are basically a representation of your application’s database layout. What we are going to do in this section is create the Django representation of the classes we modeled in the previous section: Board, Topic, and Post. The User model is already defined inside a built-in app named auth, which is listed in our INSTALLED_APPS configuration under the namespace django.contrib.auth.

We will do all the work inside the boards/models.py file. Here is how we represent our class diagram (see Figure 4). in a Django application:

```python
from django.db import models
from django.contrib.auth.models import User


class Board(models.Model):
    name = models.CharField(max_length=30, unique=True)
    description = models.CharField(max_length=100)


class Topic(models.Model):
    subject = models.CharField(max_length=255)
    last_updated = models.DateTimeField(auto_now_add=True)
    board = models.ForeignKey(Board, related_name='topics')
    starter = models.ForeignKey(User, related_name='topics')


class Post(models.Model):
    message = models.TextField(max_length=4000)
    topic = models.ForeignKey(Topic, related_name='posts')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(null=True)
    created_by = models.ForeignKey(User, related_name='posts')
    updated_by = models.ForeignKey(User, null=True, related_name='+')
```

All models are subclass of the django.db.models.Model class. Each class will be transformed into database tables. Each field is represented by instances of django.db.models.Field subclasses (built-in Django core) and will be translated into database columns.

The fields CharField, DateTimeField, etc., are all subclasses of django.db.models.Field and they come included in the Django core – ready to be used.

Here we are only using CharField, TextField, DateTimeField, and ForeignKey fields to define our models. But Django offers a wide range of options to represent different types of data, such as IntegerField, BooleanField, DecimalField, and many others. We will refer to them as we need.

Some fields have required arguments, such as the CharField. We should always set a max_length. This information will be used to create the database column. Django needs to know how big the database column needs to be. The max_length parameter will also be used by the Django Forms API, to validate user input. More on that later.

In the Board model definition, more specifically in the name field, we are also setting the parameter unique=True, as the name suggests, it will enforce the uniqueness of the field at the database level.

In the Post model, the created_at field has an optional parameter, the auto_now_add set to True. This will instruct Django to set the current date and time when a Post object is created.

One way to create a relationship between the models is by using the ForeignKey field. It will create a link between the models and create a proper relationship at the database level. The ForeignKey field expects a positional parameter with the reference to the model it will relate to.

For example, in the Topic model, the board field is a ForeignKey to the Board model. It is telling Django that a Topic instance relates to only one Board instance. The related_name parameter will be used to create a reverse relationship where the Board instances will have access a list of Topic instances that belong to it.

Django automatically creates this reverse relationship – the related_name is optional. But if we don’t set a name for it, Django will generate it with the name: (class_name)_set. For example, in the Board model, the Topic instances would be available under the topic_set property. Instead, we simply renamed it to topics, to make it feel more natural.

In the Post model, the updated_by field sets the related_name='+'. This instructs Django that we don’t need this reverse relationship, so it will ignore it.

Below you can see the comparison between the class diagram and the source code to generate the models with Django. The green lines represent how we are handling the reverse relationships.

Class Diagram Models Definition

At this point, you may be asking yourself: “what about primary keys/IDs”? If we don’t specify a primary key for a model, Django will automatically generate it for us. So we are good for now. In the next section, you will see better how it works.

Migrating the Models

The next step is to tell Django to create the database so we can start using it.

Open the Terminal , activate the virtual environment, go to the folder where the manage.py file is, and run the commands below:

python manage.py makemigrations
As an output you will get something like this:

```bash
Migrations for 'boards':
  boards/migrations/0001_initial.py
    - Create model Board
    - Create model Post
    - Create model Topic
    - Add field topic to post
    - Add field updated_by to post
```

At this point, Django created a file named 0001_initial.py inside the boards/migrations directory. It represents the current state of our application’s models. In the next step, Django will use this file to create the tables and columns.

The migration files are translated into SQL statements. If you are familiar with SQL, you can run the following command to inspect the SQL instructions that will be executed in the database:

python manage.py sqlmigrate boards 0001
If you’re not familiar with SQL, don’t worry. We won’t be working directly with SQL in this tutorial series. All the work will be done using just the Django ORM, which is an abstraction layer that communicates with the database.

The next step now is to apply the migration we generated to the database:

```bash
python manage.py migrate
```

The output should be something like this:

```bash
Operations to perform:
  Apply all migrations: admin, auth, boards, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying boards.0001_initial... OK
  Applying sessions.0001_initial... OK
```

Because this is the first time we are migrating the database, the migrate command also applied the existing migration files from the Django contrib apps, listed in the INSTALLED_APPS. This is expected.

The line Applying boards.0001_initial... OK is the migration we generated in the previous step.

That’s it! Our database is ready to be used.

SQLite

> Note: It's important to note that SQLite is a production-quality database. SQLite is used by many companies across thousands of products, like all Android and iOS devices, all major Web browsers, Windows 10, macOS, etc.

It's just not suitable for all cases. SQLite doesn't compare with databases like MySQL, PostgreSQL or Oracle. High-volume websites, write-intensive applications, very large datasets, high concurrency, are some situations that will eventually result in a problem by using SQLite.

We are going to use SQLite during the development of our project because it's convenient and we won't need to install anything else. When we deploy our project to production, we will switch to PostgreSQL. For simple websites this work fine. But for complex websites, it's advisable to use the same database for development and production.
Experimenting with the Models API

One of the great advantages of developing with Python is the interactive shell. I use it all the time. It’s a quick way to try things out and experiment libraries and APIs.

You can start a Python shell with our project loaded using the manage.py utility:

```bash
python manage.py shell
Python 3.6.2 (default, Jul 17 2017, 23:14:31)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>>
```

This is very similar to calling the interactive console just by typing python, except when we use python manage.py shell, we are adding our project to the sys.path and loading Django. That means we can import our models and any other resource within the project and play with it.

Let’s start by importing the Board class:

```python
from boards.models import Board
```

To create a new board object, we can do the following:

```python
board = Board(name='Django', description='This is a board about Django.')
```

To persist this object in the database, we have to call the save method:

```python
board.save()
```

The save method is used both to create and update objects. Here Django created a new object because the Board instance had no id. After saving it for the first time, Django will set the id automatically:

board.id
1
You can access the rest of the fields as Python attributes:

board.name
'Django'
board.description
'This is a board about Django.'
To update a value we could do:

board.description = 'Django discussion board.'
board.save()
Every Django model comes with a special attribute; we call it a Model Manager. You can access it via the Python attribute objects. It is used mainly to execute queries in the database. For example, we could use it to directly create a new Board object:

board = Board.objects.create(name='Python', description='General discussion about Python.')
board.id
2
board.name
'Python'
So, right now we have two boards. We can use the objects to list all existing boards in the database:

Board.objects.all()
<QuerySet [<Board: Board object>, <Board: Board object>]>
The result was a QuerySet. We will learn more about that later on. Basically, it’s a list of objects from the database. We can see that we have two objects, but we can only read Board object. That’s because we haven’t defined the __str__ method in the Board model.

The __str__ method is a String representation of an object. We can use the board name to represent it.

First, exit the interactive console:

exit()
Now edit the models.py file inside the boards app:

```python
class Board(models.Model):
    name = models.CharField(max_length=30, unique=True)
    description = models.CharField(max_length=100)

    def __str__(self):
        return self.name
```

Let’s try the query again. Open the interactive console again:

```bash
python manage.py shell
```

```python
from boards.models import Board

Board.objects.all()
<QuerySet [<Board: Django>, <Board: Python>]>
```

Much better, right?

We can treat this QuerySet like a list. Let’s say we wanted to iterate over it and print the description of each board:

```python
boards_list = Board.objects.all()
for board in boards_list:
    print(board.description)
```

The result would be:

Django discussion board.
General discussion about Python.
Similarly, we can use the model Manager to query the database and return a single object. For that we use the get method:

```python
django_board = Board.objects.get(id=1)

django_board.name
```

```bash
'Django'
```

But we have to be careful with this kind of operation. If we try to get an object that doesn’t exist, for example, a board with id=3, it will raise an exception:

```python
board = Board.objects.get(id=3)
```

boards.models.DoesNotExist: Board matching query does not exist.
We can use the get method with any model field, but preferably use a field that can uniquely identify an object. Otherwise, the query may return more than one object, which will cause an exception.

```python
Board.objects.get(name='Django')
<Board: Django>
```

Note that the query is case sensitive, a lower case “django” would not match:

```python
Board.objects.get(name='django')
```
```bash
boards.models.DoesNotExist: Board matching query does not exist.
```

Summary of Model’s Operations

Find below a summary of the methods and operations we learned in this section, using the Board model as a reference. Uppercase Board refers to the class, lowercase board refers to an instance (or object) of the Board model class:

Operation	Code sample
Create an object without saving	board = Board()
Save an object (create or update)	board.save()
Create and save an object in the database	Board.objects.create(name='...', description='...')
List all objects	Board.objects.all()
Get a single object, identified by a field	Board.objects.get(id=1)
In the next section, we are going to start writing views and displaying our boards in HTML pages.

> [Представления, шаблоны, статика](part-2/views-templates-static-files.md)
