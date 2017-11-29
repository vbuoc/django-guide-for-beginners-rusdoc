<h4 id="project-settings">Project Settings</h4>

<p>No matter if the code is stored in a public or private remote repository, sensitive information should <strong>never</strong> be
committed and pushed to the remote repository. That includes secret keys, passwords, API keys, etc.</p>

<p>At this point, we have to deal with two specific types of configuration in our <strong>settings.py</strong> module:</p>

<ul>
  <li>Sensitive information such as keys and passwords;</li>
  <li>Configurations that are specific to a given environment.</li>
</ul>

<p>Passwords and keys can be stored in environment variables or using local files (not committed to the remote
repository):</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="c"># environment variables</span>
<span class="kn">import</span> <span class="nn">os</span>
<span class="n">SECRET_KEY</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="p">[</span><span class="s">'SECRET_KEY'</span><span class="p">]</span>

<span class="c"># or local files</span>
<span class="k">with</span> <span class="nb">open</span><span class="p">(</span><span class="s">'/etc/secret_key.txt'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="n">SECRET_KEY</span> <span class="o">=</span> <span class="n">f</span><span class="o">.</span><span class="n">read</span><span class="p">()</span><span class="o">.</span><span class="n">strip</span><span class="p">()</span></code></pre></figure>

<p>For that, there’s a great utility library called <a href="/2015/11/26/package-of-the-week-python-decouple.html">Python Decouple</a>
that I use in every single Django project I develop. It will search for a local file named <strong>.env</strong> to set the
configuration variables and will fall back to the environment variables. It also provides an interface to define
default values, transform the data into <strong>int</strong>, <strong>bool</strong>, and <strong>list</strong> when applicable.</p>

<p>It’s not mandatory, but I really find it a very useful tool. And it works like a charm with services like Heroku.</p>

<p>First, let’s install it:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">pip install python-decouple</code></pre></figure>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">decouple</span> <span class="kn">import</span> <span class="n">config</span>

<span class="n">SECRET_KEY</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'SECRET_KEY'</span><span class="p">)</span></code></pre></figure>

<p>Now we can place the sensitive information in a special file named <strong>.env</strong> (notice the dot in front) in the same
directory where the <strong>manage.py</strong> file is:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |-- myproject/
 |    |-- static/
 |    |-- templates/
 |    |-- .env        &lt;-- here!
 |    |-- .gitignore
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p><strong>.env</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">SECRET_KEY=rqr_cjv4igscyu8&amp;&amp;(0ce(=sy=f2)p=f_wn&amp;@0xsp7m$@!kp=d</code></pre></figure>

<p>The <strong>.env</strong> file is ignored in the <strong>.gitignore</strong> file, so every time we are going to deploy the application or run
in a different machine, we will have to create a <strong>.env</strong> file and add the necessary configuration.</p>

<p>Now let’s install another library to help us write the database connection in a single line. This way it’s easier to
write different database connection strings in different environments:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">pip install dj-database-url</code></pre></figure>

<p>For now, all the configurations we will need to decouple:</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">decouple</span> <span class="kn">import</span> <span class="n">config</span><span class="p">,</span> <span class="n">Csv</span>
<span class="kn">import</span> <span class="nn">dj_database_url</span>

<span class="n">SECRET_KEY</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'SECRET_KEY'</span><span class="p">)</span>
<span class="n">DEBUG</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'DEBUG'</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="bp">False</span><span class="p">,</span> <span class="n">cast</span><span class="o">=</span><span class="nb">bool</span><span class="p">)</span>
<span class="n">ALLOWED_HOSTS</span> <span class="o">=</span> <span class="n">config</span><span class="p">(</span><span class="s">'ALLOWED_HOSTS'</span><span class="p">,</span> <span class="n">cast</span><span class="o">=</span><span class="n">Csv</span><span class="p">())</span>
<span class="n">DATABASES</span> <span class="o">=</span> <span class="p">{</span>
    <span class="s">'default'</span><span class="p">:</span> <span class="n">dj_database_url</span><span class="o">.</span><span class="n">config</span><span class="p">(</span>
        <span class="n">default</span><span class="o">=</span><span class="n">config</span><span class="p">(</span><span class="s">'DATABASE_URL'</span><span class="p">)</span>
    <span class="p">)</span>
<span class="p">}</span></code></pre></figure>

<p>Example of a <strong>.env</strong> file for our local machine:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">SECRET_KEY=rqr_cjv4igscyu8&amp;&amp;(0ce(=sy=f2)p=f_wn&amp;@0xsp7m$@!kp=d
DEBUG=True
ALLOWED_HOSTS=.localhost,127.0.0.1</code></pre></figure>

<p>Notice that in the <code class="highlighter-rouge">DEBUG</code> configuration we have a default, so in production we can ignore this configuration because
it will be set to <code class="highlighter-rouge">False</code> automatically, as it is supposed to be.</p>

<p>Now the <code class="highlighter-rouge">ALLOWED_HOSTS</code> will be transformed into a list like <code class="highlighter-rouge">['.localhost', '127.0.0.1'. ]</code>. Now, this is on our
local machine, for production we will set it to something like <code class="highlighter-rouge">['.djangoboards.com', ]</code> or whatever domain you have.</p>

<p>This particular configuration makes sure your application is only served to this domain.</p>

<hr />
