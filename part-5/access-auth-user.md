<h4 id="accessing-the-authenticated-user">Accessing the Authenticated User</h4>

<p>Now we can improve the <strong>new_topic</strong> view and this time set the proper user, instead of just querying the database
and picking the first user. That code was temporary because we had no way to authenticate the user. But now we can do
better:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/483936caca4618dc275545ad2dfbef24#file-views-py-L19" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.decorators</span> <span class="kn">import</span> <span class="n">login_required</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">get_object_or_404</span><span class="p">,</span> <span class="n">redirect</span><span class="p">,</span> <span class="n">render</span>

<span class="kn">from</span> <span class="nn">.forms</span> <span class="kn">import</span> <span class="n">NewTopicForm</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Post</span>

<span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">new_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">NewTopicForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">topic</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">board</span> <span class="o">=</span> <span class="n">board</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">starter</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">user</span>  <span class="c"># &lt;- here</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span>
                <span class="n">message</span><span class="o">=</span><span class="n">form</span><span class="o">.</span><span class="n">cleaned_data</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'message'</span><span class="p">),</span>
                <span class="n">topic</span><span class="o">=</span><span class="n">topic</span><span class="p">,</span>
                <span class="n">created_by</span><span class="o">=</span><span class="n">request</span><span class="o">.</span><span class="n">user</span>  <span class="c"># &lt;- and here</span>
            <span class="p">)</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span>  <span class="c"># TODO: redirect to the created topic page</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">NewTopicForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">,</span> <span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>We can do a quick test here by adding a new topic:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-2.png" alt="New topic" /></p>

<hr />
