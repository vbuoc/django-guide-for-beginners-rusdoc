<h4 id="humanize">Humanize</h4>

<p>I just thought it would be a nice touch to add the built-in <strong>humanize</strong> package. It’s a set of utility functions to
add a “human touch” to data.</p>

<p>For example, we can use it to display date and time fields more naturally. Instead of showing the whole date,
we can simply show: “2 minutes ago”.</p>

<p>Let’s do it. First, add the <code class="highlighter-rouge">django.contrib.humanize</code> to the <code class="highlighter-rouge">INSTALLED_APPS</code>.</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">INSTALLED_APPS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="s">'django.contrib.admin'</span><span class="p">,</span>
    <span class="s">'django.contrib.auth'</span><span class="p">,</span>
    <span class="s">'django.contrib.contenttypes'</span><span class="p">,</span>
    <span class="s">'django.contrib.sessions'</span><span class="p">,</span>
    <span class="s">'django.contrib.messages'</span><span class="p">,</span>
    <span class="s">'django.contrib.staticfiles'</span><span class="p">,</span>
    <span class="s">'django.contrib.humanize'</span><span class="p">,</span>  <span class="c"># &lt;- here</span>

    <span class="s">'widget_tweaks'</span><span class="p">,</span>

    <span class="s">'accounts'</span><span class="p">,</span>
    <span class="s">'boards'</span><span class="p">,</span>
<span class="p">]</span></code></pre></figure>

<p>Now we can use it in the templates. First, let’s edit the <strong>topics.html</strong> template:</p>

<p><strong>templates/topics.html</strong>
<small><a href="https://gist.github.com/vitorfs/45521a289677a1a406b4fb743e8141ee" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">humanize</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="c">&lt;!-- code suppressed --&gt;</span>

  <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.last_updated</span><span class="o">|</span><span class="nf">naturaltime</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>

  <span class="c">&lt;!-- code suppressed --&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>All we have to do is to load the template tags using <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">load</span><span class="w"> </span><span class="err">humanize</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> and then apply the
template filter: <code class="highlighter-rouge"><span class="p">{</span><span class="err">{</span><span class="w"> </span><span class="err">topic.last_updated|naturaltime</span><span class="w"> </span><span class="p">}</span><span class="err">}</span></code></p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/humanize.png" alt="Humanize" /></p>

<p>You may add it to other places you like.</p>

<hr />
