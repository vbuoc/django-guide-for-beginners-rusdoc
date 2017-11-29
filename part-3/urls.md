<p>Proceeding with the development of our application, now we have to implement a new page to list all the topics that
belong to a given <strong>Board</strong>. Just to recap, below you can see the wireframe we drew in the previous tutorial:</p>

<div id="figure-1">
  <p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/wireframe-topics.png" alt="Wireframe Topics" /></p>
  <p style="text-align: center">
    <small>Figure 1: Boards project wireframe listing all topics in the Django board.</small>
  </p>
</div>

<p>We will start by editing the <strong>urls.py</strong> inside the <strong>myproject</strong> folder:</p>

<p><strong>myproject/urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>

<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^admin/'</span><span class="p">,</span> <span class="n">admin</span><span class="o">.</span><span class="n">site</span><span class="o">.</span><span class="n">urls</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>This time let’s take a moment and analyze the <code class="highlighter-rouge">urlpatterns</code> and <code class="highlighter-rouge">url</code>.</p>

<p>The URL dispatcher and <strong>URLconf</strong> (URL configuration) are fundamental parts of a Django application. In the beginning,
it can look confusing; I remember having a hard time when I first started developing with Django.</p>

<p>In fact, right now the Django Developers are working on a <a href="https://github.com/django/deps/blob/master/accepted/0201-simplified-routing-syntax.rst" target="_blank">proposal to make simplified routing syntax</a>.
But for now, as per the version 1.11, that’s what we have. So let’s try to understand how it works.</p>

<p>A project can have many <strong>urls.py</strong> distributed among the apps. But Django needs a <strong>url.py</strong> to use as a starting
point. This special <strong>urls.py</strong> is called <strong>root URLconf</strong>. It’s defined in the <strong>settings.py</strong> file.</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">ROOT_URLCONF</span> <span class="o">=</span> <span class="s">'myproject.urls'</span></code></pre></figure>

<p>It already comes configured, so you don’t need to change anything here.</p>

<p>When Django receives a request, it starts searching for a match in the project’s URLconf. It starts with the first
entry of the <code class="highlighter-rouge">urlpatterns</code> variable, and test the requested URL against each <code class="highlighter-rouge">url</code> entry.</p>

<p>If Django finds a match, it will pass the request to the <strong>view function</strong>, which is the second parameter of the
<code class="highlighter-rouge">url</code>. The order in the <code class="highlighter-rouge">urlpatterns</code> matters, because Django will stop searching as soon as it finds a match. Now, if
Django doesn’t find a match in the URLconf, it will raise a <strong>404</strong> exception, which is the error code for
<strong>Page Not Found</strong>.</p>

<p>This is the anatomy of the <code class="highlighter-rouge">url</code> function:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">def</span> <span class="nf">url</span><span class="p">(</span><span class="n">regex</span><span class="p">,</span> <span class="n">view</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="bp">None</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
    <span class="c"># ...</span></code></pre></figure>

<ul>
  <li><strong>regex</strong>: A regular expression for matching URL patterns in strings. Note that these regular expressions do not
search <strong>GET</strong> or <strong>POST</strong> parameters. In a request to <strong>http://127.0.0.1:8000/boards/?page=2</strong> only <strong>/boards/</strong> will
be processed.</li>
  <li><strong>view</strong>: A view function used to process the user request for a matched URL. It also accepts the return of the
<strong>django.conf.urls.include</strong> function, which is used to reference an external <strong>urls.py</strong> file. You can, for example,
use it to define a set of app specific URLs, and include it in the root URLconf using a prefix. We will explore more
on this concept later on.</li>
  <li><strong>kwargs</strong>: Arbitrary keyword arguments that’s passed to the target view. It is normally used to do some simple
customization on reusable views. We don’t use it very often.</li>
  <li><strong>name</strong>: A unique identifier for a given URL. This is a very important feature. Always remember to name your URLs.
With this, you can change a specific URL in the whole project by just changing the regex. So it’s important to never
hard code URLs in the views or templates, and always refer to the URLs by its name.</li>
</ul>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/Pixton_Comic_URL_Patterns.png" alt="Matching URL patterns" /></p>

<h5 id="basic-urls">Basic URLs</h5>

