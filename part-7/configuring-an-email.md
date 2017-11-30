#### Configuring an Email Service

One of the best options to get started is [Mailgun](https://www.mailgun.com/). It offers a very reliable free plan covering 12,000 emails per month.

Sign up for a free account. Then just follow the steps, it’s very straightforward. You will have to work together with the service you registered your domain. In my case, it was [Namecheap](https://namecheap.pxf.io/c/477033/386170/5618).

Click on add domain to add a new domain to your account. Follow the instructions and make sure you use “mg.” subdomain:

![Mailgun](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg1.png)

Now grab the first set of DNS records, it’s two TXT records:

![Mailgun](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg2.png)

Add it to your domain, using the web interface offered by your registrar:

![Namecheap](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap1.png)

Do the same thing with the MX records:

![Mailgun](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg3.png)

Add them to the domain:

![Namecheap](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap2.png)

Now this step is not mandatory, but since we are already here, confirm it as well:

![Mailgun](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg4.png)

![Namecheap](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap3.png)

After adding all the DNS records, click in the **Check DNS Records Now** button:

![Mailgun](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg5.png)

Now we need to have some patience. Sometimes it takes a while to validate the DNS.

Meanwhile, we can configure the application to receive the connection parameters.

**myproject/settings.py**

```

```
EMAIL_BACKEND = config('EMAIL_BACKEND', default='django.core.mail.backends.smtp.EmailBackend')
EMAIL_HOST = config('EMAIL_HOST', default='')
EMAIL_PORT = config('EMAIL_PORT', default=587, cast=int)
EMAIL_HOST_USER = config('EMAIL_HOST_USER', default='')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD', default='')
EMAIL_USE_TLS = config('EMAIL_USE_TLS', default=True, cast=bool)

DEFAULT_FROM_EMAIL = 'Django Boards <noreply@djangoboards.com>'
EMAIL_SUBJECT_PREFIX = '[Django Boards] '
```

```

Then, my local machine **.env** file would look like this:

```

```
SECRET_KEY=rqr_cjv4igscyu8&&(0ce(=sy=f2)p=f_wn&@0xsp7m$@!kp=d
DEBUG=True
ALLOWED_HOSTS=.localhost,127.0.0.1
DATABASE_URL=sqlite:///db.sqlite3
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
```

```

And my production **.env** file would look like this:

```

```
SECRET_KEY=rqr_cjv4igscyu8&&(0ce(=sy=f2)p=f_wn&@0xsp7m$@!kp=d
ALLOWED_HOSTS=.djangoboards.com
DATABASE_URL=postgres://u_boards:BcAZoYWsJbvE7RMgBPzxOCexPRVAq@localhost:5432/django_boards
EMAIL_HOST=smtp.mailgun.org
EMAIL_HOST_USER=postmaster@mg.djangoboards.com
EMAIL_HOST_PASSWORD=ED2vmrnGTM1Rdwlhazyhxxcd0F
```

```

You can find your credentials in the **Domain Information** section on Mailgun.

*   EMAIL_HOST: SMTP Hostname
*   EMAIL_HOST_USER: Default SMTP Login
*   EMAIL_HOST_PASSWORD: Default Password

We can test the new settings in the production server. Make the changes in the **settings.py** file on your local machine, commit the changes to the remote repository. Then, in the server pull the new code and restart the Gunicorn process:

```

```
git pull
```

```

Edit the **.env** file with the email credentials.

Then restart the Gunicorn process:

```

```
sudo supervisorctl restart boards
```

```

Now we can try to start the password reset process:

![Password Reset Email](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/pwd.png)

On the Mailgun dashboard you can have some statistics about the email delivery:

![Mailgun](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg6.png)

* * *