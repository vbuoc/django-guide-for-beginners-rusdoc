<h4 id="querysets">QuerySets</h4>

<p>Let’s take the time now to explore some of the models’ API functionalities a little bit. First, let’s improve the
home view:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/boards-1.png" alt="Boards" /></p>

<p>We have three tasks here:</p>

<ul>
  <li>Display the posts count of the board;</li>
  <li>Display the topics count of the board;</li>
  <li>Display the last user who posted something and the date and time.</li>
</ul>

<p>Let’s play with the Python terminal first, before we jump into the implementation.</p>

<p>Since we are going to try things out in the Python terminal, it’s a good idea to define a <code class="highlighter-rouge">__str__</code> method for all
our models.</p>

<p><strong>boards/models.py</strong>
<small><a href="https://gist.github.com/vitorfs/9524eb42005697fbb79836285b50b1f4" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.db</span> <span class="kn">import</span> <span class="n">models</span>
<span class="kn">from</span> <span class="nn">django.utils.text</span> <span class="kn">import</span> <span class="n">Truncator</span>

<span class="k">class</span> <span class="nc">Board</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># ...</span>
    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">name</span>

<span class="k">class</span> <span class="nc">Topic</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># ...</span>
    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">subject</span>

<span class="k">class</span> <span class="nc">Post</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># ...</span>
    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">truncated_message</span> <span class="o">=</span> <span class="n">Truncator</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">message</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">truncated_message</span><span class="o">.</span><span class="n">chars</span><span class="p">(</span><span class="mi">30</span><span class="p">)</span></code></pre></figure>

<p>In the Post model we are using the <strong>Truncator</strong> utility class. It’s a convenient way to truncate long strings into
an arbitrary string size (here we are using 30).</p>

<p>Now let’s open the Python shell terminal:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">python</span> <span class="n">manage</span><span class="o">.</span><span class="n">py</span> <span class="n">shell</span>

<span class="c"># First get a board instance from the database</span>
<span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">)</span></code></pre></figure>

<p>The easiest of the three tasks is to get the current topics count, because the Topic and Board are directly related:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="n">topics</span><span class="o">.</span><span class="nb">all</span><span class="p">()</span>
<span class="o">&lt;</span><span class="n">QuerySet</span> <span class="p">[</span><span class="o">&lt;</span><span class="n">Topic</span><span class="p">:</span> <span class="n">Hello</span> <span class="n">everyone</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Topic</span><span class="p">:</span> <span class="n">Test</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Topic</span><span class="p">:</span> <span class="n">Testing</span> <span class="n">a</span> <span class="n">new</span> <span class="n">post</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Topic</span><span class="p">:</span> <span class="n">Hi</span><span class="o">&gt;</span><span class="p">]</span><span class="o">&gt;</span>

<span class="n">board</span><span class="o">.</span><span class="n">topics</span><span class="o">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">4</span></code></pre></figure>

<p>That’s about it.</p>

<p>Now the number of <em>posts</em> within a <em>board</em> is a little bit trickier because Post is not directly related to Board.</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">boards.models</span> <span class="kn">import</span> <span class="n">Post</span>

<span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">all</span><span class="p">()</span>
<span class="o">&lt;</span><span class="n">QuerySet</span> <span class="p">[</span><span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">This</span> <span class="ow">is</span> <span class="n">my</span> <span class="n">first</span> <span class="n">topic</span><span class="o">..</span> <span class="p">:</span><span class="o">-</span><span class="p">)</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">test</span><span class="o">.&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Hi</span> <span class="n">everyone</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">New</span> <span class="n">test</span> <span class="n">here</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Testing</span> <span class="n">the</span> <span class="n">new</span> <span class="n">reply</span> <span class="n">feature</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Lorem</span> <span class="n">ipsum</span> <span class="n">dolor</span> <span class="n">sit</span> <span class="n">amet</span><span class="p">,</span><span class="o">...&gt;</span><span class="p">,</span>
  <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">hi</span> <span class="n">there</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">test</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Testing</span><span class="o">..&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">some</span> <span class="n">reply</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Random</span> <span class="n">random</span><span class="o">.&gt;</span>
<span class="p">]</span><span class="o">&gt;</span>