<p>Basic URLs are very simple to create. It’s just a matter of matching strings. For example, let’s say we wanted to
create an “about” page, it could be defined like this:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^about/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">about</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'about'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>We can also create deeper URL structures:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^about/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">about</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'about'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^about/company/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">about_company</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'about_company'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^about/author/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">about_author</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'about_author'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^about/author/vitor/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">about_vitor</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'about_vitor'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^about/author/erica/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">about_erica</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'about_erica'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^privacy/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">privacy_policy</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'privacy_policy'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Those are some examples of simple URL routing. For all the examples above, the view function will follow this
structure:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">def</span> <span class="nf">about</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="c"># do something...</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'about.html'</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">about_company</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="c"># do something else...</span>
    <span class="c"># return some data along with the view...</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'about_company.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'company_name'</span><span class="p">:</span> <span class="s">'Simple Complex'</span><span class="p">})</span></code></pre></figure>

<h5 id="advanced-urls">Advanced URLs</h5>

<p>A more advanced usage of URL routing is achieved by taking advantage of the regex to match certain types of data and
create dynamic URLs.</p>

<p>For example, to create a profile page, like many services do like github.com/vitorfs or twitter.com/vitorfs, where
“vitorfs” is my username, we can do the following:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^(?P&lt;username&gt;[</span><span class="err">\</span><span class="s">w.@+-]+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">user_profile</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'user_profile'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>This will match all valid usernames for a Django User model.</p>

<p>Now observe that the example above is a very <em>permissive</em> URL. That means it will match lots of URL patterns because
it is defined in the root of the URL, with no prefix like <strong>/profile/&lt;username&gt;/</strong>. In this case, if we wanted to
define a URL named <strong>/about/</strong>, we would have do define it <em>before</em> the username URL pattern:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^about/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">about</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'about'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^(?P&lt;username&gt;[</span><span class="err">\</span><span class="s">w.@+-]+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">user_profile</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'user_profile'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>If the “about” page was defined <em>after</em> the username URL pattern, Django would never find it, because the word “about”
would match the username regex, and the view <code class="highlighter-rouge">user_profile</code> would be processed instead of the <code class="highlighter-rouge">about</code> view function.</p>

<p>There are some side effects to that. For example, from now on, we would have to treat “about” as a forbidden username,
because if a user picked “about” as their username, this person would never see their profile page.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/Pixton_Comic_The_Order_Matters.png" alt="URL routing order matters" /></p>

<div class="info">
  <p>
    <strong><i class="fa fa-info-circle"></i> Sidenote:</strong>
    If you want to design cool URLs for user profiles, the easiest solution to avoid URL collision is by adding a prefix
    like <strong>/u/vitorfs/</strong>, or like Medium does <strong>/@vitorfs/</strong>, where "@" is the prefix.
  </p>
  <p>
    If you want no prefix at all, consider using a list of forbidden names like this:
    <a href="https://github.com/shouldbee/reserved-usernames/blob/master/reserved-usernames.csv" target="_blank">github.com/shouldbee/reserved-usernames</a>.
    Or another example is an application I developed when I was learning Django; I created my list at the time:
    <a href="https://github.com/vitorfs/parsifal/blob/master/parsifal/authentication/forms.py#L6" target="_blank">github.com/vitorfs/parsifal/</a>.
  </p>
  <p>
    Those collisions are very common. Take GitHub for example; they have this URL to list all the repositories you are
    currently watching: <a href="https://github.com/watching" target="_blank">github.com/watching</a>. Someone registered a username
    on GitHub with the name "watching," so this person can't see his profile page. We can see a user with this
    username exists by trying this URL: <a href="https://github.com/watching/repositories" target="_blank">github.com/watching/repositories</a>
    which was supposed to list the user's repositories, like mine for example <a href="https://github.com/vitorfs/repositories" target="_blank">github.com/vitorfs/repositories</a>.
  </p>
</div>

<p>The whole idea of this kind of URL routing is to create dynamic pages where part of the URL will be used as an
identifier for a certain resource, that will be used to compose a page. This identifier can be an integer ID or a
string for example.</p>

<p>Initially, we will be working with the <strong>Board</strong> ID to create a dynamic page for the <strong>Topics</strong>. Let’s read again the
example I gave at the beginning of the <strong>URLs</strong> section:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">)</span></code></pre></figure>

<p>The regex <code class="highlighter-rouge">\d+</code> will match an integer of arbitrary size. This integer will be used to retrieve the <strong>Board</strong> from the
database. Now observe that we wrote the regex as <code class="highlighter-rouge">(?P&lt;pk&gt;\d+)</code>, this is telling Django to capture the value into a
keyword argument named <strong>pk</strong>.</p>

<p>Here is how we write a view function for it:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="c"># do something...</span></code></pre></figure>

<p>Because we used the <code class="highlighter-rouge">(?P&lt;pk&gt;\d+)</code> regex, the keyword argument in the <code class="highlighter-rouge">board_topics</code> must be named <strong>pk</strong>.</p>

<p>If we wanted to use any name, we could do it like this:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">)</span></code></pre></figure>

<p>Then the view function could be defined like this:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">board_id</span><span class="p">):</span>
    <span class="c"># do something...</span></code></pre></figure>

<p>Or like this:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="nb">id</span><span class="p">):</span>
    <span class="c"># do something...</span></code></pre></figure>

