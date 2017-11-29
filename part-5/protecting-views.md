<h4 id="protecting-views">Protecting Views</h4>

<p>We have to start protecting our views against non-authorized users. So far we have the following view to start new
posts:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-1.png" alt="Topics view not logged in" /></p>

<p>In the picture above the user is not logged in, and even though they can see the page and the form.</p>

<p>Django has a built-in <em>view decorator</em> to avoid that issue:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/4d3334a0daa9e7a872653a22ff39320a#file-models-py-L19" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.decorators</span> <span class="kn">import</span> <span class="n">login_required</span>

<span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">new_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="c"># ...</span></code></pre></figure>

<p>From now on, if the user is not authenticated they will be redirected to the login page:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/login-required.png" alt="Login required redirect" /></p>

<p>Notice the query string <strong>?next=/boards/1/new/</strong>. We can improve the log in template to make use of the <strong>next</strong>
variable and improve the user experience.</p>

<h5 id="configuring-login-next-redirect">Configuring Login Next Redirect</h5>

<p><strong>templates/login.html</strong>
<small><a href="https://gist.github.com/vitorfs/1ab597fe18e2dc56028f7aa8c3b588b3#file-login-html-L13" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
  <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
  <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"hidden"</span> <span class="na">name=</span><span class="s">"next"</span> <span class="na">value=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">next</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
  <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-primary btn-block"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/button&gt;</span>
<span class="nt">&lt;/form&gt;</span></code></pre></figure>

<p>Then if we try to log in now, the application will direct us back to where we were.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/Pixton_Comic_Magic.png" alt="Magic" /></p>

<p>So the <strong>next</strong> parameter is part of a built-in functionality.</p>

<h5 id="login-required-tests">Login Required Tests</h5>

<p>Let’s now add a test case to make sure this view is protected by the <code class="highlighter-rouge">@login_required</code> decorator. But first, let’s do
some refactoring in the <strong>boards/tests/test_views.py</strong> file.</p>

<p>Let’s split the <strong>test_views.py</strong> into three files:</p>

<ul>
  <li><strong>test_view_home.py</strong> will include the <strong>HomeTests</strong> class <small><a href="https://gist.github.com/vitorfs/6ac3aad244c856d418f18890efcb4a7e#file-test_view_home-py" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></li>
  <li><strong>test_view_board_topics.py</strong> will include the <strong>BoardTopicsTests</strong> class <small><a href="https://gist.github.com/vitorfs/6ac3aad244c856d418f18890efcb4a7e#file-test_view_board_topics-py" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></li>
  <li><strong>test_view_new_topic.py</strong> will include the <strong>NewTopicTests</strong> class <small><a href="https://gist.github.com/vitorfs/6ac3aad244c856d418f18890efcb4a7e#file-test_view_new_topic-py" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></li>
</ul>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |    |-- migrations/
 |    |    |-- templatetags/
 |    |    |-- tests/
 |    |    |    |-- __init__.py
 |    |    |    |-- test_templatetags.py
 |    |    |    |-- test_view_home.py          &lt;-- here
 |    |    |    |-- test_view_board_topics.py  &lt;-- here
 |    |    |    +-- test_view_new_topic.py     &lt;-- and here
 |    |    |-- __init__.py
 |    |    |-- admin.py
 |    |    |-- apps.py
 |    |    |-- models.py
 |    |    +-- views.py
 |    |-- myproject/
 |    |-- static/
 |    |-- templates/
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>Run the tests to make sure everything is working.</p>

<p>New let’s add a new test case in the <strong>test_view_new_topic.py</strong> to check if the view is decorated with
<code class="highlighter-rouge">@login_required</code>:</p>

<p><strong>boards/tests/test_view_new_topic.py</strong>
<small><a href="https://gist.github.com/vitorfs/13e75451396d76354b476edaefadbdab#file-test_view_new_topic-py-L84" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">..models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">class</span> <span class="nc">LoginRequiredNewTopicTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">login_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'login'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'{login_url}?next={url}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">login_url</span><span class="o">=</span><span class="n">login_url</span><span class="p">,</span> <span class="n">url</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">url</span><span class="p">))</span></code></pre></figure>

<p>In the test case above we are trying to make a request to the <strong>new topic</strong> view without being authenticated. The
expected result is for the request be redirected to the login view.</p>

<hr />

