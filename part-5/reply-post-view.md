<h4 id="reply-post-view">Reply Post View</h4>

<p>Let’s implement now the reply post view so that we can add more data and progress with the implementation and tests.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/wireframe-reply.png" alt="Reply Wireframes" /></p>

<p>New URL route:</p>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/71a5f9f39202edfbab9bacf11844548b#file-urls-py-L39" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/topics/(?P&lt;topic_pk&gt;</span><span class="err">\</span><span class="s">d+)/reply/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">reply_topic</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'reply_topic'</span><span class="p">),</span></code></pre></figure>

<p>Create a new form for the post reply:</p>

<p><strong>boards/forms.py</strong>
<small><a href="https://gist.github.com/vitorfs/3dd5ed2b3e27b4c12886e9426acf8fda#file-forms-py-L20" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">forms</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Post</span>

<span class="k">class</span> <span class="nc">PostForm</span><span class="p">(</span><span class="n">forms</span><span class="o">.</span><span class="n">ModelForm</span><span class="p">):</span>
    <span class="k">class</span> <span class="nc">Meta</span><span class="p">:</span>
        <span class="n">model</span> <span class="o">=</span> <span class="n">Post</span>
        <span class="n">fields</span> <span class="o">=</span> <span class="p">[</span><span class="s">'message'</span><span class="p">,</span> <span class="p">]</span></code></pre></figure>

<p>A new view protected by <code class="highlighter-rouge">@login_required</code> and with a simple form processing logic:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/9e3811d9b11958b4106d99d9243efa71#file-views-py-L45" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.decorators</span> <span class="kn">import</span> <span class="n">login_required</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">get_object_or_404</span><span class="p">,</span> <span class="n">redirect</span><span class="p">,</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">.forms</span> <span class="kn">import</span> <span class="n">PostForm</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Topic</span>

<span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">reply_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="p">):</span>
    <span class="n">topic</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">board__pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">topic_pk</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">post</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
            <span class="n">post</span><span class="o">.</span><span class="n">topic</span> <span class="o">=</span> <span class="n">topic</span>
            <span class="n">post</span><span class="o">.</span><span class="n">created_by</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">user</span>
            <span class="n">post</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="o">=</span><span class="n">topic_pk</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'reply_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'topic'</span><span class="p">:</span> <span class="n">topic</span><span class="p">,</span> <span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>Also take the time to update the return redirect of the <strong>new_topic</strong> view function (marked with the comment
<strong># TODO</strong>).</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">new_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">NewTopicForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">topic</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
            <span class="c"># code suppressed ...</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="o">=</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span>  <span class="c"># &lt;- here</span>
    <span class="c"># code suppressed ...</span></code></pre></figure>

<p>Very important: in the view <strong>reply_topic</strong> we are using <code class="highlighter-rouge">topic_pk</code> because we are referring to the keyword argument
of the function, in the view <strong>new_topic</strong> we are using <code class="highlighter-rouge">topic.pk</code> because a <code class="highlighter-rouge">topic</code> is an object (Topic model
instance) and <code class="highlighter-rouge">.pk</code> we are accessing the <code class="highlighter-rouge">pk</code> property of the Topic model instance. Small detail, big difference.</p>

<p>The first version of our template:</p>

<p><strong>templates/reply_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Post a reply<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">topic.board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">topic.board.pk</span> <span class="nv">topic.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>Post a reply<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>

  <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
    <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Post a reply<span class="nt">&lt;/button&gt;</span>
  <span class="nt">&lt;/form&gt;</span>

  <span class="cp">{%</span> <span class="k">for</span> <span class="nv">post</span> <span class="ow">in</span> <span class="nv">topic.posts.all</span> <span class="cp">%}</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card mb-2"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body p-3"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row mb-3"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-6"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;strong</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">post.created_by.username</span> <span class="cp">}}</span><span class="nt">&lt;/strong&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-6 text-right"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"text-muted"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">post.created_at</span> <span class="cp">}}</span><span class="nt">&lt;/small&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
        <span class="cp">{{</span> <span class="nv">post.message</span> <span class="cp">}}</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/reply.png" alt="Reply form" /></p>

<p>Then after posting a reply, the user is redirected back to the topic posts:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-3.png" alt="Topic posts" /></p>

<p>We could now change the starter post, so to give it more emphasis in the page:</p>

<p><strong>templates/topic_posts.html</strong>
<small><a href="https://gist.github.com/vitorfs/3e4ad94ac3ae9d72194af4006d4aeaff#file-topic_posts-html-L20" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">for</span> <span class="nv">post</span> <span class="ow">in</span> <span class="nv">topic.posts.all</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card mb-2 </span><span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.first</span> <span class="cp">%}</span><span class="s">border-dark</span><span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.first</span> <span class="cp">%}</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-header text-white bg-dark py-2 px-3"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/div&gt;</span>
    <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body p-3"</span><span class="nt">&gt;</span>
      <span class="c">&lt;!-- code suppressed --&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/posts-4.png" alt="Topic posts" /></p>

<p>Now for the tests, pretty standard, just like we have been doing so far. Create a new file <strong>test_view_reply_topic.py</strong>
inside the <strong>boards/tests</strong> folder:</p>

<p><strong>boards/tests/test_view_reply_topic.py</strong>
<small><a href="https://gist.github.com/vitorfs/7148fcb95075fb6641e638214b751cf1" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">..models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Post</span><span class="p">,</span> <span class="n">Topic</span>
<span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">reply_topic</span>

<span class="k">class</span> <span class="nc">ReplyTopicTestCase</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="s">'''
    Base test case to be used in all `reply_topic` view tests
    '''</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">username</span> <span class="o">=</span> <span class="s">'john'</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">password</span> <span class="o">=</span> <span class="s">'123'</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">username</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">password</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">topic</span> <span class="o">=</span> <span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">subject</span><span class="o">=</span><span class="s">'Hello, world'</span><span class="p">,</span> <span class="n">board</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">board</span><span class="p">,</span> <span class="n">starter</span><span class="o">=</span><span class="n">user</span><span class="p">)</span>
        <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">message</span><span class="o">=</span><span class="s">'Lorem ipsum dolor sit amet'</span><span class="p">,</span> <span class="n">topic</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="p">,</span> <span class="n">created_by</span><span class="o">=</span><span class="n">user</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'reply_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span> <span class="s">'topic_pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">})</span>

<span class="k">class</span> <span class="nc">LoginRequiredReplyTopicTests</span><span class="p">(</span><span class="n">ReplyTopicTestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">ReplyTopicTests</span><span class="p">(</span><span class="n">ReplyTopicTestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">SuccessfulReplyTopicTests</span><span class="p">(</span><span class="n">ReplyTopicTestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">InvalidReplyTopicTests</span><span class="p">(</span><span class="n">ReplyTopicTestCase</span><span class="p">):</span>
    <span class="c"># ...</span></code></pre></figure>

<p>The essence here is the custom test case class <strong>ReplyTopicTestCase</strong>. Then all the four classes will extend this
test case.</p>

<p>First, we test if the view is protected with the <code class="highlighter-rouge">@login_required</code> decorator, then check the HTML inputs, status code.
Finally, we test a valid and an invalid form submission.</p>

<hr />