<p>The name wouldn’t matter. But it’s a good practice to use named parameters because when we start composing bigger
URLs capturing multiple IDs and variables, it will be easier to read.</p>

<div class="info">
  <p>
    <strong><i class="fa fa-info-circle"></i> Sidenote:</strong> PK or ID?
  </p>
  <p>
    <strong>PK</strong> stands for <strong>Primary Key</strong>. It's a shortcut for accessing a model's primary key.
    All Django models have this attribute.
  </p>
  <p>
    For the most cases, using the <code>pk</code> property is the same as <code>id</code>. That's because if we
    don't define a primary key for a model, Django will automatically create an <code>AutoField</code> named
    <code>id</code>, which will be its primary key.
  </p>
  <p>
    If you defined a different primary key for a model, for example, let's say the field <code>email</code> is
    your primary key. To access it you could either use <code>obj.email</code> or <code>obj.pk</code>.
  </p>
</div>

<h5 id="using-the-urls-api">Using the URLs API</h5>

<p>It’s time to write some code. Let’s implement the topic listing page (see <a href="#figure-1">Figure 1</a>) I mentioned at the
beginning of the <strong>URLs</strong> section.</p>

<p>First, edit the <strong>urls.py</strong> adding our new URL route:</p>

<p><strong>myproject/urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>

<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^admin/'</span><span class="p">,</span> <span class="n">admin</span><span class="o">.</span><span class="n">site</span><span class="o">.</span><span class="n">urls</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Now let’s create the view function <code class="highlighter-rouge">board_topics</code>:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">home</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="c"># code suppressed for brevity</span>

<span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'topics.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">})</span></code></pre></figure>

<p>In the <strong>templates</strong> folder, create a new template named <strong>topics.html</strong>:</p>

<p><strong>templates/topics.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;ol</span> <span class="na">class=</span><span class="s">"breadcrumb my-4"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/li&gt;</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/li&gt;</span>
      <span class="nt">&lt;/ol&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<div class="info">
  <p>
    <strong><i class="fa fa-info-circle"></i> Note:</strong>
    For now we are simply creating new HTML templates. No worries, in the following section I will show you how to
    create reusable templates.
  </p>
</div>

<p>Now check the URL <strong>http://127.0.0.1:8000/boards/1/</strong> in a web browser. The result should be the following page:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/topics-1.png" alt="Topics Page" /></p>

<p>Time to write some tests! Edit the <strong>tests.py</strong> file and add the following tests in the bottom of the file:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">home</span><span class="p">,</span> <span class="n">board_topics</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">class</span> <span class="nc">HomeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">BoardTopicsTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_board_topics_view_success_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_board_topics_view_not_found_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">99</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">404</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_board_topics_url_resolves_board_topics_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/boards/1/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="p">,</span> <span class="n">board_topics</span><span class="p">)</span></code></pre></figure>

<p>A few things to note here. This time we used the <code class="highlighter-rouge">setUp</code> method. In the setup method, we created a <strong>Board</strong> instance
to use in the tests. We have to do that because the Django testing suite doesn’t run your tests against the current
database. To run the tests Django creates a new database on the fly, applies all the model migrations, runs the tests,
and when done, destroys the testing database.</p>

<p>So in the <code class="highlighter-rouge">setUp</code> method, we prepare the environment to run the tests, so to simulate a scenario.</p>