<span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">11</span></code></pre></figure>

<p>Here we have 11 posts. But not all of them belongs to the “Django” board.</p>

<p>Here is how we can filter it:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">boards.models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Post</span>

<span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">)</span>

<span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">topic__board</span><span class="o">=</span><span class="n">board</span><span class="p">)</span>
<span class="o">&lt;</span><span class="n">QuerySet</span> <span class="p">[</span><span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">This</span> <span class="ow">is</span> <span class="n">my</span> <span class="n">first</span> <span class="n">topic</span><span class="o">..</span> <span class="p">:</span><span class="o">-</span><span class="p">)</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">test</span><span class="o">.&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">hi</span> <span class="n">there</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Hi</span> <span class="n">everyone</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Lorem</span> <span class="n">ipsum</span> <span class="n">dolor</span> <span class="n">sit</span> <span class="n">amet</span><span class="p">,</span><span class="o">...&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">New</span> <span class="n">test</span> <span class="n">here</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Testing</span> <span class="n">the</span> <span class="n">new</span> <span class="n">reply</span> <span class="n">feature</span><span class="err">!</span><span class="o">&gt;</span>
<span class="p">]</span><span class="o">&gt;</span>

<span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">topic__board</span><span class="o">=</span><span class="n">board</span><span class="p">)</span><span class="o">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">7</span></code></pre></figure>

<p>The double underscores <code class="highlighter-rouge">topic__board</code> is used to navigate through the models’ relationships. Under the hoods, Django
builds the bridge between the Board - Topic - Post, and build a SQL query to retrieve just the posts that belong to
a specific board.</p>

<p>Now our last mission is to identify the last post.</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="c"># order by the `created_at` field, getting the most recent first</span>
<span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">topic__board</span><span class="o">=</span><span class="n">board</span><span class="p">)</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-created_at'</span><span class="p">)</span>
<span class="o">&lt;</span><span class="n">QuerySet</span> <span class="p">[</span><span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">testing</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">new</span> <span class="n">post</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">hi</span> <span class="n">there</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Lorem</span> <span class="n">ipsum</span> <span class="n">dolor</span> <span class="n">sit</span> <span class="n">amet</span><span class="p">,</span><span class="o">...&gt;</span><span class="p">,</span>
  <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Testing</span> <span class="n">the</span> <span class="n">new</span> <span class="n">reply</span> <span class="n">feature</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">New</span> <span class="n">test</span> <span class="n">here</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">Hi</span> <span class="n">everyone</span><span class="err">!</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">test</span><span class="o">.&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">This</span> <span class="ow">is</span> <span class="n">my</span> <span class="n">first</span> <span class="n">topic</span><span class="o">..</span> <span class="p">:</span><span class="o">-</span><span class="p">)</span><span class="o">&gt;</span>
<span class="p">]</span><span class="o">&gt;</span>

<span class="c"># we can use the `first()` method to just grab the result that interest us</span>
<span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">topic__board</span><span class="o">=</span><span class="n">board</span><span class="p">)</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-created_at'</span><span class="p">)</span><span class="o">.</span><span class="n">first</span><span class="p">()</span>
<span class="o">&lt;</span><span class="n">Post</span><span class="p">:</span> <span class="n">testing</span><span class="o">&gt;</span></code></pre></figure>

<p>Sweet. Now we can implement it.</p>

<p><strong>boards/models.py</strong>
<small><a href="https://gist.github.com/vitorfs/74077336decd75292082752eb8405ad3" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.db</span> <span class="kn">import</span> <span class="n">models</span>

<span class="k">class</span> <span class="nc">Board</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">name</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">30</span><span class="p">,</span> <span class="n">unique</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">description</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">100</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">name</span>

    <span class="k">def</span> <span class="nf">get_posts_count</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">topic__board</span><span class="o">=</span><span class="bp">self</span><span class="p">)</span><span class="o">.</span><span class="n">count</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">get_last_post</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">topic__board</span><span class="o">=</span><span class="bp">self</span><span class="p">)</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-created_at'</span><span class="p">)</span><span class="o">.</span><span class="n">first</span><span class="p">()</span></code></pre></figure>

