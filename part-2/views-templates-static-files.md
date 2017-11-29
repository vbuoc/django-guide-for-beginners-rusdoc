Views, Templates, and Static Files

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html


<p>At the moment we already have a view named <code class="highlighter-rouge">home</code> displaying “Hello, World!” in the homepage of our application.</p>

<p><strong>myproject/urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>

<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^admin/'</span><span class="p">,</span> <span class="n">admin</span><span class="o">.</span><span class="n">site</span><span class="o">.</span><span class="n">urls</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.http</span> <span class="kn">import</span> <span class="n">HttpResponse</span>

<span class="k">def</span> <span class="nf">home</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">HttpResponse</span><span class="p">(</span><span class="s">'Hello, World!'</span><span class="p">)</span></code></pre></figure>

<p>We can use this as our starting point. If you recall our wireframes, the <a href="#figure-5">Figure 5</a> showed how the homepage
should look like. What we want to do is display a list of boards in a table alongside with some other information.</p>

<p>The first thing to do is import the <strong>Board</strong> model and list all the existing boards:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.http</span> <span class="kn">import</span> <span class="n">HttpResponse</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">home</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="n">boards</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">all</span><span class="p">()</span>
    <span class="n">boards_names</span> <span class="o">=</span> <span class="nb">list</span><span class="p">()</span>

    <span class="k">for</span> <span class="n">board</span> <span class="ow">in</span> <span class="n">boards</span><span class="p">:</span>
        <span class="n">boards_names</span><span class="o">.</span><span class="n">append</span><span class="p">(</span><span class="n">board</span><span class="o">.</span><span class="n">name</span><span class="p">)</span>

    <span class="n">response_html</span> <span class="o">=</span> <span class="s">'&lt;br&gt;'</span><span class="o">.</span><span class="n">join</span><span class="p">(</span><span class="n">boards_names</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">HttpResponse</span><span class="p">(</span><span class="n">response_html</span><span class="p">)</span></code></pre></figure>

<p>And the result would be this simple HTML page:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-httpresponse.png" alt="Boards Homepage HttpResponse" /></p>

<p>But let’s stop right here. We are not going very far rendering HTML like this. For this simple view, all we need is a
list of boards; then the rendering part is a job for the <strong>Django Template Engine</strong>.</p>

<h5 id="django-template-engine-setup">Django Template Engine Setup</h5>

<p>Create a new folder named <strong>templates</strong> alongside with the <strong>boards</strong> and <strong>mysite</strong> folders:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- boards/
 |    |-- myproject/
 |    |-- templates/   &lt;-- here!
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>Now within the <strong>templates</strong> folder, create an HTML file named <strong>home.html</strong>:</p>

<p><strong>templates/home.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span>Boards<span class="nt">&lt;/title&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;h1&gt;</span>Boards<span class="nt">&lt;/h1&gt;</span>

    <span class="cp">{%</span> <span class="k">for</span> <span class="nv">board</span> <span class="ow">in</span> <span class="nv">boards</span> <span class="cp">%}</span>
      <span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span> <span class="nt">&lt;br&gt;</span>
    <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>In the example above we are mixing raw HTML with some special tags <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">for</span><span class="w"> </span><span class="err">...</span><span class="w"> </span><span class="err">in</span><span class="w"> </span><span class="err">...</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> and
<code class="highlighter-rouge"><span class="p">{</span><span class="err">{</span><span class="w"> </span><span class="err">variable</span><span class="w"> </span><span class="p">}</span><span class="err">}</span></code>. They are part of the Django Template Language. The example above shows how to
iterate over a list of objects using a <code class="highlighter-rouge">for</code>. The <code class="highlighter-rouge"><span class="p">{</span><span class="err">{</span><span class="w"> </span><span class="err">board.name</span><span class="w"> </span><span class="p">}</span><span class="err">}</span></code> renders the name of the
board in the HTML template, generating a dynamic HTML document.</p>

<p>Before we can use this HTML page, we have to tell Django where to find our application’s templates.</p>

<p>Open the <strong>settings.py</strong> inside the <strong>myproject</strong> directory and search for the <code class="highlighter-rouge">TEMPLATES</code> variable and set the <code class="highlighter-rouge">DIRS</code>
key to <code class="highlighter-rouge">os.path.join(BASE_DIR, 'templates')</code>:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">TEMPLATES</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">{</span>
        <span class="s">'BACKEND'</span><span class="p">:</span> <span class="s">'django.template.backends.django.DjangoTemplates'</span><span class="p">,</span>
        <span class="s">'DIRS'</span><span class="p">:</span> <span class="p">[</span>
            <span class="n">os</span><span class="o">.</span><span class="n">path</span><span class="o">.</span><span class="n">join</span><span class="p">(</span><span class="n">BASE_DIR</span><span class="p">,</span> <span class="s">'templates'</span><span class="p">)</span>
        <span class="p">],</span>
        <span class="s">'APP_DIRS'</span><span class="p">:</span> <span class="bp">True</span><span class="p">,</span>
        <span class="s">'OPTIONS'</span><span class="p">:</span> <span class="p">{</span>
            <span class="s">'context_processors'</span><span class="p">:</span> <span class="p">[</span>
                <span class="s">'django.template.context_processors.debug'</span><span class="p">,</span>
                <span class="s">'django.template.context_processors.request'</span><span class="p">,</span>
                <span class="s">'django.contrib.auth.context_processors.auth'</span><span class="p">,</span>
                <span class="s">'django.contrib.messages.context_processors.messages'</span><span class="p">,</span>
            <span class="p">],</span>
        <span class="p">},</span>
    <span class="p">},</span>
<span class="p">]</span></code></pre></figure>

