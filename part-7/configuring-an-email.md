<h4 id="configuring-an-email-service">Configuring an Email Service</h4>

<p>One of the best options to get started is <a href="https://www.mailgun.com/" target="_blank" rel="noopener">Mailgun</a>. It
offers a very reliable free plan covering 12,000 emails per month.</p>

<p>Sign up for a free account. Then just follow the steps, it’s very straightforward. You will have to work
together with the service you registered your domain. In my case, it was <a href="https://namecheap.pxf.io/c/477033/386170/5618" target="_blank" rel="noopener nofollow">Namecheap</a>.</p>

<p>Click on add domain to add a new domain to your account. Follow the instructions and make sure you use “mg.”
subdomain:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg1.png" alt="Mailgun" /></p>

<p>Now grab the first set of DNS records, it’s two TXT records:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg2.png" alt="Mailgun" /></p>

<p>Add it to your domain, using the web interface offered by your registrar:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap1.png" alt="Namecheap" /></p>

<p>Do the same thing with the MX records:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg3.png" alt="Mailgun" /></p>

<p>Add them to the domain:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap2.png" alt="Namecheap" /></p>

<p>Now this step is not mandatory, but since we are already here, confirm it as well:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg4.png" alt="Mailgun" /></p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/namecheap3.png" alt="Namecheap" /></p>

<p>After adding all the DNS records, click in the <strong>Check DNS Records Now</strong> button:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg5.png" alt="Mailgun" /></p>

<p>Now we need to have some patience. Sometimes it takes a while to validate the DNS.</p>

<p>Meanwhile, we can configure the application to receive the connection parameters.</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">EMAIL_BACKEND</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'EMAIL_BACKEND'</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="s">'django.core.mail.backends.smtp.EmailBackend'</span><span class="p">)</span>
<span class="n">EMAIL_HOST</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'EMAIL_HOST'</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="s">''</span><span class="p">)</span>
<span class="n">EMAIL_PORT</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'EMAIL_PORT'</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="mi">587</span><span class="p">,</span> <span class="n">cast</span><span class="o">=</span><span class="nb">int</span><span class="p">)</span>
<span class="n">EMAIL_HOST_USER</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'EMAIL_HOST_USER'</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="s">''</span><span class="p">)</span>
<span class="n">EMAIL_HOST_PASSWORD</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'EMAIL_HOST_PASSWORD'</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="s">''</span><span class="p">)</span>
<span class="n">EMAIL_USE_TLS</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'EMAIL_USE_TLS'</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">cast</span><span class="o">=</span><span class="nb">bool</span><span class="p">)</span>

<span class="n">DEFAULT_FROM_EMAIL</span> <span class="o">=</span> <span class="s">'Django Boards &lt;noreply@djangoboards.com&gt;'</span>
<span class="n">EMAIL_SUBJECT_PREFIX</span> <span class="o">=</span> <span class="s">'[Django Boards] '</span></code></pre></figure>

<p>Then, my local machine <strong>.env</strong> file would look like this:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">SECRET_KEY=rqr_cjv4igscyu8&amp;&amp;(0ce(=sy=f2)p=f_wn&amp;@0xsp7m$@!kp=d
DEBUG=True
ALLOWED_HOSTS=.localhost,127.0.0.1
DATABASE_URL=sqlite:///db.sqlite3
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend</code></pre></figure>

<p>And my production <strong>.env</strong> file would look like this:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">SECRET_KEY=rqr_cjv4igscyu8&amp;&amp;(0ce(=sy=f2)p=f_wn&amp;@0xsp7m$@!kp=d
ALLOWED_HOSTS=.djangoboards.com
DATABASE_URL=postgres://u_boards:BcAZoYWsJbvE7RMgBPzxOCexPRVAq@localhost:5432/django_boards
EMAIL_HOST=smtp.mailgun.org
EMAIL_HOST_USER=postmaster@mg.djangoboards.com
EMAIL_HOST_PASSWORD=ED2vmrnGTM1Rdwlhazyhxxcd0F</code></pre></figure>

<p>You can find your credentials in the <strong>Domain Information</strong> section on Mailgun.</p>

<ul>
  <li>EMAIL_HOST: SMTP Hostname</li>
  <li>EMAIL_HOST_USER: Default SMTP Login</li>
  <li>EMAIL_HOST_PASSWORD: Default Password</li>
</ul>

<p>We can test the new settings in the production server. Make the changes in the <strong>settings.py</strong> file on your local
machine, commit the changes to the remote repository. Then, in the server pull the new code and restart the Gunicorn
process:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git pull</code></pre></figure>

<p>Edit the <strong>.env</strong> file with the email credentials.</p>

<p>Then restart the Gunicorn process:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo supervisorctl restart boards</code></pre></figure>

<p>Now we can try to start the password reset process:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/pwd.png" alt="Password Reset Email" /></p>

<p>On the Mailgun dashboard you can have some statistics about the email delivery:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/mg6.png" alt="Mailgun" /></p>

<hr />
