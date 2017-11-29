<h4 id="sign-up">Sign Up</h4>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Sign_Up_View.png" alt="Sign Up" /></p>

<p>Let’s start by creating the sign up view. First thing, create a new route in the <strong>urls.py</strong> file:</p>

<p><strong>myproject/urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>

<span class="kn">from</span> <span class="nn">accounts</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">accounts_views</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^signup/$'</span><span class="p">,</span> <span class="n">accounts_views</span><span class="o">.</span><span class="n">signup</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'signup'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/new/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">new_topic</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'new_topic'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^admin/'</span><span class="p">,</span> <span class="n">admin</span><span class="o">.</span><span class="n">site</span><span class="o">.</span><span class="n">urls</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Notice how we are importing the <strong>views</strong> module from the <strong>accounts</strong> app in a different way:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">accounts</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">accounts_views</span></code></pre></figure>

<p>We are giving an alias because otherwise, it would clash with the <strong>boards’</strong> views. We can improve the <strong>urls.py</strong>
design later on. But for now, let’s focus on the authentication features.</p>

<p>Now edit the <strong>views.py</strong> inside the <strong>accounts</strong> app and create a new view named <strong>signup</strong>:</p>

<p><strong>accounts/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span>

<span class="k">def</span> <span class="nf">signup</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'signup.html'</span><span class="p">)</span></code></pre></figure>

<p>Create the new template, named <strong>signup.html</strong>:</p>

<p><strong>templates/signup.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;h2&gt;</span>Sign up<span class="nt">&lt;/h2&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Open the URL <strong>http://127.0.0.1:8000/signup/</strong> in the browser, check if everything is working:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup.png" alt="Sign up" /></p>

<p>Time to write some tests:</p>

<p><strong>accounts/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">signup</span>

<span class="k">class</span> <span class="nc">SignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_signup_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'signup'</span><span class="p">)</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_signup_url_resolves_signup_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/signup/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="p">,</span> <span class="n">signup</span><span class="p">)</span></code></pre></figure>

<p>Testing the status code (200 = success) and if the URL <strong>/signup/</strong> is returning the correct view function.</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..................
----------------------------------------------------------------------
Ran 18 tests in 0.652s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>For the authentication views (sign up, log in, password reset, etc.) we won’t use the top bar or the breadcrumb.
We can still use the <strong>base.html</strong> template. It just needs some tweaks:</p>

<p><strong>templates/base.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span><span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Django Boards<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span><span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">href=</span><span class="s">"https://fonts.googleapis.com/css?family=Peralta"</span> <span class="na">rel=</span><span class="s">"stylesheet"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/app.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="k">block</span> <span class="nv">stylesheet</span> <span class="cp">%}{%</span> <span class="k">endblock</span> <span class="cp">%}</span>  <span class="c">&lt;!-- HERE --&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="cp">{%</span> <span class="k">block</span> <span class="nv">body</span> <span class="cp">%}</span>  <span class="c">&lt;!-- HERE --&gt;</span>
      <span class="nt">&lt;nav</span> <span class="na">class=</span><span class="s">"navbar navbar-expand-lg navbar-dark bg-dark"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"navbar-brand"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Django Boards<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/nav&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;ol</span> <span class="na">class=</span><span class="s">"breadcrumb my-4"</span><span class="nt">&gt;</span>
          <span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
          <span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
        <span class="nt">&lt;/ol&gt;</span>
        <span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="cp">{%</span> <span class="k">endblock</span> <span class="nv">body</span> <span class="cp">%}</span>  <span class="c">&lt;!-- AND HERE --&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>I marked with comments the new bits in the <strong>base.html</strong> template. The block
<code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">stylesheet</span><span class="w"> </span><span class="err">%</span><span class="p">}{</span><span class="err">%</span><span class="w"> </span><span class="err">endblock</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> will be used to add extra CSS, specific to some pages.</p>

<p>The block <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">body</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> is wrapping the whole HTML document. We can use it to have an
empty document taking advantage of the head of the <strong>base.html</strong>. Notice how we named the end block
<code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">endblock</span><span class="w"> </span><span class="err">body</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>. In cases like this, it’s a good practice to name the closing tag, so it’s
easier to identify where it ends.</p>

