<h4 id="login">Login</h4>

<p>First thing, add a new URL route:</p>

<p><strong>myproject/urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">auth_views</span>

<span class="kn">from</span> <span class="nn">accounts</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">accounts_views</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^signup/$'</span><span class="p">,</span> <span class="n">accounts_views</span><span class="o">.</span><span class="n">signup</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'signup'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^login/$'</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">LoginView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(</span><span class="n">template_name</span><span class="o">=</span><span class="s">'login.html'</span><span class="p">),</span> <span class="n">name</span><span class="o">=</span><span class="s">'login'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^logout/$'</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">LogoutView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'logout'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/new/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">new_topic</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'new_topic'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^admin/'</span><span class="p">,</span> <span class="n">admin</span><span class="o">.</span><span class="n">site</span><span class="o">.</span><span class="n">urls</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Inside the <code class="highlighter-rouge">as_view()</code> we can pass some extra parameters, so to override the defaults. In this case, we are instructing
the <strong>LoginView</strong> to look for a template at <strong>login.html</strong>.</p>

<p>Edit the <strong>settings.py</strong> and add the following configuration:</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">LOGIN_REDIRECT_URL</span> <span class="o">=</span> <span class="s">'home'</span></code></pre></figure>

<p>This configuration is telling Django where to redirect the user after a successful login.</p>

<p>Finally, add the login URL to the <strong>base.html</strong> template:</p>

<p><strong>templates/base.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'login'</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-outline-secondary"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/a&gt;</span></code></pre></figure>

<p>We can create a template similar to the sign up page. Create a new file named <strong>login.html</strong>:</p>

<p><strong>templates/login.html</strong></p>

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
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-4 col-md-6 col-sm-8"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/h3&gt;</span>
            <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
              <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
              <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
              <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-primary btn-block"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/button&gt;</span>
            <span class="nt">&lt;/form&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-footer text-muted text-center"</span><span class="nt">&gt;</span>
            New to Django Boards? <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'signup'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Sign up<span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"text-center py-2"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;small&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"#"</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span>Forgot your password?<span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;/small&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login.jpg" alt="Login" /></p>

<p>And we are repeating HTML templates. Let’s refactor it.</p>

<p>Create a new template named <strong>base_accounts.html</strong>:</p>

<p><strong>templates/base_accounts.html</strong></p>

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
    <span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Now use it on both <strong>signup.html</strong> and <strong>login.html</strong>:</p>

<p><strong>templates/login.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base_accounts.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Log in to Django Boards<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row justify-content-center"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-4 col-md-6 col-sm-8"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/h3&gt;</span>
          <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
            <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
            <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
            <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-primary btn-block"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/button&gt;</span>
          <span class="nt">&lt;/form&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-footer text-muted text-center"</span><span class="nt">&gt;</span>
          New to Django Boards? <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'signup'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Sign up<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"text-center py-2"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;small&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"#"</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span>Forgot your password?<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/small&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>We still don’t have the password reset URL, so let’s leave it as <code class="highlighter-rouge">#</code> for now.</p>

<p><strong>templates/signup.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base_accounts.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Sign up to Django Boards<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
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
          Already have an account? <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'login'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Notice that we added the log in URL: <code class="highlighter-rouge">&lt;a href="{% url 'login' %}"&gt;Log in&lt;/a&gt;</code>.</p>

<h5 id="log-in-non-field-errors">Log In Non Field Errors</h5>

<p>If we submit the log in form empty, we get some nice error messages:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-1.jpg" alt="Login" /></p>

<p>But if we submit an username that doesn’t exist or an invalid password, right now that’s what’s going to happen:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-2.jpg" alt="Login" /></p>

<p>A little bit misleading. The fields are showing green, suggesting they are okay. Also, there’s no message saying
anything.</p>

<p>That’s because forms have a special type of error, which is called <strong>non-field errors</strong>. It’s a collection of errors
that are not related to a specific field. Let’s refactor the <strong>form.html</strong> partial template to display those errors as
well:</p>

