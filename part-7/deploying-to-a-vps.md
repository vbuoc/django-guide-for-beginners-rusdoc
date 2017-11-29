#### Deploying to a VPS (Digital Ocean)

You may use any other VPS (Virtual Private Server) you like. The configuration should be very similar, after all, we are going to use Ubuntu 16.04 as our server.

First, let’s create a new server (on Digital Ocean they call it “Droplet”). Select Ubuntu 16.04:

![Digital Ocean](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do1.png)

Pick the size. The smallest droplet is enough:

![Digital Ocean](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do2.png)

Then choose a hostname for your droplet (in my case “django-boards”):

![Digital Ocean](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do3.png)

If you have an SSH key, you can add it to your account. Then you will be able to log in the server using it. Otherwise, they will email you the root password.

Now pick the server’s IP address:

![Digital Ocean](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do4.png)

Before we log in to the server, let’s point our domain name to this IP address. This will save some time because DNS settings usually take a few minutes to propagate.

![Namecheap](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap.png)

So here we added two A records, one pointing to the naked domain “djangoboards.com” and the other one for “www.djangoboards.com”. We will use NGINX to configure a canonical URL.

Now let’s log in to the server using your terminal:

<figure class="highlight">

```
ssh root@45.55.144.54
root@45.55.144.54's password:
```

</figure>

Then you should see the following message:

<figure class="highlight">

```
You are required to change your password immediately (root enforced)
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-93-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

Last login: Sun Oct 15 18:39:21 2017 from 82.128.188.51
Changing password for root.
(current) UNIX password:
```

</figure>

Set the new password, and let’s start to configure the server.

<figure class="highlight">

```
sudo apt-get update
sudo apt-get -y upgrade
```

</figure>

If you get any prompt during the upgrade, select the option “keep the local version currently installed”.

**Python 3.6**

<figure class="highlight">

```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.6
```

</figure>

**PostgreSQL**

<figure class="highlight">

```
sudo apt-get -y install postgresql postgresql-contrib
```

</figure>

**NGINX**

<figure class="highlight">

```
sudo apt-get -y install nginx
```

</figure>

**Supervisor**

<figure class="highlight">

```
sudo apt-get -y install supervisor

sudo systemctl enable supervisor
sudo systemctl start supervisor
```

</figure>

**Virtualenv**

<figure class="highlight">

```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3.6 get-pip.py
sudo pip3.6 install virtualenv
```

</figure>

##### Application User

Create a new user with the command below:

<figure class="highlight">

```
adduser boards
```

</figure>

Usually, I just pick the name of the application. Enter a password and optionally add some extra info to the prompt.

Now add the user to the sudoers list:

<figure class="highlight">

```
gpasswd -a boards sudo
```

</figure>

##### PostgreSQL Database Setup

First switch to the postgres user:

<figure class="highlight">

```
sudo su - postgres
```

</figure>

Create a database user:

<figure class="highlight">

```
createuser u_boards
```

</figure>

Create a new database and set the user as the owner:

<figure class="highlight">

```
createdb django_boards --owner u_boards
```

</figure>

Define a strong password for the user:

<figure class="highlight">

```
psql -c "ALTER USER u_boards WITH PASSWORD 'BcAZoYWsJbvE7RMgBPzxOCexPRVAq'"
```

</figure>

We can now exit the postgres user:

<figure class="highlight">

```
exit
```

</figure>

##### Django Project Setup

Switch to the application user:

<figure class="highlight">

```
sudo su - boards
```

</figure>

First, we can check where we are:

<figure class="highlight">

```
pwd
/home/boards
```

</figure>

First, let’s clone the repository with our code:

<figure class="highlight">

```
git clone https://github.com/sibtc/django-beginners-guide.git
```

</figure>

Start a virtual environment:

<figure class="highlight">

```
virtualenv venv -p python3.6
```

</figure>

Initialize the virtualenv:

<figure class="highlight">

```
source venv/bin/activate
```

</figure>

Install the requirements:

<figure class="highlight">

```
pip install -r django-beginners-guide/requirements.txt
```

</figure>

We will have to add two extra libraries here, the Gunicorn and the PostgreSQL driver:

<figure class="highlight">

```
pip install gunicorn
pip install psycopg2
```

</figure>

Now inside the **/home/boards/django-beginners-guide** folder, let’s create a **.env** file to store the database credentials, the secret key and everything else:

**/home/boards/django-beginners-guide/.env**

<figure class="highlight">

```
SECRET_KEY=rqr_cjv4igscyu8&&(0ce(=sy=f2)p=f_wn&@0xsp7m$@!kp=d
ALLOWED_HOSTS=.djangoboards.com
DATABASE_URL=postgres://u_boards:BcAZoYWsJbvE7RMgBPzxOCexPRVAq@localhost:5432/django_boards
```

</figure>

Here is the syntax of the database URL: postgres://`db_user`:`db_password`@`db_host`:`db_port`/`db_name`.

Now let’s migrate the database, collect the static files and create a super user:

<figure class="highlight">

```
cd django-beginners-guide
```

</figure>

<figure class="highlight">

```
python manage.py migrate

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
  Applying boards.0002_auto_20170917_1618... OK
  Applying boards.0003_topic_views... OK
  Applying sessions.0001_initial... OK
```

</figure>

Now the static files:

<figure class="highlight">

```
python manage.py collectstatic

Copying '/home/boards/django-beginners-guide/static/js/jquery-3.2.1.min.js'
Copying '/home/boards/django-beginners-guide/static/js/popper.min.js'
Copying '/home/boards/django-beginners-guide/static/js/bootstrap.min.js'
Copying '/home/boards/django-beginners-guide/static/js/simplemde.min.js'
Copying '/home/boards/django-beginners-guide/static/css/app.css'
Copying '/home/boards/django-beginners-guide/static/css/bootstrap.min.css'
Copying '/home/boards/django-beginners-guide/static/css/accounts.css'
Copying '/home/boards/django-beginners-guide/static/css/simplemde.min.css'
Copying '/home/boards/django-beginners-guide/static/img/avatar.svg'
Copying '/home/boards/django-beginners-guide/static/img/shattered.png'
...
```

</figure>

This command copy all the static assets to an external directory where NGINX can serve the files for us. More on that later.

Now create a super user for the application:

<figure class="highlight">

```
python manage.py createsuperuser
```

</figure>

##### Configuring Gunicorn

So, Gunicorn is the one responsible for executing the Django code behind a proxy server.

Create a new file named **gunicorn_start** inside **/home/boards**:

<figure class="highlight">

```
#!/bin/bash

NAME="django_boards"
DIR=/home/boards/django-beginners-guide
USER=boards
GROUP=boards
WORKERS=3
BIND=unix:/home/boards/run/gunicorn.sock
DJANGO_SETTINGS_MODULE=myproject.settings
DJANGO_WSGI_MODULE=myproject.wsgi
LOG_LEVEL=error

cd $DIR
source ../venv/bin/activate

export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DIR:$PYTHONPATH

exec ../venv/bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $WORKERS \
  --user=$USER \
  --group=$GROUP \
  --bind=$BIND \
  --log-level=$LOG_LEVEL \
  --log-file=-
```

</figure>

This script will start the application server. We are providing some information such as where the Django project is, which application user to be used to run the server, and so on.

Now make this file executable:

<figure class="highlight">

```
chmod u+x gunicorn_start
```

</figure>

Create two empty folders, one for the socket file and one to store the logs:

<figure class="highlight">

```
mkdir run logs
```

</figure>

Right now the directory structure inside **/home/boards** should look like this:

<figure class="highlight">

```
django-beginners-guide/
gunicorn_start
logs/
run/
staticfiles/
venv/
```

</figure>

The **staticfiles** folder was created by the **collectstatic** command.

##### Configuring Supervisor

First, create an empty log file inside the **/home/boards/logs/** folder:

<figure class="highlight">

```
touch logs/gunicorn.log
```

</figure>

Now create a new supervisor file:

<figure class="highlight">

```
sudo vim /etc/supervisor/conf.d/boards.conf
```

</figure>

<figure class="highlight">

```
[program:boards]
command=/home/boards/gunicorn_start
user=boards
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/home/boards/logs/gunicorn.log
```

</figure>

Save the file and run the commands below:

<figure class="highlight">

```
sudo supervisorctl reread
sudo supervisorctl update
```

</figure>

Now check the status:

<figure class="highlight">

```
sudo supervisorctl status boards
```

</figure>

<figure class="highlight">

```
boards                           RUNNING   pid 308, uptime 0:00:07
```

</figure>

##### Configuring NGINX

Next step is to set up the NGINX server to serve the static files and to pass the requests to Gunicorn:

Add a new configuration file named **boards** inside **/etc/nginx/sites-available/**:

<figure class="highlight">

```
upstream app_server {
    server unix:/home/boards/run/gunicorn.sock fail_timeout=0;
}

server {
    listen 80;
    server_name www.djangoboards.com;  # here can also be the IP address of the server

    keepalive_timeout 5;
    client_max_body_size 4G;

    access_log /home/boards/logs/nginx-access.log;
    error_log /home/boards/logs/nginx-error.log;

    location /static/ {
        alias /home/boards/staticfiles/;
    }

    # checks for static file, if not found proxy to app
    location / {
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;
      proxy_pass http://app_server;
    }
}
```

</figure>

Create a symbolic link to the **sites-enabled** folder:

<figure class="highlight">

```
sudo ln -s /etc/nginx/sites-available/boards /etc/nginx/sites-enabled/boards
```

</figure>

Remove the default NGINX website:

<figure class="highlight">

```
sudo rm /etc/nginx/sites-enabled/default
```

</figure>

Restart the NGINX service:

<figure class="highlight">

```
sudo service nginx restart
```

</figure>

At this point, if the DNS have already propagated, the website should be available on the URL www.djangoboards.com.

![Django Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/django-boards.png)

* * *