<p>Observe that we are using <code class="highlighter-rouge">self</code>, because this method will be used by a Board instance. So that means we are using
this instance to filter the QuerySet.</p>

<p>Now we can improve the home HTML template to display this brand new information:</p>

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
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>
            <span class="cp">{{</span> <span class="nv">board.get_posts_count</span> <span class="cp">}}</span>
          <span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>
            <span class="cp">{{</span> <span class="nv">board.topics.count</span> <span class="cp">}}</span>
          <span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td</span> <span class="na">class=</span><span class="s">"align-middle"</span><span class="nt">&gt;</span>
            <span class="cp">{%</span> <span class="k">with</span> <span class="nv">post</span><span class="err">=</span><span class="nv">board.get_last_post</span> <span class="cp">%}</span>
              <span class="nt">&lt;small&gt;</span>
                <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">board.pk</span> <span class="nv">post.topic.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
                  By <span class="cp">{{</span> <span class="nv">post.created_by.username</span> <span class="cp">}}</span> at <span class="cp">{{</span> <span class="nv">post.created_at</span> <span class="cp">}}</span>
                <span class="nt">&lt;/a&gt;</span>
              <span class="nt">&lt;/small&gt;</span>
            <span class="cp">{%</span> <span class="k">endwith</span> <span class="cp">%}</span>
          <span class="nt">&lt;/td&gt;</span>
        <span class="nt">&lt;/tr&gt;</span>
      <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
    <span class="nt">&lt;/tbody&gt;</span>
  <span class="nt">&lt;/table&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>And that’s the result for now:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/boards-2.png" alt="Boards" /></p>

<p>Run the tests:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.......................................................EEE......................
======================================================================
ERROR: test_home_url_resolves_home_view (boards.tests.test_view_home.HomeTests)
----------------------------------------------------------------------
django.urls.exceptions.NoReverseMatch: Reverse for 'topic_posts' with arguments '(1, '')' not found. 1 pattern(s) tried: ['boards/(?P&lt;pk&gt;\\d+)/topics/(?P&lt;topic_pk&gt;\\d+)/$']

======================================================================
ERROR: test_home_view_contains_link_to_topics_page (boards.tests.test_view_home.HomeTests)
----------------------------------------------------------------------
django.urls.exceptions.NoReverseMatch: Reverse for 'topic_posts' with arguments '(1, '')' not found. 1 pattern(s) tried: ['boards/(?P&lt;pk&gt;\\d+)/topics/(?P&lt;topic_pk&gt;\\d+)/$']

======================================================================
ERROR: test_home_view_status_code (boards.tests.test_view_home.HomeTests)
----------------------------------------------------------------------
django.urls.exceptions.NoReverseMatch: Reverse for 'topic_posts' with arguments '(1, '')' not found. 1 pattern(s) tried: ['boards/(?P&lt;pk&gt;\\d+)/topics/(?P&lt;topic_pk&gt;\\d+)/$']

----------------------------------------------------------------------
Ran 80 tests in 5.663s

FAILED (errors=3)
Destroying test database for alias 'default'...</code></pre></figure>

<p>It seems like we have a problem with our implementation here. The application is crashing if there are no posts.</p>

<p><strong>templates/home.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">with</span> <span class="nv">post</span><span class="err">=</span><span class="nv">board.get_last_post</span> <span class="cp">%}</span>
  <span class="cp">{%</span> <span class="k">if</span> <span class="nv">post</span> <span class="cp">%}</span>
    <span class="nt">&lt;small&gt;</span>
      <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">board.pk</span> <span class="nv">post.topic.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
        By <span class="cp">{{</span> <span class="nv">post.created_by.username</span> <span class="cp">}}</span> at <span class="cp">{{</span> <span class="nv">post.created_at</span> <span class="cp">}}</span>
      <span class="nt">&lt;/a&gt;</span>
    <span class="nt">&lt;/small&gt;</span>
  <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
    <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;em&gt;</span>No posts yet.<span class="nt">&lt;/em&gt;</span>
    <span class="nt">&lt;/small&gt;</span>
  <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
