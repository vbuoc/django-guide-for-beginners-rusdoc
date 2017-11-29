<h4 id="final-adjustments">Final Adjustments</h4>

<p>Maybe you have already noticed, but there’s a small issue when someone replies to a post. It’s not updating the
<code class="highlighter-rouge">last_update</code> field, so the ordering of the topics is broken right now.</p>

<p>Let’s fix it:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">reply_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="p">):</span>
    <span class="n">topic</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">board__pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">topic_pk</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">post</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
            <span class="n">post</span><span class="o">.</span><span class="n">topic</span> <span class="o">=</span> <span class="n">topic</span>
            <span class="n">post</span><span class="o">.</span><span class="n">created_by</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">user</span>
            <span class="n">post</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>

            <span class="n">topic</span><span class="o">.</span><span class="n">last_updated</span> <span class="o">=</span> <span class="n">timezone</span><span class="o">.</span><span class="n">now</span><span class="p">()</span>  <span class="c"># &lt;- here</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>                         <span class="c"># &lt;- and here</span>

            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="o">=</span><span class="n">topic_pk</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'reply_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'topic'</span><span class="p">:</span> <span class="n">topic</span><span class="p">,</span> <span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>Next thing we want to do is try to control the view counting system a little bit more. We don’t want to the same user
refreshing the page counting as multiple views. For this we can use sessions:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">PostListView</span><span class="p">(</span><span class="n">ListView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Post</span>
    <span class="n">context_object_name</span> <span class="o">=</span> <span class="s">'posts'</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'topic_posts.html'</span>
    <span class="n">paginate_by</span> <span class="o">=</span> <span class="mi">20</span>

    <span class="k">def</span> <span class="nf">get_context_data</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>

        <span class="n">session_key</span> <span class="o">=</span> <span class="s">'viewed_topic_{}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span>  <span class="c"># &lt;-- here</span>
        <span class="k">if</span> <span class="ow">not</span> <span class="bp">self</span><span class="o">.</span><span class="n">request</span><span class="o">.</span><span class="n">session</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">session_key</span><span class="p">,</span> <span class="bp">False</span><span class="p">):</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">views</span> <span class="o">+=</span> <span class="mi">1</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">request</span><span class="o">.</span><span class="n">session</span><span class="p">[</span><span class="n">session_key</span><span class="p">]</span> <span class="o">=</span> <span class="bp">True</span>           <span class="c"># &lt;-- until here</span>

        <span class="n">kwargs</span><span class="p">[</span><span class="s">'topic'</span><span class="p">]</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">topic</span>
        <span class="k">return</span> <span class="nb">super</span><span class="p">()</span><span class="o">.</span><span class="n">get_context_data</span><span class="p">(</span><span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get_queryset</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">topic</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">board__pk</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">kwargs</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'pk'</span><span class="p">),</span> <span class="n">pk</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">kwargs</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'topic_pk'</span><span class="p">))</span>
        <span class="n">queryset</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">posts</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'created_at'</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">queryset</span></code></pre></figure>

<p>Now we could provide a better navigation in the topics listing. Currently the only option is for the user to click
in the topic title and go to the first page. We could workout something like this:</p>

<p><strong>boards/models.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">import</span> <span class="nn">math</span>
<span class="kn">from</span> <span class="nn">django.db</span> <span class="kn">import</span> <span class="n">models</span>

