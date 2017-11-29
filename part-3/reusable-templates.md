
<p>Until now we’ve been copying and pasting HTML repeating several parts of the HTML document, which is not very
sustainable in the long run. It’s also a bad practice.</p>

<p>In this section we are going to refactor our HTML templates, creating a <strong>master page</strong> and only adding the unique part
for each template.</p>

<p>Create a new file named <strong>base.html</strong> in the <strong>templates</strong> folder:</p>

<p><strong>templates/base.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span><span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Django Boards<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span><span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;ol</span> <span class="na">class=</span><span class="s">"breadcrumb my-4"</span><span class="nt">&gt;</span>
        <span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
      <span class="nt">&lt;/ol&gt;</span>
      <span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>This is going to be our master page. Every template we create, is going to <strong>extend</strong> this special template. Observe
now we introduced the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> tag. It is used to reserve a space in the template, which
a “child” template (which extends the master page) can insert code and HTML within that space.</p>

<p>In the case of the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">title</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> we are also setting a default value, which is
“Django Boards.” It will be used if we don’t set a value for the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">title</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> in a child
template.</p>

<p>Now let’s refactor our two templates: <strong>home.html</strong> and <strong>topics.html</strong>.</p>

<p><strong>templates/home.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
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
            <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span>
            <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"text-muted d-block"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.description</span> <span class="cp">}}</span><span class="nt">&lt;/small&gt;</span>
          <span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>0<span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>0<span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td&gt;&lt;/td&gt;</span>
        <span class="nt">&lt;/tr&gt;</span>
      <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
    <span class="nt">&lt;/tbody&gt;</span>
  <span class="nt">&lt;/table&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>The first line in the <strong>home.html</strong> template is <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">extends</span><span class="w"> </span><span class="err">'base.html'</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>. This tag is telling
Django to use the <strong>base.html</strong> template as a master page. After that, we are using the the <em>blocks</em> to put the unique
content of the page.</p>

<p><strong>templates/topics.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>
  <span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span> - <span class="cp">{{</span> <span class="nv">block.super</span> <span class="cp">}}</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
    <span class="c">&lt;!-- just leaving it empty for now. we will add core here soon. --&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>In the <strong>topics.html</strong> template, we are changing the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">block</span><span class="w"> </span><span class="err">title</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> default value. Notice
that we can reuse the default value of the block by calling <code class="highlighter-rouge"><span class="p">{</span><span class="err">{</span><span class="w"> </span><span class="err">block.super</span><span class="w"> </span><span class="p">}</span><span class="err">}</span></code>. So here we are
playing with the website title, which we defined in the <strong>base.html</strong> as “Django Boards.” So for the “Python” board
page, the title will be “Python - Django Boards,” for the “Random” board the title will be “Random - Django Boards.”</p>

<p>Now let’s run the tests and see we didn’t break anything:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.......
----------------------------------------------------------------------
Ran 7 tests in 0.067s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Great! Everything is looking good.</p>

<p>Now that we have the <strong>base.html</strong> template, we can easily add a top bar with a menu:</p>

<p><strong>templates/base.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span><span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Django Boards<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span><span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>

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
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/django-boards-header-1.png" alt="Django Boards Header" /></p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/django-boards-header-2.png" alt="Django Boards Header" /></p>

<p>The HTML I used is part of the <a href="https://getbootstrap.com/docs/4.0/components/navbar/" target="_blank">Bootstrap 4 Navbar Component</a>.</p>

<p>A nice touch I like to add is to change the font in the “logo” (<code class="highlighter-rouge">.navbar-brand</code>) of the page.</p>

<p>Go to <a href="https://fonts.google.com/" target="_blank">fonts.google.com</a>, type “Django Boards” or whatever name you gave
to your project then click on <strong>apply to all fonts</strong>. Browse a bit, find one that you like.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/google-fonts.png" alt="Google Fonts" /></p>

<p>Add the font in the <strong>base.html</strong> template:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span><span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Django Boards<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span><span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">href=</span><span class="s">"https://fonts.googleapis.com/css?family=Peralta"</span> <span class="na">rel=</span><span class="s">"stylesheet"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/app.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="c">&lt;!-- code suppressed for brevity --&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>Now create a new CSS file named <strong>app.css</strong> inside the <strong>static/css</strong> folder:</p>

<p><strong>static/css/app.css</strong></p>

<figure class="highlight"><pre><code class="language-css" data-lang="css"><span class="nc">.navbar-brand</span> <span class="p">{</span>
  <span class="nl">font-family</span><span class="p">:</span> <span class="s2">'Peralta'</span><span class="p">,</span> <span class="nb">cursive</span><span class="p">;</span>
<span class="p">}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/boards-logo.png" alt="Django Boards Logo" /></p>

<hr />