<p>Basically what this line is doing is finding the full path of your project directory and appending “/templates” to it.</p>

<p>We can debug this using the Python shell:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py shell</code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf</span> <span class="kn">import</span> <span class="n">settings</span>

<span class="n">settings</span><span class="o">.</span><span class="n">BASE_DIR</span>
<span class="s">'/Users/vitorfs/Development/myproject'</span>

<span class="kn">import</span> <span class="nn">os</span>

<span class="n">os</span><span class="o">.</span><span class="n">path</span><span class="o">.</span><span class="n">join</span><span class="p">(</span><span class="n">settings</span><span class="o">.</span><span class="n">BASE_DIR</span><span class="p">,</span> <span class="s">'templates'</span><span class="p">)</span>
<span class="s">'/Users/vitorfs/Development/myproject/templates'</span></code></pre></figure>

<p>See? It’s just pointing to the <strong>templates</strong> folder we created in the previous steps.</p>

<p>Now we can update our <strong>home</strong> view:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">home</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="n">boards</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">all</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'home.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'boards'</span><span class="p">:</span> <span class="n">boards</span><span class="p">})</span></code></pre></figure>

<p>The resulting HTML:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-render.png" alt="Boards Homepage render" /></p>

<p>We can improve the HTML template to use a table instead:</p>

<p><strong>templates/home.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span>Boards<span class="nt">&lt;/title&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;h1&gt;</span>Boards<span class="nt">&lt;/h1&gt;</span>

    <span class="nt">&lt;table</span> <span class="na">border=</span><span class="s">"1"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;thead&gt;</span>
        <span class="nt">&lt;tr&gt;</span>
          <span class="nt">&lt;th&gt;</span>Board<span class="nt">&lt;/th&gt;</span>
          <span class="nt">&lt;th&gt;</span>Posts<span class="nt">&lt;/th&gt;</span>
          <span class="nt">&lt;th&gt;</span>Topics<span class="nt">&lt;/th&gt;</span>
          <span class="nt">&lt;th&gt;</span>Last Post<span class="nt">&lt;/th&gt;</span>
        <span class="nt">&lt;/tr&gt;</span>
      <span class="nt">&lt;/thead&gt;</span>
      <span class="nt">&lt;tbody&gt;</span>
        <span class="cp">{%</span> <span class="k">for</span> <span class="nv">board</span> <span class="ow">in</span> <span class="nv">boards</span> <span class="cp">%}</span>
          <span class="nt">&lt;tr&gt;</span>
            <span class="nt">&lt;td&gt;</span>
              <span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;br&gt;</span>
              <span class="nt">&lt;small</span> <span class="na">style=</span><span class="s">"color: #888"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.description</span> <span class="cp">}}</span><span class="nt">&lt;/small&gt;</span>
            <span class="nt">&lt;/td&gt;</span>
            <span class="nt">&lt;td&gt;</span>0<span class="nt">&lt;/td&gt;</span>
            <span class="nt">&lt;td&gt;</span>0<span class="nt">&lt;/td&gt;</span>
            <span class="nt">&lt;td&gt;&lt;/td&gt;</span>
          <span class="nt">&lt;/tr&gt;</span>
        <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
      <span class="nt">&lt;/tbody&gt;</span>
    <span class="nt">&lt;/table&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-render-2.png" alt="Boards Homepage render" /></p>

<h5 id="testing-the-homepage">Testing the Homepage</h5>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Testing.png" alt="Testing Comic" /></p>

