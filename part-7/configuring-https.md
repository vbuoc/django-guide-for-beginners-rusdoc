<h4 id="configuring-https-certificate">Configuring HTTPS Certificate</h4>

<p>Now let’s protect our application with a nice HTTPS certificate provided by <a href="https://letsencrypt.org/" target="_blank" rel="noopener">Let’s Encrypt</a>.</p>

<p>Setting up HTTPS has never been this easy. And better, we can get it for free nowadays. They provide a solution called
<strong>certbot</strong> which takes care of installing and renewing the certificates for us. It’s very straightforward:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx</code></pre></figure>

<p>Now install the certs:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo certbot --nginx</code></pre></figure>

<p>Just follow the prompts. When asked about:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.</code></pre></figure>

<p>Choose <code class="highlighter-rouge">2</code> to redirect all HTTP traffic to HTTPS.</p>

<p>With that the site is already being served over HTTPS:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/https.png" alt="HTTPS" /></p>

<p>Setup the auto renew of the certs. Run the command below to edit the crontab file:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">sudo crontab -e</code></pre></figure>

<p>Add the following line to the end of the file:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">0 4 * * * /usr/bin/certbot renew --quiet</code></pre></figure>

<p>This command will run every day at 4 am. All certificates expiring within 30 days will automatically be renewed.</p>

<hr />
