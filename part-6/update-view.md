<h4 id="update-view">Update View</h4>

<p>Let’s get back to the implementation of our project. This time we are going to use a GCBV to implement the
<strong>edit post</strong> view:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/edit.png" alt="Edit post" /></p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/376657085570a3dedf7e5f6e6fffc5e3#file-views-py-L66" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">redirect</span>
<span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">UpdateView</span>
<span class="kn">from</span> <span class="nn">django.utils</span> <span class="kn">import</span> <span class="n">timezone</span>

<span class="k">class</span> <span class="nc">PostUpdateView</span><span class="p">(</span><span class="n">UpdateView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Post</span>
    <span class="n">fields</span> <span class="o">=</span> <span class="p">(</span><span class="s">'message'</span><span class="p">,</span> <span class="p">)</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'edit_post.html'</span>
    <span class="n">pk_url_kwarg</span> <span class="o">=</span> <span class="s">'post_pk'</span>
    <span class="n">context_object_name</span> <span class="o">=</span> <span class="s">'post'</span>

    <span class="k">def</span> <span class="nf">form_valid</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">form</span><span class="p">):</span>
        <span class="n">post</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
        <span class="n">post</span><span class="o">.</span><span class="n">updated_by</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">request</span><span class="o">.</span><span class="n">user</span>
        <span class="n">post</span><span class="o">.</span><span class="n">updated_at</span> <span class="o">=</span> <span class="n">timezone</span><span class="o">.</span><span class="n">now</span><span class="p">()</span>
        <span class="n">post</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">post</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="o">=</span><span class="n">post</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span></code></pre></figure>

<p>With the <strong>UpdateView</strong> and the <strong>CreateView</strong>, we have the option to either define <strong>form_class</strong> or the <strong>fields</strong>
attribute. In the example above we are using the <strong>fields</strong> attribute to create a model form on-the-fly. Internally,
Django will use a model form factory to compose a form of the <strong>Post</strong> model. Since it’s a very simple form with just
the <strong>message</strong> field, we can afford to work like this. But for complex form definitions, it’s better to define a model
form externally and refer to it here.</p>

<p>The <strong>pk_url_kwarg</strong> will be used to identify the name of the keyword argument used to retrieve the <strong>Post</strong> object.
It’s the same as we define in the <strong>urls.py</strong>.</p>

<p>If we don’t set the <strong>context_object_name</strong> attribute, the <strong>Post</strong> object will be available in the template as
“object.” So, here we are using the <strong>context_object_name</strong> to rename it to <strong>post</strong> instead. You will see how we are
using it in the template below.</p>

<p>In this particular example, we had to override the <strong>form_valid()</strong> method so as to set some extra fields such as the
<strong>updated_by</strong> and <strong>updated_at</strong>. You can see what the base <strong>form_valid()</strong> method looks like here:
<a href="https://ccbv.co.uk/projects/Django/1.11/django.views.generic.edit/UpdateView/#form_valid" target="_blank" rel="noopener">UpdateView#form_valid</a>.</p>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/376657085570a3dedf7e5f6e6fffc5e3#file-urls-py-L40" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c"># ...</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/topics/(?P&lt;topic_pk&gt;</span><span class="err">\</span><span class="s">d+)/posts/(?P&lt;post_pk&gt;</span><span class="err">\</span><span class="s">d+)/edit/$'</span><span class="p">,</span>
        <span class="n">views</span><span class="o">.</span><span class="n">PostUpdateView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'edit_post'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Now we can add the link to the edit page:</p>

<p><strong>templates/topic_posts.html</strong>
<small><a href="https://gist.github.com/vitorfs/589d31af9d06c5b21ec9b1623be0c357#file-topic_posts-html-L40" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">if</span> <span class="nv">post.created_by</span> <span class="o">==</span> <span class="nv">user</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mt-3"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'edit_post'</span> <span class="nv">post.topic.board.pk</span> <span class="nv">post.topic.pk</span> <span class="nv">post.pk</span> <span class="cp">%}</span><span class="s">"</span>
       <span class="na">class=</span><span class="s">"btn btn-primary btn-sm"</span>
       <span class="na">role=</span><span class="s">"button"</span><span class="nt">&gt;</span>Edit<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span></code></pre></figure>

<p><strong>templates/edit_post.html</strong>
<small><a href="https://gist.github.com/vitorfs/376657085570a3dedf7e5f6e6fffc5e3#file-edit_post-html" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Edit post<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">post.topic.board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">post.topic.board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">post.topic.board.pk</span> <span class="nv">post.topic.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">post.topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>Edit post<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">class=</span><span class="s">"mb-4"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
    <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Save changes<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'topic_posts'</span> <span class="nv">post.topic.board.pk</span> <span class="nv">post.topic.pk</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-outline-secondary"</span> <span class="na">role=</span><span class="s">"button"</span><span class="nt">&gt;</span>Cancel<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/form&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Observe now how we are navigating through the post object: <strong>post.topic.board.pk</strong>. If we didn’t set the
