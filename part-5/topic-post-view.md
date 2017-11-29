<h4 id="topic-posts-view">Topic Posts View</h4>

<p>Let’s take the time now to implement the posts listing page, accordingly to the wireframe below:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/wireframe-posts.png" alt="Wireframe Posts" /></p>

<p>First, we need a route:</p>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/aede6d3b7dc3494cf0df48f796075403#file-urls-py-L38" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/topics/(?P&lt;topic_pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">topic_posts</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'topic_posts'</span><span class="p">),</span></code></pre></figure>

<p>Observe that now we are dealing with two keyword arguments: <code class="highlighter-rouge">pk</code> which is used to identify the Board, and now we have
the <code class="highlighter-rouge">topic_pk</code> which is used to identify which topic to retrieve from the database.</p>

<p>The matching view would be like this:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/3d73ef25a01eceea07ef3ad8538437cf#file-views-py-L39" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">get_object_or_404</span><span class="p">,</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Topic</span>

<span class="k">def</span> <span class="nf">topic_posts</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="p">):</span>
    <span class="n">topic</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">board__pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">topic_pk</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'topic_posts.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'topic'</span><span class="p">:</span> <span class="n">topic</span><span class="p">})</span></code></pre></figure>

<p>Note that we are indirectly retrieving the current board. Remember that the topic model is related to the board model,
so we can access the current board. You will see in the next snippet:</p>

<p><strong>templates/topic_posts.html</strong>
<small><a href="https://gist.github.com/vitorfs/17e583f4f0068850c5929bd307dd436a" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}{{</span> <span class="nv">topic.subject</span> <span class="cp">}}{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">topic.board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Observe that now instead of using <code class="highlighter-rouge">board.name</code> in the template, we are navigating through the topic properties, using
<code class="highlighter-rouge">topic.board.name</code>.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-1.png" alt="Posts" /></p>

<p>Now let’s create a new test file for the <strong>topic_posts</strong> view:</p>

<p><strong>boards/tests/test_view_topic_posts.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span><span class="p">,</span> <span class="n">reverse</span>

<span class="kn">from</span> <span class="nn">..models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Post</span><span class="p">,</span> <span class="n">Topic</span>
<span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">topic_posts</span>


<span class="k">class</span> <span class="nc">TopicPostsTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'123'</span><span class="p">)</span>
        <span class="n">topic</span> <span class="o">=</span> <span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">subject</span><span class="o">=</span><span class="s">'Hello, world'</span><span class="p">,</span> <span class="n">board</span><span class="o">=</span><span class="n">board</span><span class="p">,</span> <span class="n">starter</span><span class="o">=</span><span class="n">user</span><span class="p">)</span>
        <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">message</span><span class="o">=</span><span class="s">'Lorem ipsum dolor sit amet'</span><span class="p">,</span> <span class="n">topic</span><span class="o">=</span><span class="n">topic</span><span class="p">,</span> <span class="n">created_by</span><span class="o">=</span><span class="n">user</span><span class="p">)</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span> <span class="s">'topic_pk'</span><span class="p">:</span> <span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_view_function</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/boards/1/topics/1/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="p">,</span> <span class="n">topic_posts</span><span class="p">)</span></code></pre></figure>

<p>Note that the test setup is starting to get more complex. We can create mixins or an abstract class to reuse the code
as needed. We can also use a third party library to setup some test data, to reduce the boilerplate code.</p>

<p>Also, by now we already have a significant amount of tests, and it’s gradually starting to run slower. We can instruct
the test suite just to run tests from a given app:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test boards</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.......................
----------------------------------------------------------------------
Ran 23 tests in 1.246s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>We could also run only a specific test file:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test boards.tests.test_view_topic_posts</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.129s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Or just a specific test case:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test boards.tests.test_view_topic_posts.TopicPostsTests.test_status_code</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.100s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Cool, right?</p>

<p>Let’s keep moving forward.</p>

<p>Inside the <strong>topic_posts.html</strong>, we can create a for loop iterating over the topic posts:</p>