<span class="cp">{%</span> <span class="k">endwith</span> <span class="cp">%}</span></code></pre></figure>

<p>Run the tests again:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
................................................................................
----------------------------------------------------------------------
Ran 80 tests in 5.630s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>I added a new board with no messages just to check the “empty message”:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/boards-3.png" alt="Boards" /></p>

<p>Now it’s time to improve the topics listing view.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-3.png" alt="Topics" /></p>

<p>I will show you another way to include the count, this time to the number of replies, in a more effective way.</p>

<p>As usual, let’s try first with the Python shell:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py shell</code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.db.models</span> <span class="kn">import</span> <span class="n">Count</span>
<span class="kn">from</span> <span class="nn">boards.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">)</span>

<span class="n">topics</span> <span class="o">=</span> <span class="n">board</span><span class="o">.</span><span class="n">topics</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-last_updated'</span><span class="p">)</span><span class="o">.</span><span class="n">annotate</span><span class="p">(</span><span class="n">replies</span><span class="o">=</span><span class="n">Count</span><span class="p">(</span><span class="s">'posts'</span><span class="p">))</span>

<span class="k">for</span> <span class="n">topic</span> <span class="ow">in</span> <span class="n">topics</span><span class="p">:</span>
    <span class="k">print</span><span class="p">(</span><span class="n">topic</span><span class="o">.</span><span class="n">replies</span><span class="p">)</span>

<span class="mi">2</span>
<span class="mi">4</span>
<span class="mi">2</span>
<span class="mi">1</span></code></pre></figure>

<p>Here we are using the <code class="highlighter-rouge">annotate</code> QuerySet method to generate a new “column” on the fly. This new column, which will
be translated into a property, accessible via <code class="highlighter-rouge">topic.replies</code> contain the count of posts a given topic has.</p>

<p>We can do just a minor fix because the replies should not consider the starter topic (which is also a Post instance).</p>

<p>So here is how we do it:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">topics</span> <span class="o">=</span> <span class="n">board</span><span class="o">.</span><span class="n">topics</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-last_updated'</span><span class="p">)</span><span class="o">.</span><span class="n">annotate</span><span class="p">(</span><span class="n">replies</span><span class="o">=</span><span class="n">Count</span><span class="p">(</span><span class="s">'posts'</span><span class="p">)</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span>

<span class="k">for</span> <span class="n">topic</span> <span class="ow">in</span> <span class="n">topics</span><span class="p">:</span>
    <span class="k">print</span><span class="p">(</span><span class="n">topic</span><span class="o">.</span><span class="n">replies</span><span class="p">)</span>

<span class="mi">1</span>
<span class="mi">3</span>
<span class="mi">1</span>
<span class="mi">0</span></code></pre></figure>

<p>Cool, right?</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/f22b493b3e076aba9351c9d98f547f5e#file-views-py-L14" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.db.models</span> <span class="kn">import</span> <span class="n">Count</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">get_object_or_404</span><span class="p">,</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="n">topics</span> <span class="o">=</span> <span class="n">board</span><span class="o">.</span><span class="n">topics</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-last_updated'</span><span class="p">)</span><span class="o">.</span><span class="n">annotate</span><span class="p">(</span><span class="n">replies</span><span class="o">=</span><span class="n">Count</span><span class="p">(</span><span class="s">'posts'</span><span class="p">)</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'topics.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">,</span> <span class="s">'topics'</span><span class="p">:</span> <span class="n">topics</span><span class="p">})</span></code></pre></figure>

<p><strong>templates/topics.html</strong>
<small><a href="https://gist.github.com/vitorfs/1a2235f05f436c92025dc86028c22fc4#file-topics-html-L28" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">for</span> <span class="nv">topic</span> <span class="ow">in</span> <span class="nv">topics</span> <span class="cp">%}</span>
  <span class="nt">&lt;tr&gt;</span>
    <span class="nt">&lt;td&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">board.pk</span> <span class="nv">topic.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.starter.username</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.replies</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span>0<span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.last_updated</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
  <span class="nt">&lt;/tr&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-4.png" alt="Topics" /></p>

<p>Next step now is to fix the <em>views</em> count. But for that, we will need to create a new field.</p>

<hr />