<p>Now on the <strong>signup.html</strong> template, instead of using the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">content</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>, we can use the
<code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">body</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>.</p>

<p><strong>templates/signup.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">body</span> <span class="cp">%}</span>
  <span class="nt">&lt;h2&gt;</span>Sign up<span class="nt">&lt;/h2&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-2.png" alt="Sign up" /></p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Empty.png" alt="Too Empty" /></p>

<p>Time to create the sign up form. Django has a built-in form named <strong>UserCreationForm</strong>. Let’s use it:</p>

<p><strong>accounts/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">UserCreationForm</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span>

<span class="k">def</span> <span class="nf">signup</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="n">form</span> <span class="o">=</span> <span class="n">UserCreationForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'signup.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p><strong>templates/signup.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">body</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;h2&gt;</span>Sign up<span class="nt">&lt;/h2&gt;</span>
    <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
      <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
      <span class="cp">{{</span> <span class="nv">form.as_p</span> <span class="cp">}}</span>
      <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span><span class="nt">&gt;</span>Create an account<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;/form&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-3.png" alt="Sign up" /></p>

<p>Looking a little bit messy, right? We can use our <strong>form.html</strong> template to make it look better:</p>

<p><strong>templates/signup.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">body</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;h2&gt;</span>Sign up<span class="nt">&lt;/h2&gt;</span>
    <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
      <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
      <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span><span class="nt">&gt;</span>Create an account<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;/form&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-4.png" alt="Sign up" /></p>

<p>Uh, almost there. Currently, our <strong>form.html</strong> partial template is displaying some raw HTML. It’s a security feature.
By default Django treats all strings as unsafe, escaping all the special characters that may cause trouble. But in this
case, we can trust it.</p>

<p><strong>templates/includes/form.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">widget_tweaks</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">for</span> <span class="nv">field</span> <span class="ow">in</span> <span class="nv">form</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"form-group"</span><span class="nt">&gt;</span>
    <span class="cp">{{</span> <span class="nv">field.label_tag</span> <span class="cp">}}</span>

    <span class="c">&lt;!-- code suppressed for brevity --&gt;</span>

    <span class="cp">{%</span> <span class="k">if</span> <span class="nv">field.help_text</span> <span class="cp">%}</span>
      <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"form-text text-muted"</span><span class="nt">&gt;</span>
        <span class="cp">{{</span> <span class="nv">field.help_text</span><span class="o">|</span><span class="nf">safe</span> <span class="cp">}}</span>  <span class="c">&lt;!-- new code here --&gt;</span>
      <span class="nt">&lt;/small&gt;</span>
    <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<p>Basically, in the previous template we added the option <code class="highlighter-rouge">safe</code> to the <code class="highlighter-rouge">field.help_text</code>: <code class="highlighter-rouge"><span class="p">{</span><span class="err">{</span><span class="w"> </span><span class="err">field.help_text|safe</span><span class="w"> </span><span class="p">}</span><span class="err">}</span></code>.</p>

<p>Save the <strong>form.html</strong> file, and check the sign up page again:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-5.png" alt="Sign up" /></p>

<p>Now let’s implement the business logic in the <strong>signup</strong> view:</p>

<p><strong>accounts/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">login</span> <span class="k">as</span> <span class="n">auth_login</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">UserCreationForm</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span><span class="p">,</span> <span class="n">redirect</span>

<span class="k">def</span> <span class="nf">signup</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">UserCreationForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">user</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="n">auth_login</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">user</span><span class="p">)</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">UserCreationForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'signup.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>A basic form processing with a small detail: the <strong>login</strong> function (renamed to <strong>auth_login</strong> to avoid clashing with
the built-in login view).</p>

<div class="info">
  <p>
    <strong><i class="fa fa-info-circle"></i> Note:</strong>
    I renamed the <code>login</code> function to <code>auth_login</code>, but later I realized that Django 1.11
    has a class-based view for the login view, <code>LoginView</code>, so there was no risk of clashing the names.
  </p>
  <p>
    On the older versions there was a <code>auth.login</code> and <code>auth.view.login</code>, which used to
    cause some confusion, because one was the function that logs the user in, and the other was the view.
  </p>
  <p>
    Long story short: you can import it just as <code>login</code> if you want, it will not cause any problem.
  </p>
