#### Project Settings

No matter if the code is stored in a public or private remote repository, sensitive information should **never** be committed and pushed to the remote repository. That includes secret keys, passwords, API keys, etc.

At this point, we have to deal with two specific types of configuration in our **settings.py** module:

*   Sensitive information such as keys and passwords;
*   Configurations that are specific to a given environment.

Passwords and keys can be stored in environment variables or using local files (not committed to the remote repository):

```
# environment variables
import os
SECRET_KEY = os.environ['SECRET_KEY']

# or local files
with open('/etc/secret_key.txt') as f:
    SECRET_KEY = f.read().strip()
```

For that, there’s a great utility library called [Python Decouple](/2015/11/26/package-of-the-week-python-decouple.html) that I use in every single Django project I develop. It will search for a local file named **.env** to set the configuration variables and will fall back to the environment variables. It also provides an interface to define default values, transform the data into **int**, **bool**, and **list** when applicable.

It’s not mandatory, but I really find it a very useful tool. And it works like a charm with services like Heroku.

First, let’s install it:

```
pip install python-decouple
```

**myproject/settings.py**

```
from decouple import config

SECRET_KEY = config('SECRET_KEY')
```

Now we can place the sensitive information in a special file named **.env** (notice the dot in front) in the same directory where the **manage.py** file is:

```
myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |-- myproject/
 |    |-- static/
 |    |-- templates/
 |    |-- .env        <-- here!
 |    |-- .gitignore
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/
```

**.env**

```
SECRET_KEY=rqr_cjv4igscyu8&&(0ce(=sy=f2)p=f_wn&@0xsp7m$@!kp=d
```

The **.env** file is ignored in the **.gitignore** file, so every time we are going to deploy the application or run in a different machine, we will have to create a **.env** file and add the necessary configuration.

Now let’s install another library to help us write the database connection in a single line. This way it’s easier to write different database connection strings in different environments:

```
pip install dj-database-url
```

For now, all the configurations we will need to decouple:

**myproject/settings.py**

```
from decouple import config, Csv
import dj_database_url

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())
DATABASES = {
    'default': dj_database_url.config(
        default=config('DATABASE_URL')
    )
}
```

Example of a **.env** file for our local machine:

```
SECRET_KEY=rqr_cjv4igscyu8&&(0ce(=sy=f2)p=f_wn&@0xsp7m$@!kp=d
DEBUG=True
ALLOWED_HOSTS=.localhost,127.0.0.1
```

Notice that in the `DEBUG` configuration we have a default, so in production we can ignore this configuration because it will be set to `False` automatically, as it is supposed to be.

Now the `ALLOWED_HOSTS` will be transformed into a list like `['.localhost', '127.0.0.1'. ]`. Now, this is on our local machine, for production we will set it to something like `['.djangoboards.com', ]` or whatever domain you have.

This particular configuration makes sure your application is only served to this domain.

* * *