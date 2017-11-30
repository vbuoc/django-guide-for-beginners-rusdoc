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

```
ssh root@45.55.144.54
root@45.55.144.54's password:
```

Then you should see the following message:

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

Set the new password, and let’s start to configure the server.

```
sudo apt-get update
sudo apt-get -y upgrade
```

If you get any prompt during the upgrade, select the option “keep the local version currently installed”.

**Python 3.6**

```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.6
```

**PostgreSQL**

```
sudo apt-get -y install postgresql postgresql-contrib
```

**NGINX**

```
sudo apt-get -y install nginx
```

**Supervisor**

```
sudo apt-get -y install supervisor

sudo systemctl enable supervisor
sudo systemctl start supervisor
```

**Virtualenv**

```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3.6 get-pip.py
sudo pip3.6 install virtualenv
```

##### Application User

Create a new user with the command below:

```
adduser boards
```

Usually, I just pick the name of the application. Enter a password and optionally add some extra info to the prompt.

Now add the user to the sudoers list:

```
gpasswd -a boards sudo
```

##### PostgreSQL Database Setup

First switch to the postgres user:

```
sudo su - postgres
```

Create a database user:

```
createuser u_boards
```

Create a new database and set the user as the owner:

```
createdb django_boards --owner u_boards
```

Define a strong password for the user:

```
psql -c "ALTER USER u_boards WITH PASSWORD 'BcAZoYWsJbvE7RMgBPzxOCexPRVAq'"
```

We can now exit the postgres user:

```
exit
```

##### Django Project Setup

Switch to the application user:

```
sudo su - boards
```

First, we can check where we are:

```
pwd
/home/boards
```

First, let’s clone the repository with our code:

```
git clone https://github.com/sibtc/django-beginners-guide.git
```

Start a virtual environment:

```
virtualenv venv -p python3.6
```

Initialize the virtualenv:

```
source venv/bin/activate
```

Install the requirements:

```
pip install -r django-beginners-guide/requirements.txt
```

We will have to add two extra libraries here, the Gunicorn and the PostgreSQL driver:

```
pip install gunicorn
pip install psycopg2
```

Now inside the **/home/boards/django-beginners-guide** folder, let’s create a **.env** file to store the database credentials, the secret key and everything else:

**/home/boards/django-beginners-guide/.env**

```
SECRET_KEY=rqr_cjv4igscyu8&&(0ce(=sy=f2)p=f_wn&@0xsp7m$@!kp=d
ALLOWED_HOSTS=.djangoboards.com
DATABASE_URL=postgres://u_boards:BcAZoYWsJbvE7RMgBPzxOCexPRVAq@localhost:5432/django_boards
```

Here is the syntax of the database URL: postgres://`db_user`:`db_password`@`db_host`:`db_port`/`db_name`.

Now let’s migrate the database, collect the static files and create a super user:

```
cd django-beginners-guide
```

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

Now the static files:

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

This command copy all the static assets to an external directory where NGINX can serve the files for us. More on that later.

Now create a super user for the application:

```
python manage.py createsuperuser
```

##### Configuring Gunicorn

So, Gunicorn is the one responsible for executing the Django code behind a proxy server.

Create a new file named **gunicorn_start** inside **/home/boards**:

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

This script will start the application server. We are providing some information such as where the Django project is, which application user to be used to run the server, and so on.

Now make this file executable:

```
chmod u+x gunicorn_start
```

Create two empty folders, one for the socket file and one to store the logs:

```
mkdir run logs
```

Right now the directory structure inside **/home/boards** should look like this:

```
django-beginners-guide/
gunicorn_start
logs/
run/
staticfiles/
venv/
```

The **staticfiles** folder was created by the **collectstatic** command.

##### Configuring Supervisor

First, create an empty log file inside the **/home/boards/logs/** folder:

```
touch logs/gunicorn.log
```

Now create a new supervisor file:

```
sudo vim /etc/supervisor/conf.d/boards.conf
```

```
[program:boards]
command=/home/boards/gunicorn_start
user=boards
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/home/boards/logs/gunicorn.log
```

Save the file and run the commands below:

```
sudo supervisorctl reread
sudo supervisorctl update
```

Now check the status:

```
sudo supervisorctl status boards
```

```
boards                           RUNNING   pid 308, uptime 0:00:07
```

##### Configuring NGINX

Next step is to set up the NGINX server to serve the static files and to pass the requests to Gunicorn:

Add a new configuration file named **boards** inside **/etc/nginx/sites-available/**:

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

Create a symbolic link to the **sites-enabled** folder:

```
sudo ln -s /etc/nginx/sites-available/boards /etc/nginx/sites-enabled/boards
```

Remove the default NGINX website:

```
sudo rm /etc/nginx/sites-enabled/default
```

Restart the NGINX service:

```
sudo service nginx restart
```

At this point, if the DNS have already propagated, the website should be available on the URL www.djangoboards.com.

![Django Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/django-boards.png)

* * *