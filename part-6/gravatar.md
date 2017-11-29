<h4 id="gravatar">Gravatar</h4>

<p>A very easy way to add user profile pictures is by using <a href="https://gravatar.com" target="noopener">Gravatar</a>.</p>

<p>Inside the <strong>boards/templatetags</strong> folder, create a new file named <strong>gravatar.py</strong>:</p>

<p><strong>boards/templatetags/gravatar.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">import</span> <span class="nn">hashlib</span>
<span class="kn">from</span> <span class="nn">urllib.parse</span> <span class="kn">import</span> <span class="n">urlencode</span>

<span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">template</span>
<span class="kn">from</span> <span class="nn">django.conf</span> <span class="kn">import</span> <span class="n">settings</span>

<span class="n">register</span> <span class="o">=</span> <span class="n">template</span><span class="o">.</span><span class="n">Library</span><span class="p">()</span>


<span class="nd">@register.filter</span>
<span class="k">def</span> <span class="nf">gravatar</span><span class="p">(</span><span class="n">user</span><span class="p">):</span>
    <span class="n">email</span> <span class="o">=</span> <span class="n">user</span><span class="o">.</span><span class="n">email</span><span class="o">.</span><span class="n">lower</span><span class="p">()</span><span class="o">.</span><span class="n">encode</span><span class="p">(</span><span class="s">'utf-8'</span><span class="p">)</span>
    <span class="n">default</span> <span class="o">=</span> <span class="s">'mm'</span>
    <span class="n">size</span> <span class="o">=</span> <span class="mi">256</span>
    <span class="n">url</span> <span class="o">=</span> <span class="s">'https://www.gravatar.com/avatar/{md5}?{params}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span>
        <span class="n">md5</span><span class="o">=</span><span class="n">hashlib</span><span class="o">.</span><span class="n">md5</span><span class="p">(</span><span class="n">email</span><span class="p">)</span><span class="o">.</span><span class="n">hexdigest</span><span class="p">(),</span>
        <span class="n">params</span><span class="o">=</span><span class="n">urlencode</span><span class="p">({</span><span class="s">'d'</span><span class="p">:</span> <span class="n">default</span><span class="p">,</span> <span class="s">'s'</span><span class="p">:</span> <span class="nb">str</span><span class="p">(</span><span class="n">size</span><span class="p">)})</span>
    <span class="p">)</span>
    <span class="k">return</span> <span class="n">url</span></code></pre></figure>

<p>Basically I’m using the <a href="https://fi.gravatar.com/site/implement/images/python/" target="_blank" rel="noopener">code snippet they provide</a>.
I just adapted it to work with Python 3.</p>

<p>Great, now we can load it in our template, just like we did with the Humanize template filter:</p>

<p><strong>templates/topic_posts.html</strong>
<small><a href="https://gist.github.com/vitorfs/23d5c5bc9e6c7ac94506a2660a61012c" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">gravatar</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="c">&lt;!-- code suppressed --&gt;</span>

  <span class="nt">&lt;img</span> <span class="na">src=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">post.created_by</span><span class="o">|</span><span class="nf">gravatar</span> <span class="cp">}}</span><span class="s">"</span> <span class="na">alt=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">post.created_by.username</span> <span class="cp">}}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"w-100 rounded"</span><span class="nt">&gt;</span>

  <span class="c">&lt;!-- code suppressed --&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/gravatar.png" alt="Gravatar" /></p>

<hr />
