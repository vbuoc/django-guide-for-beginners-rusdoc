<h4 id="logout">Logout</h4>

<p>To keep a natural flow in the implementation, let’s add the log out view. First, edit the <strong>urls.py</strong> to add a new
route:</p>

<p><strong>myproject/urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">auth_views</span>

<span class="kn">from</span> <span class="nn">accounts</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">accounts_views</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^signup/$'</span><span class="p">,</span> <span class="n">accounts_views</span><span class="o">.</span><span class="n">signup</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'signup'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^logout/$'</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">LogoutView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'logout'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/new/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">new_topic</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'new_topic'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^admin/'</span><span class="p">,</span> <span class="n">admin</span><span class="o">.</span><span class="n">site</span><span class="o">.</span><span class="n">urls</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>We imported the <strong>views</strong> from the Django’s contrib module. We renamed it to <strong>auth_views</strong> to avoid clashing
with the <strong>boards.views</strong>. Notice that this view is a little bit different: <code class="highlighter-rouge">LogoutView.as_view()</code>. It’s a Django’s
class-based view. So far we have only implemented classes as Python functions. The class-based views provide a more
flexible way to extend and reuse views. We will discuss more that subject later on.</p>

<p>Open the <strong>settings.py</strong> file and add the <code class="highlighter-rouge">LOGOUT_REDIRECT_URL</code> variable to the bottom of the file:</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">LOGOUT_REDIRECT_URL</span> <span class="o">=</span> <span class="s">'home'</span></code></pre></figure>

<p>Here we are passing the name of the URL pattern we want to redirect the user after the log out.</p>

<p>After that, it’s already done. Just access the URL <strong>127.0.0.1:8000/logout/</strong> and you will be logged out. But hold on
a second. Before you log out, let’s create the dropdown menu for logged in users.</p>

<hr />