<p><strong>templates/includes/form.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">widget_tweaks</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">if</span> <span class="nv">form.non_field_errors</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"alert alert-danger"</span> <span class="na">role=</span><span class="s">"alert"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="k">for</span> <span class="nv">error</span> <span class="ow">in</span> <span class="nv">form.non_field_errors</span> <span class="cp">%}</span>
      <span class="nt">&lt;p</span><span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.last</span> <span class="cp">%}</span> <span class="na">class=</span><span class="s">"mb-0"</span><span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">error</span> <span class="cp">}}</span><span class="nt">&lt;/p&gt;</span>
    <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">for</span> <span class="nv">field</span> <span class="ow">in</span> <span class="nv">form</span> <span class="cp">%}</span>
  <span class="c">&lt;!-- code suppressed --&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<p>The <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">if</span><span class="w"> </span><span class="err">forloop.last</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> is just a minor thing. Because the <code class="highlighter-rouge">p</code> tag has a <code class="highlighter-rouge">margin-bottom</code>.
And a form may have several non-field errors. For each non-field error, we render a <code class="highlighter-rouge">p</code> tag with the error. Then
I’m checking if it’s the last error to render. If so, we add a Bootstrap 4 CSS class <code class="highlighter-rouge">mb-0</code> which stands for
“margin bottom = 0”. Then the alert doesn’t look weird, with some extra space. Again, just a very minor detail. I did
that just to keep the consistency of the spacing.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-3.jpg" alt="Login" /></p>

<p>We still have to deal with the password field though. The thing is, Django never returned the data of password fields
to the client. So, instead of trying to do something smart, let’s just ignore the <code class="highlighter-rouge">is-valid</code> and <code class="highlighter-rouge">is-invalid</code> CSS
classes in some cases. But our form template already looks complicated. We can move some of the code to a
<strong>template tag</strong>.</p>

<h5 id="creating-custom-template-tags">Creating Custom Template Tags</h5>

<p>Inside the <strong>boards</strong> app, create a new folder named <strong>templatetags</strong>. Then inside this folder, create two empty files
named <strong>__init__.py</strong> and <strong>form_tags.py</strong>.</p>

<p>The structure should be the following:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |    |-- migrations/
 |    |    |-- templatetags/        &lt;-- here
 |    |    |    |-- __init__.py
 |    |    |    +-- form_tags.py
 |    |    |-- __init__.py
 |    |    |-- admin.py
 |    |    |-- apps.py
 |    |    |-- models.py
 |    |    |-- tests.py
 |    |    +-- views.py
 |    |-- myproject/
 |    |-- static/
 |    |-- templates/
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>In the <strong>form_tags.py</strong> file, let’s create two template tags:</p>

<p><strong>boards/templatetags/form_tags.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">template</span>

<span class="n">register</span> <span class="o">=</span> <span class="n">template</span><span class="o">.</span><span class="n">Library</span><span class="p">()</span>

