<h4 id="migrations">Migrations</h4>

<p>Migration is a fundamental part of Web development with Django. It’s how we evolve our application’s models keeping
the models’ files synchronized with the database.</p>

<p>When we first run the command <code class="highlighter-rouge">python manage.py migrate</code> Django grab all migration files and generate the database
schema.</p>

<p>When Django applies a migration, it has a special table called <strong>django_migrations</strong>. In this table, Django registers
all the applied migrations.</p>

<p>So if we try to run the command again:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py migrate</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Operations to perform:
  Apply all migrations: admin, auth, boards, contenttypes, sessions
Running migrations:
  No migrations to apply.</code></pre></figure>

<p>Django will know there’s nothing to do.</p>

<p>Let’s create a migration by adding a new field to the Topic model:</p>

<p><strong>boards/models.py</strong>
<small><a href="https://gist.github.com/vitorfs/816f47aa4df8e7b157df75e0ff209aac#file-models-py-L25" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">Topic</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">subject</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">255</span><span class="p">)</span>
    <span class="n">last_updated</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">DateTimeField</span><span class="p">(</span><span class="n">auto_now_add</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'topics'</span><span class="p">)</span>
    <span class="n">starter</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">User</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'topics'</span><span class="p">)</span>
    <span class="n">views</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">PositiveIntegerField</span><span class="p">(</span><span class="n">default</span><span class="o">=</span><span class="mi">0</span><span class="p">)</span>  <span class="c"># &lt;- here</span>

    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">subject</span></code></pre></figure>

<p>Here we added a <code class="highlighter-rouge">PositiveIntegerField</code>. Since this field is going to store the number of page views, a negative page
view wouldn’t make sense.</p>

<p>Before we can use our new field, we have to update the database schema. Execute the <code class="highlighter-rouge">makemigrations</code> command:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py makemigrations

Migrations for 'boards':
  boards/migrations/0003_topic_views.py
    - Add field views to topic</code></pre></figure>

<p>The <code class="highlighter-rouge">makemigrations</code> command automatically generated the <strong>0003_topic_views.py</strong> file, which will be used to modify the
database, adding the <strong>views</strong> field.</p>

<p>Now apply the migration by running the command <code class="highlighter-rouge">migrate</code>:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, boards, contenttypes, sessions
Running migrations:
  Applying boards.0003_topic_views... OK</code></pre></figure>

<p>Now we can use it to keep track of the number of views a given topic is receiving:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/c0c97c1e050204d9152c59b4da2f9305#file-views-py-L41" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">get_object_or_404</span><span class="p">,</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Topic</span>

<span class="k">def</span> <span class="nf">topic_posts</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="p">):</span>
    <span class="n">topic</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">board__pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">topic_pk</span><span class="p">)</span>
    <span class="n">topic</span><span class="o">.</span><span class="n">views</span> <span class="o">+=</span> <span class="mi">1</span>
    <span class="n">topic</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'topic_posts.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'topic'</span><span class="p">:</span> <span class="n">topic</span><span class="p">})</span></code></pre></figure>

<p><strong>templates/topics.html</strong>
<small><a href="https://gist.github.com/vitorfs/70ebb1a06e1044387943ee83bafcd526" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">for</span> <span class="nv">topic</span> <span class="ow">in</span> <span class="nv">topics</span> <span class="cp">%}</span>
  <span class="nt">&lt;tr&gt;</span>
    <span class="nt">&lt;td&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">board.pk</span> <span class="nv">topic.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.starter.username</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.replies</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.views</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>  <span class="c">&lt;!-- here --&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.last_updated</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
  <span class="nt">&lt;/tr&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<p>Now open a topic and refresh the page a few times, and see if it’s counting the page views:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-5.png" alt="Posts" /></p>

<hr />

