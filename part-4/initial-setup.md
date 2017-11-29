<h4 id="initial-setup">Initial Setup</h4>

<p>To manage all this information, we can break it down in a different app. In the project root, in the same page where
the <strong>manage.py</strong> script is, run the following command to start a new app:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">django-admin startapp accounts</code></pre></figure>

<p>The project structure should like this right now:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/     &lt;-- our new django app!
 |    |-- boards/
 |    |-- myproject/
 |    |-- static/
 |    |-- templates/
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>Next step, include the <strong>accounts</strong> app to the <code class="highlighter-rouge">INSTALLED_APPS</code> in the <strong>settings.py</strong> file:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">INSTALLED_APPS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="s">'django.contrib.admin'</span><span class="p">,</span>
    <span class="s">'django.contrib.auth'</span><span class="p">,</span>
    <span class="s">'django.contrib.contenttypes'</span><span class="p">,</span>
    <span class="s">'django.contrib.sessions'</span><span class="p">,</span>
    <span class="s">'django.contrib.messages'</span><span class="p">,</span>
    <span class="s">'django.contrib.staticfiles'</span><span class="p">,</span>

    <span class="s">'widget_tweaks'</span><span class="p">,</span>

    <span class="s">'accounts'</span><span class="p">,</span>
    <span class="s">'boards'</span><span class="p">,</span>
<span class="p">]</span></code></pre></figure>

<p>From now on, we will be working on the <strong>accounts</strong> app.</p>

<hr />