</div>

<p>If the form is valid, a <strong>User</strong> instance is created with the <code class="highlighter-rouge">user = form.save()</code>. The created user is then passed
as an argument to the <strong>auth_login</strong> function, manually authenticating the user. After that, the view redirects the
user to the homepage, keeping the flow of the application.</p>

<p>Let’s try it. First, submit some invalid data. Either an empty form, non-matching fields, or an existing username:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-6.png" alt="Sign up" /></p>

<p>Now fill the form and submit it, check if the user is created and redirected to the homepage:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-7.png" alt="Sign up" /></p>

<h5 id="referencing-the-authenticated-user-in-the-template">Referencing the Authenticated User in the Template</h5>

<p>How can we know if it worked? Well, we can edit the <strong>base.html</strong> template to add the name of the user on the top bar:</p>

<p><strong>templates/base.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">block</span> <span class="nv">body</span> <span class="cp">%}</span>
  <span class="nt">&lt;nav</span> <span class="na">class=</span><span class="s">"navbar navbar-expand-sm navbar-dark bg-dark"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"navbar-brand"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Django Boards<span class="nt">&lt;/a&gt;</span>
      <span class="nt">&lt;button</span> <span class="na">class=</span><span class="s">"navbar-toggler"</span> <span class="na">type=</span><span class="s">"button"</span> <span class="na">data-toggle=</span><span class="s">"collapse"</span> <span class="na">data-target=</span><span class="s">"#mainMenu"</span> <span class="na">aria-controls=</span><span class="s">"mainMenu"</span> <span class="na">aria-expanded=</span><span class="s">"false"</span> <span class="na">aria-label=</span><span class="s">"Toggle navigation"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"navbar-toggler-icon"</span><span class="nt">&gt;&lt;/span&gt;</span>
      <span class="nt">&lt;/button&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"collapse navbar-collapse"</span> <span class="na">id=</span><span class="s">"mainMenu"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;ul</span> <span class="na">class=</span><span class="s">"navbar-nav ml-auto"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"nav-item"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"nav-link"</span> <span class="na">href=</span><span class="s">"#"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">user.username</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;/li&gt;</span>
        <span class="nt">&lt;/ul&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/nav&gt;</span>

  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;ol</span> <span class="na">class=</span><span class="s">"breadcrumb my-4"</span><span class="nt">&gt;</span>
      <span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
    <span class="nt">&lt;/ol&gt;</span>
    <span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="nv">body</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-8.png" alt="Sign up" /></p>

<h5 id="testing-the-sign-up-view">Testing the Sign Up View</h5>

<p>Let’s now improve our test cases:</p>

<p><strong>accounts/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">UserCreationForm</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">signup</span>

<span class="k">class</span> <span class="nc">SignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'signup'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_signup_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_signup_url_resolves_signup_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/signup/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="p">,</span> <span class="n">signup</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_csrf</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'csrfmiddlewaretoken'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_contains_form</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIsInstance</span><span class="p">(</span><span class="n">form</span><span class="p">,</span> <span class="n">UserCreationForm</span><span class="p">)</span></code></pre></figure>

<p>We changed a little bit the <strong>SignUpTests</strong> class. Defined a <strong>setUp</strong> method, moved the response object to there. Then
now we are also testing if there are a form and the CSRF token in the response.</p>

<p>Now we are going to test a successful sign up. This time, let’s create a new class to organize better the tests:</p>

<p><strong>accounts/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">UserCreationForm</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">signup</span>

<span class="k">class</span> <span class="nc">SignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># code suppressed...</span>