<span class="nd">@register.filter</span>
<span class="k">def</span> <span class="nf">field_type</span><span class="p">(</span><span class="n">bound_field</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">bound_field</span><span class="o">.</span><span class="n">field</span><span class="o">.</span><span class="n">widget</span><span class="o">.</span><span class="n">__class__</span><span class="o">.</span><span class="n">__name__</span>

<span class="nd">@register.filter</span>
<span class="k">def</span> <span class="nf">input_class</span><span class="p">(</span><span class="n">bound_field</span><span class="p">):</span>
    <span class="n">css_class</span> <span class="o">=</span> <span class="s">''</span>
    <span class="k">if</span> <span class="n">bound_field</span><span class="o">.</span><span class="n">form</span><span class="o">.</span><span class="n">is_bound</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">bound_field</span><span class="o">.</span><span class="n">errors</span><span class="p">:</span>
            <span class="n">css_class</span> <span class="o">=</span> <span class="s">'is-invalid'</span>
        <span class="k">elif</span> <span class="n">field_type</span><span class="p">(</span><span class="n">bound_field</span><span class="p">)</span> <span class="o">!=</span> <span class="s">'PasswordInput'</span><span class="p">:</span>
            <span class="n">css_class</span> <span class="o">=</span> <span class="s">'is-valid'</span>
    <span class="k">return</span> <span class="s">'form-control {}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">css_class</span><span class="p">)</span></code></pre></figure>

<p>Those are <em>template filters</em>. They work like this:</p>

<p>First, we load it in a template as we do with the <strong>widget_tweaks</strong> or <strong>static</strong> template tags. Note that after
creating this file, you will have to manually stop the development server and start it again so Django can identify
the new template tags.</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">form_tags</span> <span class="cp">%}</span></code></pre></figure>

<p>Then after that, we can use them in a template:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{{</span> <span class="nv">form.username</span><span class="o">|</span><span class="nf">field_type</span> <span class="cp">}}</span></code></pre></figure>

<p>Will return:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django">'TextInput'</code></pre></figure>

<p>Or in case of the <strong>input_class</strong>:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{{</span> <span class="nv">form.username</span><span class="o">|</span><span class="nf">input_class</span> <span class="cp">}}</span>

<span class="c">&lt;!-- if the form is not bound, it will simply return: --&gt;</span>
'form-control '

<span class="c">&lt;!-- if the form is bound and valid: --&gt;</span>
'form-control is-valid'

<span class="c">&lt;!-- if the form is bound and invalid: --&gt;</span>
'form-control is-invalid'</code></pre></figure>

<p>Now update the <strong>form.html</strong> to use the new template tags:</p>

<p><strong>templates/includes/form.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">form_tags</span> <span class="nv">widget_tweaks</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">if</span> <span class="nv">form.non_field_errors</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"alert alert-danger"</span> <span class="na">role=</span><span class="s">"alert"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="k">for</span> <span class="nv">error</span> <span class="ow">in</span> <span class="nv">form.non_field_errors</span> <span class="cp">%}</span>
      <span class="nt">&lt;p</span><span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.last</span> <span class="cp">%}</span> <span class="na">class=</span><span class="s">"mb-0"</span><span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">error</span> <span class="cp">}}</span><span class="nt">&lt;/p&gt;</span>
    <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">for</span> <span class="nv">field</span> <span class="ow">in</span> <span class="nv">form</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"form-group"</span><span class="nt">&gt;</span>
    <span class="cp">{{</span> <span class="nv">field.label_tag</span> <span class="cp">}}</span>
    <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="nv">field</span><span class="o">|</span><span class="nf">input_class</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">for</span> <span class="nv">error</span> <span class="ow">in</span> <span class="nv">field.errors</span> <span class="cp">%}</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"invalid-feedback"</span><span class="nt">&gt;</span>
        <span class="cp">{{</span> <span class="nv">error</span> <span class="cp">}}</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">if</span> <span class="nv">field.help_text</span> <span class="cp">%}</span>
      <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"form-text text-muted"</span><span class="nt">&gt;</span>
        <span class="cp">{{</span> <span class="nv">field.help_text</span><span class="o">|</span><span class="nf">safe</span> <span class="cp">}}</span>
      <span class="nt">&lt;/small&gt;</span>
    <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<p>Much better, right? Reduced the complexity of the template. It looks cleaner now. And it also solved the problem with
the password field displaying a green border:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-4.jpg" alt="Login" /></p>

<h5 id="testing-the-template-tags">Testing the Template Tags</h5>

<p>First, let’s just organize the <strong>boards’</strong> tests a little bit. Like we did with the <strong>accounts</strong> app, create a new folder
named <strong>tests</strong>, add a <strong>__init__.py</strong>, copy the <strong>tests.py</strong> and rename it to just <strong>test_views.py</strong> for now.</p>

<p>Add a new empty file named <strong>test_templatetags.py</strong>.</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |    |-- migrations/
 |    |    |-- templatetags/
 |    |    |-- tests/
 |    |    |    |-- __init__.py
 |    |    |    |-- test_templatetags.py  &lt;-- new file, empty <span class="k">for </span>now
 |    |    |    +-- test_views.py  &lt;-- our old file with all the tests
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

