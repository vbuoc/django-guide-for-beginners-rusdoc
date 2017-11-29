<h4 id="deployment-strategy">Deployment Strategy</h4>

<p>Here is an overview of the deployment strategy we are going to use in this tutorial:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/deployment.jpg" alt="Deployment" /></p>

<p>The cloud is our Virtual Private Server provided by <a href="https://m.do.co/c/074832454ff1" target="_blank" rel="nofollow noopener">Digital Ocean</a>.
You can sign up to Digital Ocean using my affiliate link to get a <a href="https://m.do.co/c/074832454ff1" target="_blank" rel="nofollow noopener">free $10 credit</a> (only valid for new accounts).</p>

<p>Upfront we will have NGINX, illustrated by the ogre. NGINX will receive all requests to the server. But it won’t try
to do anything smart if the request data. All it is going to do is decide if the requested information is a static
asset that it can serve by itself, or if it’s something more complicated. If so, it will pass the request to Gunicorn.</p>

<p>The NGINX will also be configured with HTTPS certificates. Meaning it will only accept requests via HTTPS. If the client
tries to request via HTTP, NGINX will first redirect the user to the HTTPS, and only then it will decide what to do
with the request.</p>

<p>We are also going to install this certbot to automatically renew the Let’s Encrypt certificates.</p>

<p>Gunicorn is an application server. Depending on the number of processors the server has, it can spawn multiple workers
to process multiple requests in parallel. It manages the workload and executes the Python and Django code.</p>

<p>Django is the one doing the hard work. It may access the database (PostgreSQL) or the file system. But for the
most part, the work is done inside the views, rendering templates, all those things that we’ve been coding for the
past weeks. After Django process the request, it returns a response to Gunicorn, who returns the result to NGINX
that will finally deliver the response to the client.</p>

<p>We are also going to install PostgreSQL, a production quality database system. Because of Django’s ORM system, it’s
easy to switch databases.</p>

<p>The last step is to install Supervisor. It’s a process control system and it will keep an eye on Gunicorn and
Django to make sure everything runs smoothly. If the server restarts, or if Gunicorn crashes, it will automatically
restart it.</p>

<hr />
