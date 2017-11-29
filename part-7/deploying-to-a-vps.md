<h4 id="deploying-to-a-vps-digital-ocean">Deploying to a VPS (Digital Ocean)</h4>

<p>You may use any other VPS (Virtual Private Server) you like. The configuration should be very similar, after all, we
are going to use Ubuntu 16.04 as our server.</p>

<p>First, let’s create a new server (on Digital Ocean they call it “Droplet”). Select Ubuntu 16.04:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do1.png" alt="Digital Ocean" /></p>

<p>Pick the size. The smallest droplet is enough:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do2.png" alt="Digital Ocean" /></p>

<p>Then choose a hostname for your droplet (in my case “django-boards”):</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do3.png" alt="Digital Ocean" /></p>

<p>If you have an SSH key, you can add it to your account. Then you will be able to log in the server using it. Otherwise,
they will email you the root password.</p>

<p>Now pick the server’s IP address:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/do4.png" alt="Digital Ocean" /></p>

<p>Before we log in to the server, let’s point our domain name to this IP address. This will save some time because
DNS settings usually take a few minutes to propagate.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap.png" alt="Namecheap" /></p>

<p>So here we added two A records, one pointing to the naked domain “djangoboards.com” and the other one for
“www.djangoboards.com”. We will use NGINX to configure a canonical URL.</p>

<p>Now let’s log in to the server using your terminal:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">ssh root@45.55.144.54
root@45.55.144.54's password:</code></pre></figure>

<p>Then you should see the following message:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">You are required to change your password immediately (root enforced)
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
(current) UNIX password:</code></pre></figure>

<p>Set the new password, and let’s start to configure the server.</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo apt-get update
sudo apt-get -y upgrade</code></pre></figure>

<p>If you get any prompt during the upgrade, select the option “keep the local version currently installed”.</p>

<p><strong>Python 3.6</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.6</code></pre></figure>

<p><strong>PostgreSQL</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo apt-get -y install postgresql postgresql-contrib</code></pre></figure>

<p><strong>NGINX</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo apt-get -y install nginx</code></pre></figure>

<p><strong>Supervisor</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo apt-get -y install supervisor

sudo systemctl enable supervisor
sudo systemctl start supervisor</code></pre></figure>

<p><strong>Virtualenv</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">wget https://bootstrap.pypa.io/get-pip.py
sudo python3.6 get-pip.py
sudo pip3.6 install virtualenv</code></pre></figure>

<h5 id="application-user">Application User</h5>

<p>Create a new user with the command below:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">adduser boards</code></pre></figure>

<p>Usually, I just pick the name of the application. Enter a password and optionally add some extra info to the prompt.</p>

<p>Now add the user to the sudoers list:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">gpasswd -a boards sudo</code></pre></figure>

<h5 id="postgresql-database-setup">PostgreSQL Database Setup</h5>

<p>First switch to the postgres user:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo su - postgres</code></pre></figure>

<p>Create a database user:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">createuser u_boards</code></pre></figure>

<p>Create a new database and set the user as the owner:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">createdb django_boards --owner u_boards</code></pre></figure>

<p>Define a strong password for the user:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">psql -c "ALTER USER u_boards WITH PASSWORD 'BcAZoYWsJbvE7RMgBPzxOCexPRVAq'"</code></pre></figure>

<p>We can now exit the postgres user:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">exit</code></pre></figure>

<h5 id="django-project-setup">Django Project Setup</h5>

<p>Switch to the application user:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo su - boards</code></pre></figure>

<p>First, we can check where we are:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">pwd
/home/boards</code></pre></figure>

<p>First, let’s clone the repository with our code:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git clone https://github.com/sibtc/django-beginners-guide.git</code></pre></figure>

<p>Start a virtual environment:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">virtualenv venv -p python3.6</code></pre></figure>

<p>Initialize the virtualenv:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">source venv/bin/activate</code></pre></figure>

<p>Install the requirements:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">pip install -r django-beginners-guide/requirements.txt</code></pre></figure>