<ul>
  <li>The <code class="highlighter-rouge">test_board_topics_view_success_status_code</code> method: is testing if Django is returning a status code 200 (success)
for an existing <strong>Board</strong>.</li>
  <li>The <code class="highlighter-rouge">test_board_topics_view_not_found_status_code</code> method: is testing if Django is returning a status code 404
(page not found) for a <strong>Board</strong> that doesn’t exist in the database.</li>
  <li>The <code class="highlighter-rouge">test_board_topics_url_resolves_board_topics_view</code> method: is testing if Django is using the correct view
function to render the topics.</li>
</ul>

<p>Now it’s time to run the tests:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<p>And the output:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.E...
======================================================================
ERROR: test_board_topics_view_not_found_status_code (boards.tests.BoardTopicsTests)
----------------------------------------------------------------------
Traceback (most recent call last):
# ...
boards.models.DoesNotExist: Board matching query does not exist.

----------------------------------------------------------------------
Ran 5 tests in 0.093s

FAILED (errors=1)
Destroying test database for alias 'default'...</code></pre></figure>

<p>The test <strong>test_board_topics_view_not_found_status_code</strong> failed. We can see in the Traceback it returned an exception
“boards.models.DoesNotExist: Board matching query does not exist.”</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/topics-error-500.png" alt="Topics Error 500 Page" /></p>

<p>In production with <code class="highlighter-rouge">DEBUG=False</code>, the visitor would see a <strong>500 Internal Server Error</strong> page. But that’s not the
behavior we want.</p>

<p>We want to show a <strong>404 Page Not Found</strong>. So let’s refactor our view:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">django.http</span> <span class="kn">import</span> <span class="n">Http404</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">home</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="c"># code suppressed for brevity</span>

<span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="k">except</span> <span class="n">Board</span><span class="o">.</span><span class="n">DoesNotExist</span><span class="p">:</span>
        <span class="k">raise</span> <span class="n">Http404</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'topics.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">})</span></code></pre></figure>

<p>Let’s test again:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.....
----------------------------------------------------------------------
Ran 5 tests in 0.042s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Yay! Now it’s working as expected.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/topics-error-404.png" alt="Topics Error 404 Page" /></p>

<p>This is the default page Django show while with <code class="highlighter-rouge">DEBUG=False</code>. Later on, we can customize the 404 page to show something
else.</p>

<p>Now that’s a very common use case. In fact, Django has a shortcut to try to get an object, or return a 404 with the
object does not exist.</p>

<p>So let’s refactor the <strong>board_topics</strong> view again:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span><span class="p">,</span> <span class="n">get_object_or_404</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">home</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="c"># code suppressed for brevity</span>

<span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'topics.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">})</span></code></pre></figure>

<p>Changed the code? Test it.</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.....
----------------------------------------------------------------------
Ran 5 tests in 0.052s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Didn’t break anything. We can proceed with the development.</p>

<p>The next step now is to create the navigation links in the screens. The homepage should have a link to take the visitor
to the topics page of a given <strong>Board</strong>. Similarly, the topics page should have a link back to the homepage.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/wireframe-links.png" alt="Wireframe Links" /></p>

<p>We can start by writing some tests for the <code class="highlighter-rouge">HomeTests</code> class:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">HomeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_home_view_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_home_url_resolves_home_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="p">,</span> <span class="n">home</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_home_view_contains_link_to_topics_page</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">board_topics_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'href="{0}"'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">board_topics_url</span><span class="p">))</span></code></pre></figure>

<p>Observe that now we added a <strong>setUp</strong> method for the <strong>HomeTests</strong> as well. That’s because now we are going to need a
<strong>Board</strong> instance and also we moved the <strong>url</strong> and <strong>response</strong> to the <strong>setUp</strong>, so we can reuse the same response in the
new test.</p>

<p>The new test here is the <strong>test_home_view_contains_link_to_topics_page</strong>. Here we are using the <strong>assertContains</strong>
method to test if the response body contains a given text. The text we are using in the test, is the <code class="highlighter-rouge">href</code> part of an
<code class="highlighter-rouge">a</code> tag. So basically we are testing if the response body has the text <code class="highlighter-rouge">href="/boards/1/"</code>.</p>

<p>Let’s run the tests:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
....F.
======================================================================
FAIL: test_home_view_contains_link_to_topics_page (boards.tests.HomeTests)
----------------------------------------------------------------------
# ...