<strong>context_object_name</strong> to <strong>post</strong>, it would be available as: <strong>object.topic.board.pk</strong>. Got it?</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/edit-1.png" alt="Edit post" /></p>

<h5 id="testing-the-update-view">Testing The Update View</h5>

<p>Create a new test file named <strong>test_view_edit_post.py</strong> inside the <strong>boards/tests</strong> folder. Clicking on the link below
you will see many routine tests, just like we have been doing in this tutorial. So I will just highlight the new parts
here:</p>

<p><strong>boards/tests/test_view_edit_post.py</strong>
<small><a href="https://gist.github.com/vitorfs/286917ce31687732eba4e545fc3cbdea" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">..models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Post</span><span class="p">,</span> <span class="n">Topic</span>
<span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">PostUpdateView</span>

<span class="k">class</span> <span class="nc">PostUpdateViewTestCase</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="s">'''
    Base test case to be used in all `PostUpdateView` view tests
    '''</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">username</span> <span class="o">=</span> <span class="s">'john'</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">password</span> <span class="o">=</span> <span class="s">'123'</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">username</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">password</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">topic</span> <span class="o">=</span> <span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">subject</span><span class="o">=</span><span class="s">'Hello, world'</span><span class="p">,</span> <span class="n">board</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">board</span><span class="p">,</span> <span class="n">starter</span><span class="o">=</span><span class="n">user</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">post</span> <span class="o">=</span> <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">message</span><span class="o">=</span><span class="s">'Lorem ipsum dolor sit amet'</span><span class="p">,</span> <span class="n">topic</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="p">,</span> <span class="n">created_by</span><span class="o">=</span><span class="n">user</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'edit_post'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span>
            <span class="s">'pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span>
            <span class="s">'topic_pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span>
            <span class="s">'post_pk'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">post</span><span class="o">.</span><span class="n">pk</span>
        <span class="p">})</span>

