#### Deployment Strategy

Here is an overview of the deployment strategy we are going to use in this tutorial:

![Deployment](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/deployment.jpg)

The cloud is our Virtual Private Server provided by [Digital Ocean](https://m.do.co/c/074832454ff1). You can sign up to Digital Ocean using my affiliate link to get a [free $10 credit](https://m.do.co/c/074832454ff1) (only valid for new accounts).

Upfront we will have NGINX, illustrated by the ogre. NGINX will receive all requests to the server. But it won’t try to do anything smart if the request data. All it is going to do is decide if the requested information is a static asset that it can serve by itself, or if it’s something more complicated. If so, it will pass the request to Gunicorn.

The NGINX will also be configured with HTTPS certificates. Meaning it will only accept requests via HTTPS. If the client tries to request via HTTP, NGINX will first redirect the user to the HTTPS, and only then it will decide what to do with the request.

We are also going to install this certbot to automatically renew the Let’s Encrypt certificates.

Gunicorn is an application server. Depending on the number of processors the server has, it can spawn multiple workers to process multiple requests in parallel. It manages the workload and executes the Python and Django code.

Django is the one doing the hard work. It may access the database (PostgreSQL) or the file system. But for the most part, the work is done inside the views, rendering templates, all those things that we’ve been coding for the past weeks. After Django process the request, it returns a response to Gunicorn, who returns the result to NGINX that will finally deliver the response to the client.

We are also going to install PostgreSQL, a production quality database system. Because of Django’s ORM system, it’s easy to switch databases.

The last step is to install Supervisor. It’s a process control system and it will keep an eye on Gunicorn and Django to make sure everything runs smoothly. If the server restarts, or if Gunicorn crashes, it will automatically restart it.

* * *