AssertionError: False is not true : Couldn't find 'href="/boards/1/"' in response

----------------------------------------------------------------------
Ran 6 tests in 0.034s

FAILED (failures=1)
Destroying test database for alias 'default'...</code></pre></figure>

<p>Now we can write the code that will make this test pass.</p>

<p>Edit the <strong>home.html</strong> template:</p>

<p><strong>templates/home.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="c">&lt;!-- code suppressed for brevity --&gt;</span>
<span class="nt">&lt;tbody&gt;</span>
  <span class="cp">{%</span> <span class="k">for</span> <span class="nv">board</span> <span class="ow">in</span> <span class="nv">boards</span> <span class="cp">%}</span>
    <span class="nt">&lt;tr&gt;</span>
      <span class="nt">&lt;td&gt;</span>
        <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"text-muted d-block"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.description</span> <span class="cp">}}</span><span class="nt">&lt;/small&gt;</span>
      <span class="nt">&lt;/td&gt;</span>
      <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>0<span class="nt">&lt;/td&gt;</span>
      <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>0<span class="nt">&lt;/td&gt;</span>
      <span class="nt">&lt;td&gt;&lt;/td&gt;</span>
    <span class="nt">&lt;/tr&gt;</span>
  <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
<span class="nt">&lt;/tbody&gt;</span>
<span class="c">&lt;!-- code suppressed for brevity --&gt;</span></code></pre></figure>

<p>So basically we changed the line:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span></code></pre></figure>

<p>To:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span></code></pre></figure>

<p>Always use the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">url</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> template tag to compose the applications URLs. The first parameter
is the <strong>name</strong> of the URL (defined in the URLconf, i.e., the <strong>urls.py</strong>), then you can pass an arbitrary number of arguments as needed.</p>

<p>If it were a simple URL, like the homepage, it would be just <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">url</span><span class="w"> </span><span class="err">'home'</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>.</p>

<p>Save the file and run the tests again:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
......
----------------------------------------------------------------------
Ran 6 tests in 0.037s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Good! Now we can check how it looks in the web browser:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/boards.png" alt="Boards with Link" /></p>

<p>Now the link back. We can write the test first:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">BoardTopicsTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># code suppressed for brevity...</span>

    <span class="k">def</span> <span class="nf">test_board_topics_view_contains_link_back_to_homepage</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">board_topics_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">board_topics_url</span><span class="p">)</span>
        <span class="n">homepage_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="n">response</span><span class="p">,</span> <span class="s">'href="{0}"'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">homepage_url</span><span class="p">))</span></code></pre></figure>

<p>Run the tests:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.F.....
======================================================================
FAIL: test_board_topics_view_contains_link_back_to_homepage (boards.tests.BoardTopicsTests)
----------------------------------------------------------------------
Traceback (most recent call last):
# ...

AssertionError: False is not true : Couldn't find 'href="/"' in response

----------------------------------------------------------------------
Ran 7 tests in 0.054s

FAILED (failures=1)
Destroying test database for alias 'default'...</code></pre></figure>

<p>Update the board topics template:</p>

<p><strong>templates/topics.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span><span class="c">&lt;!-- code suppressed for brevity --&gt;</span><span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;ol</span> <span class="na">class=</span><span class="s">"breadcrumb my-4"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/li&gt;</span>
      <span class="nt">&lt;/ol&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>Run the tests:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.......
----------------------------------------------------------------------
Ran 7 tests in 0.061s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/board_topics.png" alt="Board Topics with Link" /></p>

<p>As I mentioned before, URL routing is a fundamental part of a web application. With this knowledge, we should be able
to proceed with the development. Next, to complete the section about URLs, you will find a summary of useful URL
patterns.</p>

<h5 id="list-of-useful-url-patterns">List of Useful URL Patterns</h5>

<p>The trick part is the <strong>regex</strong>. So I prepared a list of the most used URL patterns. You can always refer to this
list when you need a specific URL.</p>

