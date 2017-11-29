<h4 id="my-account-view">My Account View</h4>

<p>Okay, so, this is going to be our last view. After that, we will be working on improving the existing features.</p>

<p><strong>accounts/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/ea62417b7a450050f2feeeb69b775996" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.decorators</span> <span class="kn">import</span> <span class="n">login_required</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">reverse_lazy</span>
<span class="kn">from</span> <span class="nn">django.utils.decorators</span> <span class="kn">import</span> <span class="n">method_decorator</span>
<span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">UpdateView</span>

<span class="nd">@method_decorator</span><span class="p">(</span><span class="n">login_required</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'dispatch'</span><span class="p">)</span>
<span class="k">class</span> <span class="nc">UserUpdateView</span><span class="p">(</span><span class="n">UpdateView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">User</span>
    <span class="n">fields</span> <span class="o">=</span> <span class="p">(</span><span class="s">'first_name'</span><span class="p">,</span> <span class="s">'last_name'</span><span class="p">,</span> <span class="s">'email'</span><span class="p">,</span> <span class="p">)</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'my_account.html'</span>
    <span class="n">success_url</span> <span class="o">=</span> <span class="n">reverse_lazy</span><span class="p">(</span><span class="s">'my_account'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get_object</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">request</span><span class="o">.</span><span class="n">user</span></code></pre></figure>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/27d87452e7584cb8c489625f507ed7aa#file-urls-py-L32" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">accounts</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">accounts_views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c"># ...</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^settings/account/$'</span><span class="p">,</span> <span class="n">accounts_views</span><span class="o">.</span><span class="n">UserUpdateView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'my_account'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p><strong>templates/my_account.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>My account<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>My account<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-6 col-md-8 col-sm-10"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
        <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
        <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Save changes<span class="nt">&lt;/button&gt;</span>
      <span class="nt">&lt;/form&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/account.png" alt="My Account" /></p>

<hr />