<p><strong>templates/topic_posts.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}{{</span> <span class="nv">topic.subject</span> <span class="cp">}}{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">topic.board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>

  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"#"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span> <span class="na">role=</span><span class="s">"button"</span><span class="nt">&gt;</span>Reply<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/div&gt;</span>

  <span class="cp">{%</span> <span class="k">for</span> <span class="nv">post</span> <span class="ow">in</span> <span class="nv">topic.posts.all</span> <span class="cp">%}</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card mb-2"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body p-3"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-2"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;img</span> <span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'img/avatar.svg'</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">alt=</span><span class="s">"</span><span class="cp">{{</span> <span class="nv">post.created_by.username</span> <span class="cp">}}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"w-100"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;small&gt;</span>Posts: <span class="cp">{{</span> <span class="nv">post.created_by.posts.count</span> <span class="cp">}}</span><span class="nt">&lt;/small&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-10"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row mb-3"</span><span class="nt">&gt;</span>
              <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-6"</span><span class="nt">&gt;</span>
                <span class="nt">&lt;strong</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">post.created_by.username</span> <span class="cp">}}</span><span class="nt">&lt;/strong&gt;</span>
              <span class="nt">&lt;/div&gt;</span>
              <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-6 text-right"</span><span class="nt">&gt;</span>
                <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">post.created_at</span> <span class="cp">}}</span><span class="nt">&lt;/small&gt;</span>
              <span class="nt">&lt;/div&gt;</span>
            <span class="nt">&lt;/div&gt;</span>
            <span class="cp">{{</span> <span class="nv">post.message</span> <span class="cp">}}</span>
            <span class="cp">{%</span> <span class="k">if</span> <span class="nv">post.created_by</span> <span class="o">==</span> <span class="nv">user</span> <span class="cp">%}</span>
              <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mt-3"</span><span class="nt">&gt;</span>
                <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"#"</span> <span class="na">class=</span><span class="s">"btn btn-primary btn-sm"</span> <span class="na">role=</span><span class="s">"button"</span><span class="nt">&gt;</span>Edit<span class="nt">&lt;/a&gt;</span>
              <span class="nt">&lt;/div&gt;</span>
            <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
          <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-2.png" alt="Posts" /></p>

<p>Since right now we don’t have a way to upload a user picture, let’s just have an empty image.</p>

<p>I downloaded a free image from <a href="https://www.iconfinder.com/search/?q=user&amp;license=2&amp;price=free" target="_blank" rel="noopener">IconFinder</a>
and saved in the <strong>static/img</strong> folder of the project.</p>

<p>We still haven’t really explored Django’s ORM, but the code <code class="highlighter-rouge"><span class="p">{</span><span class="err">{</span><span class="w"> </span><span class="err">post.created_by.posts.count</span><span class="w"> </span><span class="p">}</span><span class="err">}</span></code>
is executing a <code class="highlighter-rouge">select count</code> in the database. Even though the result is correct, it is a bad approach. Right now it’s
causing several unnecessary queries in the database. But hey, don’t worry about that right now. Let’s focus on how
we interact with the application. Later on, we are going to improve this code, and how to diagnose heavy queries.</p>

<p>Another interesting point here is that we are testing if the current post belongs to the authenticated user:
<code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">if</span><span class="w"> </span><span class="err">post.created_by</span><span class="w"> </span><span class="err">==</span><span class="w"> </span><span class="err">user</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>. And we are only showing the edit button for the owner of the
post.</p>

<p>Since we now have the URL route to the topic posts listing, update the <strong>topics.html</strong> template with the link:</p>

<p><strong>templates/topics.html</strong>
<small><a href="https://gist.github.com/vitorfs/cb4b7c9ff382ddeafb4114d0c84b3869" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">for</span> <span class="nv">topic</span> <span class="ow">in</span> <span class="nv">board.topics.all</span> <span class="cp">%}</span>
  <span class="nt">&lt;tr&gt;</span>
    <span class="nt">&lt;td&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">board.pk</span> <span class="nv">topic.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.starter.username</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span>0<span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span>0<span class="nt">&lt;/td&gt;</span>
    <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.last_updated</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
  <span class="nt">&lt;/tr&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<hr />