<p>We will have to add two extra libraries here, the Gunicorn and the PostgreSQL driver:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">pip install gunicorn
pip install psycopg2</code></pre></figure>

<p>Now inside the <strong>/home/boards/django-beginners-guide</strong> folder, let’s create a <strong>.env</strong> file to store the database
credentials, the secret key and everything else:</p>

<p><strong>/home/boards/django-beginners-guide/.env</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">SECRET_KEY=rqr_cjv4igscyu8&amp;&amp;(0ce(=sy=f2)p=f_wn&amp;@0xsp7m$@!kp=d
ALLOWED_HOSTS=.djangoboards.com
DATABASE_URL=postgres://u_boards:BcAZoYWsJbvE7RMgBPzxOCexPRVAq@localhost:5432/django_boards</code></pre></figure>

<p>Here is the syntax of the database URL: postgres://<code class="highlighter-rouge">db_user</code>:<code class="highlighter-rouge">db_password</code>@<code class="highlighter-rouge">db_host</code>:<code class="highlighter-rouge">db_port</code>/<code class="highlighter-rouge">db_name</code>.</p>

<p>Now let’s migrate the database, collect the static files and create a super user:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">cd django-beginners-guide</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py migrate

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
  Applying sessions.0001_initial... OK</code></pre></figure>

<p>Now the static files:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py collectstatic

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
...</code></pre></figure>

<p>This command copy all the static assets to an external directory where NGINX can serve the files for us. More on that
later.</p>

<p>Now create a super user for the application:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py createsuperuser</code></pre></figure>

<h5 id="configuring-gunicorn">Configuring Gunicorn</h5>

<p>So, Gunicorn is the one responsible for executing the Django code behind a proxy server.</p>

<p>Create a new file named <strong>gunicorn_start</strong> inside <strong>/home/boards</strong>:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">#!/bin/bash

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
  --log-file=-</code></pre></figure>

<p>This script will start the application server. We are providing some information such as where the Django project is,
which application user to be used to run the server, and so on.</p>

<p>Now make this file executable:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">chmod u+x gunicorn_start</code></pre></figure>

<p>Create two empty folders, one for the socket file and one to store the logs:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">mkdir run logs</code></pre></figure>

<p>Right now the directory structure inside <strong>/home/boards</strong> should look like this:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">django-beginners-guide/
gunicorn_start
logs/
run/
staticfiles/
venv/</code></pre></figure>

<p>The <strong>staticfiles</strong> folder was created by the <strong>collectstatic</strong> command.</p>

<h5 id="configuring-supervisor">Configuring Supervisor</h5>

<p>First, create an empty log file inside the <strong>/home/boards/logs/</strong> folder:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">touch logs/gunicorn.log</code></pre></figure>

<p>Now create a new supervisor file:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo vim /etc/supervisor/conf.d/boards.conf</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">[program:boards]
command=/home/boards/gunicorn_start
user=boards
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/home/boards/logs/gunicorn.log</code></pre></figure>

<p>Save the file and run the commands below:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo supervisorctl reread
sudo supervisorctl update</code></pre></figure>

<p>Now check the status:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo supervisorctl status boards</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">boards                           RUNNING   pid 308, uptime 0:00:07</code></pre></figure>

<h5 id="configuring-nginx">Configuring NGINX</h5>

<p>Next step is to set up the NGINX server to serve the static files and to pass the requests to Gunicorn:</p>

<p>Add a new configuration file named <strong>boards</strong> inside <strong>/etc/nginx/sites-available/</strong>:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">upstream app_server {
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
}</code></pre></figure>

<p>Create a symbolic link to the <strong>sites-enabled</strong> folder:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo ln -s /etc/nginx/sites-available/boards /etc/nginx/sites-enabled/boards</code></pre></figure>

<p>Remove the default NGINX website:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo rm /etc/nginx/sites-enabled/default</code></pre></figure>

<p>Restart the NGINX service:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo service nginx restart</code></pre></figure>

<p>At this point, if the DNS have already propagated, the website should be available on the URL www.djangoboards.com.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/django-boards.png" alt="Django Boards" /></p>

<hr />