<p>This is going to be a recurrent subject, and we are going to explore together different concepts and strategies
throughout the whole tutorial series.</p>

<p>Let’s write our first test. For now, we will be working in the <strong>tests.py</strong> file inside the <strong>boards</strong> app:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>

<span class="k">class</span> <span class="nc">HomeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_home_view_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span></code></pre></figure>

<p>This is a very simple test case but extremely useful. We are testing the <em>status code</em> of the response. The status
code 200 means <strong>success</strong>.</p>

<p>We can check the status code of the response in the console:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/test-homepage-view-status-code-200.png" alt="Response 200" /></p>

<p>If there were an uncaught exception, syntax error, or anything, Django would return a status code <strong>500</strong> instead,
which means <strong>Internal Server Error</strong>. Now, imagine our application has 100 views. If we wrote just this simple test
for all our views, with just one command, we would be able to test if all views are returning a success code, so the
user does not see any error message anywhere. Without automate tests, we would need to check each page, one by one.</p>

<p>To execute the Django’s test suite:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.041s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Now we can test if Django returned the correct view function for the requested URL. This is also a useful test because
as we progress with the development, you will see that the <strong>urls.py</strong> module can get very big and complex. The URL
conf is all about resolving regex. There are some cases where we have a very permissive URL, so Django can end up
returning the wrong view function.</p>

<p>Here’s how we do it:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">home</span>

<span class="k">class</span> <span class="nc">HomeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_home_view_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_home_url_resolves_home_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="p">,</span> <span class="n">home</span><span class="p">)</span></code></pre></figure>

<p>In the second test, we are making use of the <code class="highlighter-rouge">resolve</code> function. Django uses it to match a requested URL with a list of
URLs listed in the <strong>urls.py</strong> module. This test will make sure the URL <code class="highlighter-rouge">/</code>, which is the root URL, is returning the
home view.</p>

<p>Test it again:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.027s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>To see more detail about the test execution, set the <strong>verbosity</strong> to a higher level:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span> --verbosity<span class="o">=</span>2</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default' ('file:memorydb_default?mode=memory&amp;cache=shared')...
Operations to perform:
  Synchronize unmigrated apps: messages, staticfiles
  Apply all migrations: admin, auth, boards, contenttypes, sessions
Synchronizing apps without migrations:
  Creating tables...
    Running deferred SQL...
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
  Applying sessions.0001_initial... OK
System check identified no issues (0 silenced).
test_home_url_resolves_home_view (boards.tests.HomeTests) ... ok
test_home_view_status_code (boards.tests.HomeTests) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.017s

OK
Destroying test database for alias 'default' ('file:memorydb_default?mode=memory&amp;cache=shared')...</code></pre></figure>

<p>Verbosity determines the amount of notification and debug information that will be printed to the console; 0 is no
output, 1 is normal output, and 2 is verbose output.</p>

<h5 id="static-files-setup">Static Files Setup</h5>

<p>Static files are the CSS, JavaScripts, Fonts, Images, or any other resources we may use to compose the user interface.</p>

<p>As it is, Django doesn’t serve those files. Except during the development process, so to make our lives easier.
But Django provides some features to help us manage the static files. Those features are available in the
<strong>django.contrib.staticfiles</strong> application already listed in the <code class="highlighter-rouge">INSTALLED_APPS</code> configuration.</p>

<p>With so many front-end component libraries available, there’s no reason for us keep rendering basic HTML documents.
We can easily add Bootstrap 4 to our project. Bootstrap is an open source toolkit for developing with HTML, CSS, and
JavaScript.</p>

<p>In the project root directory, alongside with the <strong>boards</strong>, <strong>templates</strong>, and <strong>myproject</strong> folders, create a new
folder named <strong>static</strong>, and within the <strong>static</strong> folder create another one named <strong>css</strong>:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- boards/
 |    |-- myproject/
 |    |-- templates/
 |    |-- static/       &lt;-- here
 |    |    +-- css/     &lt;-- and here
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>Go to <a href="https://getbootstrap.com/docs/4.0/getting-started/download/#compiled-css-and-js" target="_blank">getbootstrap.com</a> and download the latest version:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/bootstrap-download.png" alt="Bootstrap Download" /></p>

<p>Download the <strong>Compiled CSS and JS</strong> version.</p>

<p>In your computer, extract the <strong>bootstrap-4.0.0-beta-dist.zip</strong> file you downloaded from the Bootstrap website,
copy the file <strong>css/bootstrap.min.css</strong> to our project’s css folder:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- boards/
 |    |-- myproject/
 |    |-- templates/
 |    |-- static/
 |    |    +-- css/
 |    |         +-- bootstrap.min.css    &lt;-- here
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>The next step is to instruct Django where to find the static files. Open the <strong>settings.py</strong>, scroll to the bottom
of the file and just after the <code class="highlighter-rouge">STATIC_URL</code>, add the following:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">STATIC_URL</span> <span class="o">=</span> <span class="s">'/static/'</span>