<table style="font-size: 1.4rem">
  <thead>
    <tr>
      <th colspan="2" style="font-size: 2rem">Primary Key AutoField</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Regex</th>
      <td><code>(?P&lt;pk&gt;\d+)</code></td>
    </tr>
    <tr>
      <th>Example</th>
      <td><code>url(r'^questions/(?P&lt;pk&gt;\d+)/$', views.question, name='question')</code></td>
    </tr>
    <tr>
      <th>Valid URL</th>
      <td><code>/questions/934/</code></td>
    </tr>
    <tr>
      <th>Captures</th>
      <td><code>{'pk': '934'}</code></td>
    </tr>
  </tbody>
</table>

<table style="font-size: 1.4rem">
  <thead>
    <tr>
      <th colspan="2" style="font-size: 2rem">Slug Field</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Regex</th>
      <td><code>(?P&lt;slug&gt;[-\w]+)</code></td>
    </tr>
    <tr>
      <th>Example</th>
      <td><code>url(r'^posts/(?P&lt;slug&gt;[-\w]+)/$', views.post, name='post')</code></td>
    </tr>
    <tr>
      <th>Valid URL</th>
      <td><code>/posts/hello-world/</code></td>
    </tr>
    <tr>
      <th>Captures</th>
      <td><code>{'slug': 'hello-world'}</code></td>
    </tr>
  </tbody>
</table>

<table style="font-size: 1.4rem">
  <thead>
    <tr>
      <th colspan="2" style="font-size: 2rem">Slug Field with Primary Key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Regex</th>
      <td><code>(?P&lt;slug&gt;[-\w]+)-(?P&lt;pk&gt;\d+)</code></td>
    </tr>
    <tr>
      <th>Example</th>
      <td><code>url(r'^blog/(?P&lt;slug&gt;[-\w]+)-(?P&lt;pk&gt;\d+)/$', views.blog_post, name='blog_post')</code></td>
    </tr>
    <tr>
      <th>Valid URL</th>
      <td><code>/blog/hello-world-159/</code></td>
    </tr>
    <tr>
      <th>Captures</th>
      <td><code>{'slug': 'hello-world', 'pk': '159'}</code></td>
    </tr>
  </tbody>
</table>

<table style="font-size: 1.4rem">
  <thead>
    <tr>
      <th colspan="2" style="font-size: 2rem">Django User Username</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Regex</th>
      <td><code>(?P&lt;username&gt;[\w.@+-]+)</code></td>
    </tr>
    <tr>
      <th>Example</th>
      <td><code>url(r'^profile/(?P&lt;username&gt;[\w.@+-]+)/$', views.user_profile, name='user_profile')</code></td>
    </tr>
    <tr>
      <th>Valid URL</th>
      <td><code>/profile/vitorfs/</code></td>
    </tr>
    <tr>
      <th>Captures</th>
      <td><code>{'username': 'vitorfs'}</code></td>
    </tr>
  </tbody>
</table>

<table style="font-size: 1.4rem">
  <thead>
    <tr>
      <th colspan="2" style="font-size: 2rem">Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Regex</th>
      <td><code>(?P&lt;year&gt;[0-9]{4})</code></td>
    </tr>
    <tr>
      <th>Example</th>
      <td><code>url(r'^articles/(?P&lt;year&gt;[0-9]{4})/$', views.year_archive, name='year')</code></td>
    </tr>
    <tr>
      <th>Valid URL</th>
      <td><code>/articles/2016/</code></td>
    </tr>
    <tr>
      <th>Captures</th>
      <td><code>{'year': '2016'}</code></td>
    </tr>
  </tbody>
</table>

<table style="font-size: 1.4rem">
  <thead>
    <tr>
      <th colspan="2" style="font-size: 2rem">Year / Month</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Regex</th>
      <td><code>(?P&lt;year&gt;[0-9]{4})/(?P&lt;month&gt;[0-9]{2})</code></td>
    </tr>
    <tr>
      <th>Example</th>
      <td><code>url(r'^articles/(?P&lt;year&gt;[0-9]{4})/(?P&lt;month&gt;[0-9]{2})/$', views.month_archive, name='month')</code></td>
    </tr>
    <tr>
      <th>Valid URL</th>
      <td><code>/articles/2016/01/</code></td>
    </tr>
    <tr>
      <th>Captures</th>
      <td><code>{'year': '2016', 'month': '01'}</code></td>
    </tr>
  </tbody>
</table>

<p>You can find more details about those patterns in this post: <a href="/references/2016/10/10/url-patterns.html">List of Useful URL Patterns</a>.</p>

<hr />