<span class="k">class</span> <span class="nc">Topic</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">subject</span>

    <span class="k">def</span> <span class="nf">get_page_count</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">count</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">posts</span><span class="o">.</span><span class="n">count</span><span class="p">()</span>
        <span class="n">pages</span> <span class="o">=</span> <span class="n">count</span> <span class="o">/</span> <span class="mi">20</span>
        <span class="k">return</span> <span class="n">math</span><span class="o">.</span><span class="n">ceil</span><span class="p">(</span><span class="n">pages</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">has_many_pages</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">count</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">count</span> <span class="ow">is</span> <span class="bp">None</span><span class="p">:</span>
            <span class="n">count</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">get_page_count</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">count</span> <span class="o">&gt;</span> <span class="mi">6</span>

    <span class="k">def</span> <span class="nf">get_page_range</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">count</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">get_page_count</span><span class="p">()</span>
        <span class="k">if</span> <span class="bp">self</span><span class="o">.</span><span class="n">has_many_pages</span><span class="p">(</span><span class="n">count</span><span class="p">):</span>
            <span class="k">return</span> <span class="nb">range</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">5</span><span class="p">)</span>
        <span class="k">return</span> <span class="nb">range</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="n">count</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span></code></pre></figure>

<p>Then in the <strong>topics.html</strong> template we could implement something like this:</p>

<p><strong>templates/topics.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django">  <span class="nt">&lt;table</span> <span class="na">class=</span><span class="s">"table table-striped mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;thead</span> <span class="na">class=</span><span class="s">"thead-inverse"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;tr&gt;</span>
        <span class="nt">&lt;th&gt;</span>Topic<span class="nt">&lt;/th&gt;</span>
        <span class="nt">&lt;th&gt;</span>Starter<span class="nt">&lt;/th&gt;</span>
        <span class="nt">&lt;th&gt;</span>Replies<span class="nt">&lt;/th&gt;</span>
        <span class="nt">&lt;th&gt;</span>Views<span class="nt">&lt;/th&gt;</span>
        <span class="nt">&lt;th&gt;</span>Last Update<span class="nt">&lt;/th&gt;</span>
      <span class="nt">&lt;/tr&gt;</span>
    <span class="nt">&lt;/thead&gt;</span>
    <span class="nt">&lt;tbody&gt;</span>
      <span class="cp">{%</span> <span class="k">for</span> <span class="nv">topic</span> <span class="ow">in</span> <span class="nv">topics</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">board.pk</span> <span class="nv">topic.pk</span> <span class="k">as</span> <span class="nv">topic_url</span> <span class="cp">%}</span>
        <span class="nt">&lt;tr&gt;</span>
          <span class="nt">&lt;td&gt;</span>
            <span class="nt">&lt;p</span> <span class="na">class=</span><span class="s">"mb-0"</span><span class="nt">&gt;</span>
              <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">topic_url</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span>
            <span class="nt">&lt;/p&gt;</span>
            <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span>
              Pages:
              <span class="cp">{%</span> <span class="k">for</span> <span class="nv">i</span> <span class="ow">in</span> <span class="nv">topic.get_page_range</span> <span class="cp">%}</span>
                <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">topic_url</span> <span class="cp">}}</span><span class="s">?page=</span><span class="cp">{{</span> <span class="nv">i</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">i</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span>
              <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
              <span class="cp">{%</span> <span class="k">if</span> <span class="nv">topic.has_many_pages</span> <span class="cp">%}</span>
              ... <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">topic_url</span> <span class="cp">}}</span><span class="s">?page=</span><span class="cp">{{</span> <span class="nv">topic.get_page_count</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span>Last Page<span class="nt">&lt;/a&gt;</span>
              <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
            <span class="nt">&lt;/small&gt;</span>
          <span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.starter.username</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.replies</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.views</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.last_updated</span><span class="o">|</span><span class="nf">naturaltime</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
        <span class="nt">&lt;/tr&gt;</span>
      <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
    <span class="nt">&lt;/tbody&gt;</span>
  <span class="nt">&lt;/table&gt;</span></code></pre></figure>

<p>Like a tiny pagination for each topic. Note that I also took the time to add the <code class="highlighter-rouge">table-striped</code> class for a better
styling of the table.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/pages.png" alt="Pages" /></p>

<p>In the reply page, we are currently listing all topic replies. We could limit it to just the last ten posts.</p>

<p><strong>boards/models.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">Topic</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">get_last_ten_posts</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">posts</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-created_at'</span><span class="p">)[:</span><span class="mi">10</span><span class="p">]</span></code></pre></figure>

<p><strong>templates/reply_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>

  <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">class=</span><span class="s">"mb-4"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
    <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Post a reply<span class="nt">&lt;/button&gt;</span>
  <span class="nt">&lt;/form&gt;</span>

  <span class="cp">{%</span> <span class="k">for</span> <span class="nv">post</span> <span class="ow">in</span> <span class="nv">topic.get_last_ten_posts</span> <span class="cp">%}</span>  <span class="c">&lt;!-- here! --&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card mb-2"</span><span class="nt">&gt;</span>
      <span class="c">&lt;!-- code suppressed --&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/replies.png" alt="Replies" /></p>

<p>Another thing is that when the user replies to a post, we are redirecting the user to the first page again. We could
improve it by sending the user to the last page.</p>

<p>We can add an id to the post card:</p>

<p><strong>templates/topic_posts.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>

  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'reply_topic'</span> <span class="nv">topic.board.pk</span> <span class="nv">topic.pk</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span> <span class="na">role=</span><span class="s">"button"</span><span class="nt">&gt;</span>Reply<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/div&gt;</span>

  <span class="cp">{%</span> <span class="k">for</span> <span class="nv">post</span> <span class="ow">in</span> <span class="nv">posts</span> <span class="cp">%}</span>
    <span class="nt">&lt;div</span> <span class="na">id=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">post.pk</span> <span class="cp">}}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"card </span><span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.last</span> <span class="cp">%}</span><span class="s">mb-4</span><span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span><span class="s">mb-2</span><span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span><span class="s"> </span><span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.first</span> <span class="cp">%}</span><span class="s">border-dark</span><span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
      <span class="c">&lt;!-- code suppressed --&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

  <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/pagination.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>The important bit here is the <code class="highlighter-rouge">&lt;div id="{{ post.pk }}" ...&gt;</code>.</p>

