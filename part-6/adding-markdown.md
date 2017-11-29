<h4 id="adding-markdown">Adding Markdown</h4>

<p>Let’s improve the user experience by adding Markdown to our text areas. You will see it’s very easy and simple.</p>

<p>First, let’s install a library called <strong>Python-Markdown</strong>:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">pip install markdown</code></pre></figure>

<p>We can add a new method to the <strong>Post</strong> model:</p>

<p><strong>boards/models.py</strong>
<small><a href="https://gist.github.com/vitorfs/caa24fcf2b66903617ebbb41337d3d3d#file-models-py-L46" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.db</span> <span class="kn">import</span> <span class="n">models</span>
<span class="kn">from</span> <span class="nn">django.utils.html</span> <span class="kn">import</span> <span class="n">mark_safe</span>
<span class="kn">from</span> <span class="nn">markdown</span> <span class="kn">import</span> <span class="n">markdown</span>

<span class="k">class</span> <span class="nc">Post</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">get_message_as_markdown</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="n">mark_safe</span><span class="p">(</span><span class="n">markdown</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">message</span><span class="p">,</span> <span class="n">safe_mode</span><span class="o">=</span><span class="s">'escape'</span><span class="p">))</span></code></pre></figure>

<p>Here we are dealing with user input, so we must take care. When using the <code class="highlighter-rouge">markdown</code> function, we are instructing it
to escape the special characters first and then parse the markdown tags. After that, we mark the output string as safe
to be used in the template.</p>

<p>Now in the templates <strong>topic_posts.html</strong> and <strong>reply_topic.html</strong> just change from:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{{</span> <span class="nv">post.message</span> <span class="cp">}}</span></code></pre></figure>

<p>To:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{{</span> <span class="nv">post.get_message_as_markdown</span> <span class="cp">}}</span></code></pre></figure>

<p>From now on the users can already use markdown in the posts:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/markdown-1.png" alt="Markdown" /></p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/markdown-2.png" alt="Markdown" /></p>

<h5 id="markdown-editor">Markdown Editor</h5>

<p>We can also add a very cool Markdown editor called <a href="https://simplemde.com" target="_blank" rel="noopener">SimpleMD</a>.</p>

<p>Either download the JavaScript library or use their CDN:</p>

<figure class="highlight"><pre><code class="language-html" data-lang="html"><span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"https://cdn.jsdelivr.net/simplemde/latest/simplemde.min.css"</span><span class="nt">&gt;</span>
<span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://cdn.jsdelivr.net/simplemde/latest/simplemde.min.js"</span><span class="nt">&gt;&lt;/script&gt;</span></code></pre></figure>

<p>Now edit the <strong>base.html</strong> to make space for extra JavaScripts:</p>

<p><strong>templates/base.html</strong>
<small><a href="https://gist.github.com/vitorfs/5a7ad8e7eae88d64f62fec82d037b168#file-base-html-L57" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django">    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/jquery-3.2.1.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/popper.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/bootstrap.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="cp">{%</span> <span class="k">block</span> <span class="nv">javascript</span> <span class="cp">%}{%</span> <span class="k">endblock</span> <span class="cp">%}</span>  <span class="c">&lt;!-- Add this empty block here! --&gt;</span></code></pre></figure>

<p>First edit the <strong>reply_topic.html</strong> template:</p>

<p><strong>templates/reply_topic.html</strong>
<small><a href="https://gist.github.com/vitorfs/fb63bb7530690d62787c3ed8b7e15241" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Post a reply<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">stylesheet</span> <span class="cp">%}</span>
  <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/simplemde.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">javascript</span> <span class="cp">%}</span>
  <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/simplemde.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
  <span class="nt">&lt;script&gt;</span>
    <span class="kd">var</span> <span class="nx">simplemde</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">SimpleMDE</span><span class="p">();</span>
  <span class="nt">&lt;/script&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>By default, this plugin will transform the first text area it finds into a markdown editor. So just that code should
be enough:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/editor-1.png" alt="Editor" /></p>

<p>Now do the same thing with the <strong>edit_post.html</strong> template:</p>

<p><strong>templates/edit_post.html</strong>
<small><a href="https://gist.github.com/vitorfs/ee9d8c91888b0bc60013b8f037bae7bb" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Edit post<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">stylesheet</span> <span class="cp">%}</span>
  <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/simplemde.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">javascript</span> <span class="cp">%}</span>
  <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/simplemde.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
  <span class="nt">&lt;script&gt;</span>
    <span class="kd">var</span> <span class="nx">simplemde</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">SimpleMDE</span><span class="p">();</span>
  <span class="nt">&lt;/script&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/editor-2.png" alt="Editor" /></p>

<hr />