<p>Fix the <strong>test_views.py</strong> imports:</p>

<p><strong>boards/tests/test_views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">home</span><span class="p">,</span> <span class="n">board_topics</span><span class="p">,</span> <span class="n">new_topic</span>
<span class="kn">from</span> <span class="nn">..models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Topic</span><span class="p">,</span> <span class="n">Post</span>
<span class="kn">from</span> <span class="nn">..forms</span> <span class="kn">import</span> <span class="n">NewTopicForm</span></code></pre></figure>

<p>Execute the tests just to make sure everything is working.</p>

<p><strong>boards/tests/test_templatetags.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">forms</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">..templatetags.form_tags</span> <span class="kn">import</span> <span class="n">field_type</span><span class="p">,</span> <span class="n">input_class</span>

<span class="k">class</span> <span class="nc">ExampleForm</span><span class="p">(</span><span class="n">forms</span><span class="o">.</span><span class="n">Form</span><span class="p">):</span>
    <span class="n">name</span> <span class="o">=</span> <span class="n">forms</span><span class="o">.</span><span class="n">CharField</span><span class="p">()</span>
    <span class="n">password</span> <span class="o">=</span> <span class="n">forms</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">widget</span><span class="o">=</span><span class="n">forms</span><span class="o">.</span><span class="n">PasswordInput</span><span class="p">())</span>
    <span class="k">class</span> <span class="nc">Meta</span><span class="p">:</span>
        <span class="n">fields</span> <span class="o">=</span> <span class="p">(</span><span class="s">'name'</span><span class="p">,</span> <span class="s">'password'</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">FieldTypeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_field_widget_type</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">ExampleForm</span><span class="p">()</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="s">'TextInput'</span><span class="p">,</span> <span class="n">field_type</span><span class="p">(</span><span class="n">form</span><span class="p">[</span><span class="s">'name'</span><span class="p">]))</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="s">'PasswordInput'</span><span class="p">,</span> <span class="n">field_type</span><span class="p">(</span><span class="n">form</span><span class="p">[</span><span class="s">'password'</span><span class="p">]))</span>

<span class="k">class</span> <span class="nc">InputClassTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_unbound_field_initial_state</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">ExampleForm</span><span class="p">()</span>  <span class="c"># unbound form</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="s">'form-control '</span><span class="p">,</span> <span class="n">input_class</span><span class="p">(</span><span class="n">form</span><span class="p">[</span><span class="s">'name'</span><span class="p">]))</span>

    <span class="k">def</span> <span class="nf">test_valid_bound_field</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">ExampleForm</span><span class="p">({</span><span class="s">'name'</span><span class="p">:</span> <span class="s">'john'</span><span class="p">,</span> <span class="s">'password'</span><span class="p">:</span> <span class="s">'123'</span><span class="p">})</span>  <span class="c"># bound form (field + data)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="s">'form-control is-valid'</span><span class="p">,</span> <span class="n">input_class</span><span class="p">(</span><span class="n">form</span><span class="p">[</span><span class="s">'name'</span><span class="p">]))</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="s">'form-control '</span><span class="p">,</span> <span class="n">input_class</span><span class="p">(</span><span class="n">form</span><span class="p">[</span><span class="s">'password'</span><span class="p">]))</span>

    <span class="k">def</span> <span class="nf">test_invalid_bound_field</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">ExampleForm</span><span class="p">({</span><span class="s">'name'</span><span class="p">:</span> <span class="s">''</span><span class="p">,</span> <span class="s">'password'</span><span class="p">:</span> <span class="s">'123'</span><span class="p">})</span>  <span class="c"># bound form (field + data)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="s">'form-control is-invalid'</span><span class="p">,</span> <span class="n">input_class</span><span class="p">(</span><span class="n">form</span><span class="p">[</span><span class="s">'name'</span><span class="p">]))</span></code></pre></figure>

<p>We created a form class to be used in the tests then added test cases covering the possible scenarios in the two
template tags.</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
................................
----------------------------------------------------------------------
Ran 32 tests in 0.846s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<hr />