<span class="n">STATICFILES_DIRS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">os</span><span class="o">.</span><span class="n">path</span><span class="o">.</span><span class="n">join</span><span class="p">(</span><span class="n">BASE_DIR</span><span class="p">,</span> <span class="s">'static'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Same thing as the <code class="highlighter-rouge">TEMPLATES</code> directory, remember?</p>

<p>Now we have to load the static files (the Bootstrap CSS file) in our template:</p>

<p><strong>templates/home.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span>Boards<span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="c">&lt;!-- body suppressed for brevity ... --&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>First we load the Static Files App template tags by using the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">load</span><span class="w"> </span><span class="err">static</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> in the beginning
of the template.</p>

<p>The template tag <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">static</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> is used to compose the URL where the resource lives. In this case,
the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">static</span><span class="w"> </span><span class="err">'css/bootstrap.min.css'</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> will return <strong>/static/css/bootstrap.min.css</strong>, which is
equivalent to <strong>http://127.0.0.1:8000/static/css/bootstrap.min.css</strong>.</p>

<p>The <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">static</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> template tag uses the <code class="highlighter-rouge">STATIC_URL</code> configuration in the <strong>settings.py</strong> to
compose the final URL. For example, if you hosted your static files in a subdomain like
<strong>https://static.example.com/</strong>, we would set the <code class="highlighter-rouge">STATIC_URL=https://static.example.com/</code> then the
<code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">static</span><span class="w"> </span><span class="err">'css/bootstrap.min.css'</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> would return
<strong>https://static.example.com/css/bootstrap.min.css</strong>.</p>

<p>If none of this makes sense for you at the moment, don’t worry. Just remember to use the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">static</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>
whenever you need to refer to a CSS, JavaScript or image file. Later on, when we start working with Deployment, we will
discuss more it. For now, we are all set up.</p>

<p>Refreshing the page <strong>127.0.0.1:8000</strong> we can see it worked:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-bootstrap.png" alt="Boards Homepage Bootstrap" /></p>

<p>Now we can edit the template so to take advantage of the Bootstrap CSS:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span>Boards<span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;ol</span> <span class="na">class=</span><span class="s">"breadcrumb my-4"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/li&gt;</span>
      <span class="nt">&lt;/ol&gt;</span>
      <span class="nt">&lt;table</span> <span class="na">class=</span><span class="s">"table"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;thead</span> <span class="na">class=</span><span class="s">"thead-inverse"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;tr&gt;</span>
            <span class="nt">&lt;th&gt;</span>Board<span class="nt">&lt;/th&gt;</span>
            <span class="nt">&lt;th&gt;</span>Posts<span class="nt">&lt;/th&gt;</span>
            <span class="nt">&lt;th&gt;</span>Topics<span class="nt">&lt;/th&gt;</span>
            <span class="nt">&lt;th&gt;</span>Last Post<span class="nt">&lt;/th&gt;</span>
          <span class="nt">&lt;/tr&gt;</span>
        <span class="nt">&lt;/thead&gt;</span>
        <span class="nt">&lt;tbody&gt;</span>
          <span class="cp">{%</span> <span class="k">for</span> <span class="nv">board</span> <span class="ow">in</span> <span class="nv">boards</span> <span class="cp">%}</span>
            <span class="nt">&lt;tr&gt;</span>
              <span class="nt">&lt;td&gt;</span>
                <span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span>
                <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"text-muted d-block"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.description</span> <span class="cp">}}</span><span class="nt">&lt;/small&gt;</span>
              <span class="nt">&lt;/td&gt;</span>
              <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>0<span class="nt">&lt;/td&gt;</span>
              <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>0<span class="nt">&lt;/td&gt;</span>
              <span class="nt">&lt;td&gt;&lt;/td&gt;</span>
            <span class="nt">&lt;/tr&gt;</span>
          <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
        <span class="nt">&lt;/tbody&gt;</span>
      <span class="nt">&lt;/table&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>The result now:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-bootstrap-2.png" alt="Boards Homepage Bootstrap" /></p>

<p>So far we are adding new boards using the interactive console (<code class="highlighter-rouge">python manage.py shell</code>). But we need a better way to
do it. In the next section, we are going to implement an admin interface for the website administrator manage it.</p>

<hr />