<span class="k">class</span> <span class="nc">LoginRequiredPostUpdateViewTests</span><span class="p">(</span><span class="n">PostUpdateViewTestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        Test if only logged in users can edit the posts
        '''</span>
        <span class="n">login_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'login'</span><span class="p">)</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="n">response</span><span class="p">,</span> <span class="s">'{login_url}?next={url}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">login_url</span><span class="o">=</span><span class="n">login_url</span><span class="p">,</span> <span class="n">url</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">url</span><span class="p">))</span>

<span class="k">class</span> <span class="nc">UnauthorizedPostUpdateViewTests</span><span class="p">(</span><span class="n">PostUpdateViewTestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        Create a new user different from the one who posted
        '''</span>
        <span class="nb">super</span><span class="p">()</span><span class="o">.</span><span class="n">setUp</span><span class="p">()</span>
        <span class="n">username</span> <span class="o">=</span> <span class="s">'jane'</span>
        <span class="n">password</span> <span class="o">=</span> <span class="s">'321'</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="n">username</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'jane@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="n">password</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">login</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="n">username</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="n">password</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        A topic should be edited only by the owner.
        Unauthorized users should get a 404 response (Page Not Found)
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">404</span><span class="p">)</span>


<span class="k">class</span> <span class="nc">PostUpdateViewTests</span><span class="p">(</span><span class="n">PostUpdateViewTestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">SuccessfulPostUpdateViewTests</span><span class="p">(</span><span class="n">PostUpdateViewTestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">InvalidPostUpdateViewTests</span><span class="p">(</span><span class="n">PostUpdateViewTestCase</span><span class="p">):</span>
    <span class="c"># ...</span></code></pre></figure>

<p>For now, the important parts are: <strong>PostUpdateViewTestCase</strong> is a class we defined to be reused across the other
test cases. It’s just the basic setup, creating users, topic, boards, and so on.</p>

<p>The class <strong>LoginRequiredPostUpdateViewTests</strong> will test if the view is protected with the <code class="highlighter-rouge">@login_required</code>
decorator. That is if only authenticated users can access the edit page.</p>

<p>The class <strong>UnauthorizedPostUpdateViewTests</strong> creates a new user, different from the one who posted and tries to
access the edit page. The application should only authorize the owner of the post to edit it.</p>

<p>Let’s run the tests:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test boards.tests.test_view_edit_post</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..F.......F
======================================================================
FAIL: test_redirection (boards.tests.test_view_edit_post.LoginRequiredPostUpdateViewTests)
----------------------------------------------------------------------
...
AssertionError: 200 != 302 : Response didn't redirect as expected: Response code was 200 (expected 302)

======================================================================
FAIL: test_status_code (boards.tests.test_view_edit_post.UnauthorizedPostUpdateViewTests)
----------------------------------------------------------------------
...
AssertionError: 200 != 404

----------------------------------------------------------------------
Ran 11 tests in 1.360s

FAILED (failures=2)
Destroying test database for alias 'default'...</code></pre></figure>

<p>First, let’s fix the problem with the <code class="highlighter-rouge">@login_required</code> decorator. The way we use view decorators on class-based views
is a little bit different. We need an extra import:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/826a6d421ebbeb80a0aee8e1b9b70398#file-views-py-L67" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.decorators</span> <span class="kn">import</span> <span class="n">login_required</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">redirect</span>
<span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">UpdateView</span>
<span class="kn">from</span> <span class="nn">django.utils</span> <span class="kn">import</span> <span class="n">timezone</span>
<span class="kn">from</span> <span class="nn">django.utils.decorators</span> <span class="kn">import</span> <span class="n">method_decorator</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Post</span>

<span class="nd">@method_decorator</span><span class="p">(</span><span class="n">login_required</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'dispatch'</span><span class="p">)</span>
<span class="k">class</span> <span class="nc">PostUpdateView</span><span class="p">(</span><span class="n">UpdateView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Post</span>
    <span class="n">fields</span> <span class="o">=</span> <span class="p">(</span><span class="s">'message'</span><span class="p">,</span> <span class="p">)</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'edit_post.html'</span>
    <span class="n">pk_url_kwarg</span> <span class="o">=</span> <span class="s">'post_pk'</span>
    <span class="n">context_object_name</span> <span class="o">=</span> <span class="s">'post'</span>

    <span class="k">def</span> <span class="nf">form_valid</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">form</span><span class="p">):</span>
        <span class="n">post</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
        <span class="n">post</span><span class="o">.</span><span class="n">updated_by</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">request</span><span class="o">.</span><span class="n">user</span>
        <span class="n">post</span><span class="o">.</span><span class="n">updated_at</span> <span class="o">=</span> <span class="n">timezone</span><span class="o">.</span><span class="n">now</span><span class="p">()</span>
        <span class="n">post</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">post</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="o">=</span><span class="n">post</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span></code></pre></figure>

<p>We can’t decorate the class directly with the <code class="highlighter-rouge">@login_required</code> decorator. We have to use the utility
<code class="highlighter-rouge">@method_decorator</code>, and pass a decorator (or a list of decorators) and tell which method should be decorated. In
class-based views it’s common to decorate the <strong>dispatch</strong> method. It’s an internal method Django use (defined inside
the View class). All requests pass through this method, so it’s safe to decorate it.</p>

<p>Run the tests:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test boards.tests.test_view_edit_post</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..........F
======================================================================
FAIL: test_status_code (boards.tests.test_view_edit_post.UnauthorizedPostUpdateViewTests)
----------------------------------------------------------------------
...
AssertionError: 200 != 404

----------------------------------------------------------------------
Ran 11 tests in 1.353s

FAILED (failures=1)
Destroying test database for alias 'default'...</code></pre></figure>

<p>Okay! We fixed the <code class="highlighter-rouge">@login_required</code> problem. Now we have to deal with the other users editing any posts problem.</p>

<p>The easiest way to solve this problem is by overriding the <code class="highlighter-rouge">get_queryset</code> method of the <strong>UpdateView</strong>. You can see
what the original method looks like here <a href="https://ccbv.co.uk/projects/Django/1.11/django.views.generic.edit/UpdateView/#get_queryset" target="_blank" rel="noopener">UpdateView#get_queryset</a>.</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/667d8439ecf05e58f14fcc74672e48da" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="nd">@method_decorator</span><span class="p">(</span><span class="n">login_required</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'dispatch'</span><span class="p">)</span>
<span class="k">class</span> <span class="nc">PostUpdateView</span><span class="p">(</span><span class="n">UpdateView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Post</span>
    <span class="n">fields</span> <span class="o">=</span> <span class="p">(</span><span class="s">'message'</span><span class="p">,</span> <span class="p">)</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'edit_post.html'</span>
    <span class="n">pk_url_kwarg</span> <span class="o">=</span> <span class="s">'post_pk'</span>
    <span class="n">context_object_name</span> <span class="o">=</span> <span class="s">'post'</span>

    <span class="k">def</span> <span class="nf">get_queryset</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">queryset</span> <span class="o">=</span> <span class="nb">super</span><span class="p">()</span><span class="o">.</span><span class="n">get_queryset</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">queryset</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">created_by</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">request</span><span class="o">.</span><span class="n">user</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">form_valid</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">form</span><span class="p">):</span>
        <span class="n">post</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
        <span class="n">post</span><span class="o">.</span><span class="n">updated_by</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">request</span><span class="o">.</span><span class="n">user</span>
        <span class="n">post</span><span class="o">.</span><span class="n">updated_at</span> <span class="o">=</span> <span class="n">timezone</span><span class="o">.</span><span class="n">now</span><span class="p">()</span>
        <span class="n">post</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'topic_posts'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">post</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">,</span> <span class="n">topic_pk</span><span class="o">=</span><span class="n">post</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span></code></pre></figure>

<p>With the line <code class="highlighter-rouge">queryset = super().get_queryset()</code> we are reusing the <code class="highlighter-rouge">get_queryset</code> method from the parent class, that
is, the <strong>UpateView</strong> class. Then, we are adding an extra filter to the queryset, which is filtering the post using
the logged in user, available inside the <strong>request</strong> object.</p>

<p>Test it again:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py test boards.tests.test_view_edit_post</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
...........
----------------------------------------------------------------------
Ran 11 tests in 1.321s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>All good!</p>

<hr />