<p>Then we can play with it like this in the views:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">reply_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="p">):</span>
    <span class="n">topic</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">board__pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">topic_pk</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">post</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
            <span class="n">post</span><span class="o">.</span><span class="n">topic</span> <span class="o">=</span> <span class="n">topic</span>
            <span class="n">post</span><span class="o">.</span><span class="n">created_by</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">user</span>
            <span class="n">post</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>

            <span class="n">topic</span><span class="o">.</span><span class="n">last_updated</span> <span class="o">=</span> <span class="n">timezone</span><span class="o">.</span><span class="n">now</span><span class="p">()</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>

            <span class="n">topic_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="n">pk</span><span class="p">,</span> <span class="s">'topic_pk'</span><span class="p">:</span> <span class="n">topic_pk</span><span class="p">})</span>
            <span class="n">topic_post_url</span> <span class="o">=</span> <span class="s">'{url}?page={page}#{id}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span>
                <span class="n">url</span><span class="o">=</span><span class="n">topic_url</span><span class="p">,</span>
                <span class="nb">id</span><span class="o">=</span><span class="n">post</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span>
                <span class="n">page</span><span class="o">=</span><span class="n">topic</span><span class="o">.</span><span class="n">get_page_count</span><span class="p">()</span>
            <span class="p">)</span>

            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="n">topic_post_url</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'reply_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'topic'</span><span class="p">:</span> <span class="n">topic</span><span class="p">,</span> <span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>In the <strong>topic_post_url</strong> we are building a URL with the last page and adding an anchor to the element with id equals
to the post ID.</p>

<p>With this, it will required us to update the following test case:</p>

<p><strong>boards/tests/test_view_reply_topic.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">SuccessfulReplyTopicTests</span><span class="p">(</span><span class="n">ReplyTopicTestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        A valid form submission should redirect the user
        '''</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span> <span class="s">'topic_pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">})</span>
        <span class="n">topic_posts_url</span> <span class="o">=</span> <span class="s">'{url}?page=1#2'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">url</span><span class="o">=</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="n">topic_posts_url</span><span class="p">)</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/reply-last-page.png" alt="Replies" /></p>

<p>Next issue, as you can see in the previous screenshot, is to solve the problem with the pagination when the number
of pages is too high.</p>

<p>The easiest way is to tweak the <strong>pagination.html</strong> template:</p>

<p><strong>templates/includes/pagination.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">if</span> <span class="nv">is_paginated</span> <span class="cp">%}</span>
  <span class="nt">&lt;nav</span> <span class="na">aria-label=</span><span class="s">"Topics pagination"</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;ul</span> <span class="na">class=</span><span class="s">"pagination"</span><span class="nt">&gt;</span>
      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">page_obj.</span><span class="nb">number</span> <span class="o">&gt;</span> <span class="nv">1</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=1"</span><span class="nt">&gt;</span>First<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item disabled"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>First<span class="nt">&lt;/span&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">page_obj.has_previous</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=</span><span class="cp">{{</span> <span class="nv">page_obj.previous_page_number</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span>Previous<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item disabled"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>Previous<span class="nt">&lt;/span&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

      <span class="cp">{%</span> <span class="k">for</span> <span class="nv">page_num</span> <span class="ow">in</span> <span class="nv">paginator.page_range</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">if</span> <span class="nv">page_obj.</span><span class="nb">number</span> <span class="o">==</span> <span class="nv">page_num</span> <span class="cp">%}</span>
          <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item active"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>
              <span class="cp">{{</span> <span class="nv">page_num</span> <span class="cp">}}</span>
              <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"sr-only"</span><span class="nt">&gt;</span>(current)<span class="nt">&lt;/span&gt;</span>
            <span class="nt">&lt;/span&gt;</span>
          <span class="nt">&lt;/li&gt;</span>
        <span class="cp">{%</span> <span class="nv">elif</span> <span class="nv">page_num</span> <span class="o">&gt;</span> <span class="nv">page_obj.</span><span class="nb">number</span><span class="o">|</span><span class="nf">add</span><span class="err">:</span><span class="s1">'-3'</span> <span class="ow">and</span> <span class="nv">page_num</span> <span class="o">&lt;</span> <span class="nv">page_obj.</span><span class="nb">number</span><span class="o">|</span><span class="nf">add</span><span class="err">:</span><span class="s1">'3'</span> <span class="cp">%}</span>
          <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=</span><span class="cp">{{</span> <span class="nv">page_num</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">page_num</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;/li&gt;</span>
        <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">page_obj.has_next</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=</span><span class="cp">{{</span> <span class="nv">page_obj.next_page_number</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span>Next<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item disabled"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>Next<span class="nt">&lt;/span&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">page_obj.</span><span class="nb">number</span> <span class="o">!=</span> <span class="nv">paginator.num_pages</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=</span><span class="cp">{{</span> <span class="nv">paginator.num_pages</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span>Last<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item disabled"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>Last<span class="nt">&lt;/span&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
    <span class="nt">&lt;/ul&gt;</span>
  <span class="nt">&lt;/nav&gt;</span>
<span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/long-pages.png" alt="Long pages" /></p>

<hr />