<span class="k">class</span> <span class="nc">SuccessfulSignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'signup'</span><span class="p">)</span>
        <span class="n">data</span> <span class="o">=</span> <span class="p">{</span>
            <span class="s">'username'</span><span class="p">:</span> <span class="s">'john'</span><span class="p">,</span>
            <span class="s">'password1'</span><span class="p">:</span> <span class="s">'abcdef123456'</span><span class="p">,</span>
            <span class="s">'password2'</span><span class="p">:</span> <span class="s">'abcdef123456'</span>
        <span class="p">}</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">data</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">home_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        A valid form submission should redirect the user to the home page
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">home_url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_user_creation</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">exists</span><span class="p">())</span>

    <span class="k">def</span> <span class="nf">test_user_authentication</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        Create a new request to an arbitrary page.
        The resulting response should now have a `user` to its context,
        after a successful sign up.
        '''</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">home_url</span><span class="p">)</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'user'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">user</span><span class="o">.</span><span class="n">is_authenticated</span><span class="p">)</span></code></pre></figure>

<p>Run the tests.</p>

<p>Using a similar strategy, now let’s create a new class for sign up tests when the data is invalid:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">UserCreationForm</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">signup</span>

<span class="k">class</span> <span class="nc">SignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># code suppressed...</span>

<span class="k">class</span> <span class="nc">SuccessfulSignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># code suppressed...</span>

<span class="k">class</span> <span class="nc">InvalidSignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'signup'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="p">{})</span>  <span class="c"># submit an empty dictionary</span>

    <span class="k">def</span> <span class="nf">test_signup_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        An invalid form submission should return to the same page
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_form_errors</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">form</span><span class="o">.</span><span class="n">errors</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_dont_create_user</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertFalse</span><span class="p">(</span><span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">exists</span><span class="p">())</span></code></pre></figure>

<h5 id="adding-the-email-field-to-the-form">Adding the Email Field to the Form</h5>

<p>Everything is working, but… The <strong>email address</strong> field is missing. Well, the <strong>UserCreationForm</strong> does not provide
an <strong>email</strong> field. But we can extend it.</p>

<p>Create a file named <strong>forms.py</strong> inside the <strong>accounts</strong> folder:</p>

<p><strong>accounts/forms.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">forms</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">UserCreationForm</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>

<span class="k">class</span> <span class="nc">SignUpForm</span><span class="p">(</span><span class="n">UserCreationForm</span><span class="p">):</span>
    <span class="n">email</span> <span class="o">=</span> <span class="n">forms</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">254</span><span class="p">,</span> <span class="n">required</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">widget</span><span class="o">=</span><span class="n">forms</span><span class="o">.</span><span class="n">EmailInput</span><span class="p">())</span>
    <span class="k">class</span> <span class="nc">Meta</span><span class="p">:</span>
        <span class="n">model</span> <span class="o">=</span> <span class="n">User</span>
        <span class="n">fields</span> <span class="o">=</span> <span class="p">(</span><span class="s">'username'</span><span class="p">,</span> <span class="s">'email'</span><span class="p">,</span> <span class="s">'password1'</span><span class="p">,</span> <span class="s">'password2'</span><span class="p">)</span></code></pre></figure>

<p>Now, instead of using the <strong>UserCreationForm</strong> in our <strong>views.py</strong>, let’s import the new form, <strong>SignUpForm</strong>, and use
it instead:</p>

<p><strong>accounts/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">login</span> <span class="k">as</span> <span class="n">auth_login</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span><span class="p">,</span> <span class="n">redirect</span>

<span class="kn">from</span> <span class="nn">.forms</span> <span class="kn">import</span> <span class="n">SignUpForm</span>

<span class="k">def</span> <span class="nf">signup</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">SignUpForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">user</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="n">auth_login</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">user</span><span class="p">)</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">SignUpForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'signup.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>Just with this small change, everything is already working:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-9.png" alt="Sign up" /></p>

<p>Remember to change the test case to use the <strong>SignUpForm</strong> instead of <strong>UserCreationForm</strong>:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">.forms</span> <span class="kn">import</span> <span class="n">SignUpForm</span>

<span class="k">class</span> <span class="nc">SignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">test_contains_form</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIsInstance</span><span class="p">(</span><span class="n">form</span><span class="p">,</span> <span class="n">SignUpForm</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">SuccessfulSignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'signup'</span><span class="p">)</span>
        <span class="n">data</span> <span class="o">=</span> <span class="p">{</span>
            <span class="s">'username'</span><span class="p">:</span> <span class="s">'john'</span><span class="p">,</span>
            <span class="s">'email'</span><span class="p">:</span> <span class="s">'john@doe.com'</span><span class="p">,</span>
            <span class="s">'password1'</span><span class="p">:</span> <span class="s">'abcdef123456'</span><span class="p">,</span>
            <span class="s">'password2'</span><span class="p">:</span> <span class="s">'abcdef123456'</span>
        <span class="p">}</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">data</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">home_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>

    <span class="c"># ...</span></code></pre></figure>

<p>The previous test case would still pass because since <strong>SignUpForm</strong> extends the <strong>UserCreationForm</strong>, <em>it is</em> an
instance of <strong>UserCreationForm</strong>.</p>

<p>Now let’s think about what happened for a moment. We added a new form field:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">fields</span> <span class="o">=</span> <span class="p">(</span><span class="s">'username'</span><span class="p">,</span> <span class="s">'email'</span><span class="p">,</span> <span class="s">'password1'</span><span class="p">,</span> <span class="s">'password2'</span><span class="p">)</span></code></pre></figure>

<p>And it automatically reflected in the HTML template. It’s good, right? Well, depends. What if in the future, a new
developer wanted to re-use the <strong>SignUpForm</strong> for something else, and add some extra fields to it. Then those new
fields would also show up in the <strong>signup.html</strong>, which may not be the desired behavior. This change could pass
unnoticed, and we don’t want any surprises.</p>

<p>So let’s create a new test, that verifies the HTML inputs in the template:</p>

<p><strong>accounts/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">SignUpTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">test_form_inputs</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        The view must contain five inputs: csrf, username, email,
        password1, password2
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'&lt;input'</span><span class="p">,</span> <span class="mi">5</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'type="text"'</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'type="email"'</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'type="password"'</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span></code></pre></figure>

<h5 id="improving-the-tests-layout">Improving the Tests Layout</h5>

<p>Alright, so we are testing the inputs and everything, but we still have to test the form itself. Instead of just keep
adding tests to the <strong>accounts/tests.py</strong> file, let’s improve the project design a little bit.</p>

<p>Create a new folder named <strong>tests</strong> within the <strong>accounts</strong> folder. Then, inside the <strong>tests</strong> folder, create an empty
file named <strong>__init__.py</strong>.</p>

<p>Now, move the <strong>tests.py</strong> file to inside the <strong>tests</strong> folder, and rename it to <strong>test_view_signup.py</strong>.</p>

<p>The final result should be the following:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |    |-- migrations/
 |    |    |-- tests/
 |    |    |    |-- __init__.py
 |    |    |    +-- test_view_signup.py
 |    |    |-- __init__.py
 |    |    |-- admin.py
 |    |    |-- apps.py
 |    |    |-- models.py
 |    |    +-- views.py
 |    |-- boards/
 |    |-- myproject/
 |    |-- static/
 |    |-- templates/
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>Note that since we are using relative import within the context of the apps, we need to fix the imports in the new
<strong>test_view_signup.py</strong>:</p>

<p><strong>accounts/tests/test_view_signup.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>

<span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">signup</span>
<span class="kn">from</span> <span class="nn">..forms</span> <span class="kn">import</span> <span class="n">SignUpForm</span></code></pre></figure>

<p>We are using relative imports inside the app modules so we can have the freedom to rename the Django app later on,
without having to fix all the absolute imports.</p>

<p>Now let’s create a new test file, to test the <strong>SignUpForm</strong>. Add a new test file named <strong>test_form_signup.py</strong>:</p>

<p><strong>accounts/tests/test_form_signup.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">..forms</span> <span class="kn">import</span> <span class="n">SignUpForm</span>

<span class="k">class</span> <span class="nc">SignUpFormTest</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_form_has_fields</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">SignUpForm</span><span class="p">()</span>
        <span class="n">expected</span> <span class="o">=</span> <span class="p">[</span><span class="s">'username'</span><span class="p">,</span> <span class="s">'email'</span><span class="p">,</span> <span class="s">'password1'</span><span class="p">,</span> <span class="s">'password2'</span><span class="p">,]</span>
        <span class="n">actual</span> <span class="o">=</span> <span class="nb">list</span><span class="p">(</span><span class="n">form</span><span class="o">.</span><span class="n">fields</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertSequenceEqual</span><span class="p">(</span><span class="n">expected</span><span class="p">,</span> <span class="n">actual</span><span class="p">)</span></code></pre></figure>

<p>It looks very strict, right? For example, if in the future we have to change the <strong>SignUpForm</strong>, to include the user’s
first and last name, we will probably end up having to fix a few test cases, even if we didn’t break anything.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Test_Alerts.png" alt="Test Case Alert" /></p>

<p>Those alerts are useful because they help to bring awareness, especially for newcomers touching the code for the first
time. It helps them code with confidence.</p>

<h5 id="improving-the-sign-up-template">Improving the Sign Up Template</h5>

<p>Let’s work a little bit on it. Here we can use Bootstrap 4 cards components to make it look good.</p>

<p>Go to <a href="https://www.toptal.com/designers/subtlepatterns/" target="_blank">https://www.toptal.com/designers/subtlepatterns/</a>
and find a nice background pattern to use as a background of the accounts pages. Download it, create a new folder
named <strong>img</strong> inside the <strong>static</strong> folder, and place the image there.</p>

<p>Then after that, create a new CSS file named <strong>accounts.css</strong> in the <strong>static/css</strong>. The result should be the
following:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |-- myproject/
 |    |-- static/
 |    |    |-- css/
 |    |    |    |-- accounts.css  &lt;-- here
 |    |    |    |-- app.css
 |    |    |    +-- bootstrap.min.css
 |    |    +-- img/
 |    |    |    +-- shattered.png  &lt;-- here <span class="o">(</span>the name may be different, depending on the patter you downloaded<span class="o">)</span>
 |    |-- templates/
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>Now edit the <strong>accounts.css</strong> file:</p>

<p><strong>static/css/accounts.css</strong></p>

<figure class="highlight"><pre><code class="language-css" data-lang="css"><span class="nt">body</span> <span class="p">{</span>
  <span class="nl">background-image</span><span class="p">:</span> <span class="sx">url(../img/shattered.png)</span><span class="p">;</span>
<span class="p">}</span>

<span class="nc">.logo</span> <span class="p">{</span>
  <span class="nl">font-family</span><span class="p">:</span> <span class="s2">'Peralta'</span><span class="p">,</span> <span class="nb">cursive</span><span class="p">;</span>
<span class="p">}</span>

<span class="nc">.logo</span> <span class="nt">a</span> <span class="p">{</span>
  <span class="nl">color</span><span class="p">:</span> <span class="n">rgba</span><span class="p">(</span><span class="m">0</span><span class="p">,</span><span class="m">0</span><span class="p">,</span><span class="m">0</span><span class="p">,</span><span class="m">.9</span><span class="p">);</span>
<span class="p">}</span>

<span class="nc">.logo</span> <span class="nt">a</span><span class="nd">:hover</span><span class="o">,</span>
<span class="nc">.logo</span> <span class="nt">a</span><span class="nd">:active</span> <span class="p">{</span>
  <span class="nl">text-decoration</span><span class="p">:</span> <span class="nb">none</span><span class="p">;</span>
<span class="p">}</span></code></pre></figure>

<p>In the <strong>signup.html</strong> template, we can change it to make use of the new CSS and also take the Bootstrap 4 card
components into use:</p>

<p><strong>templates/signup.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">stylesheet</span> <span class="cp">%}</span>
  <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/accounts.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">body</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;h1</span> <span class="na">class=</span><span class="s">"text-center logo my-4"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Django Boards<span class="nt">&lt;/a&gt;</span>
    <span class="nt">&lt;/h1&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row justify-content-center"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-8 col-md-10 col-sm-12"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Sign up<span class="nt">&lt;/h3&gt;</span>
            <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
              <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
              <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
              <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-primary btn-block"</span><span class="nt">&gt;</span>Create an account<span class="nt">&lt;/button&gt;</span>
            <span class="nt">&lt;/form&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-footer text-muted text-center"</span><span class="nt">&gt;</span>
            Already have an account? <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"#"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>With that, this should be our sign up page right now:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-10.jpg" alt="Sign up" /></p>

<hr />
