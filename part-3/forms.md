
<h4 id="forms">Forms</h4>

<p>Forms are used to deal with user input. It’s a very common task in any web application or website. The standard way
to do it is through HTML forms, where the user input some data, submit it to the server, and then the server does
something with it.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/Pixton_Comic_All_Input.png" alt="All input is evil" /></p>

<p>Form processing is a fairly complex task because it involves interacting with many layers of an application. There are
also many issues to take care of. For example, all data submitted to the server comes in a string format, so we have to
transform it into a proper data type (integer, float, date, etc.) before doing anything with it. We have to validate
the data regarding the business logic of the application. We also have to clean, sanitize the data properly so to avoid
security issues such as SQL Injection and XSS attacks.</p>

<p>Good news is that the Django Forms API makes the whole process a lot easier, automating a good chunk of this work. Also,
the final result is a much more secure code than most programmers would be able to implement by themselves. So, no
matter how simple the HTML form is, always use the forms API.</p>

<h5 id="how-not-implement-a-form">How Not Implement a Form</h5>

<p>At first, I thought about jumping straight to the forms API. But I think it would be a good idea for us to spend some
time trying to understand the underlying details of form processing. Otherwise, it will end up looking like magic,
which is a bad thing, because when things go wrong, you have no idea where to look for the problem.</p>

<p>With a deeper understanding of some programming concepts, we can feel more in control of the situation. Being in
control is important because it let us write code with more confidence. The moment we know exactly what is going on,
it’s much easier to implement a code of predictable behavior. It’s also a lot easier to debug and find errors because
you know where to look.</p>

<p>Anyway, let’s start by implementing the form below:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/wireframe-new-topic.png" alt="Wireframe New Topic" /></p>

<p>It’s one of the wireframes we drew in the previous tutorial. I now realize this may be a bad example to start because
this particular form involves processing data of two different models: <strong>Topic</strong> (subject) and <strong>Post</strong> (message).</p>

<p>There’s another important aspect that we haven’t discussed it so far, which is user authentication. We are only
supposed to show this screen for authenticated users. This way we can tell who created a <strong>Topic</strong> or a <strong>Post</strong>.</p>

<p>So let’s abstract some details for now and focus on understanding how to save user input in the database.</p>

<p>First thing, let’s create a new URL route named <strong>new_topic</strong>:</p>

<p><strong>myproject/urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">django.contrib</span> <span class="kn">import</span> <span class="n">admin</span>

<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">home</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">board_topics</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/new/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">new_topic</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'new_topic'</span><span class="p">),</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^admin/'</span><span class="p">,</span> <span class="n">admin</span><span class="o">.</span><span class="n">site</span><span class="o">.</span><span class="n">urls</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>The way we are building the URL will help us identify the correct <strong>Board</strong>.</p>

<p>Now let’s create the <strong>new_topic</strong> view function:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span><span class="p">,</span> <span class="n">get_object_or_404</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">new_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">})</span></code></pre></figure>

<p>For now, the <strong>new_topic</strong> view function is looking exactly the same as the <strong>board_topics</strong>. That’s on purpose, let’s
take a step at a time.</p>

<p>Now we just need a template named <strong>new_topic.html</strong> to see some code working:</p>

<p><strong>templates/new_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Start a New Topic<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>For now we just have the breadcrumb assuring the navigation. Observe that we included the URL back to the
<strong>board_topics</strong> view.</p>

<p>Open the URL <strong>http://127.0.0.1:8000/boards/1/new/</strong>. The result, for now, is the following page:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/start-a-new-topic.png" alt="Start a New Topic" /></p>

<p>We still haven’t implemented a way to reach this new page, but if we change the URL to
<strong>http://127.0.0.1:8000/boards/2/new/</strong>, it should take us to the <strong>Python Board</strong>:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/start-a-new-topic-python.png" alt="Start a New Topic" /></p>

<div class="info">
  <p>
    <strong><i class="fa fa-info-circle"></i> Note:</strong>
  </p>
  <p>
    The result may be different for you if you haven't followed the steps from the previous tutorial. In my case, I
    have three <strong>Board</strong> instances in the database, being Django = 1, Python = 2, and Random = 3. Those
    numbers are the IDs from the database, used from the URL to identify the right resource.
  </p>
</div>

<p>We can already add some tests:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">home</span><span class="p">,</span> <span class="n">board_topics</span><span class="p">,</span> <span class="n">new_topic</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">class</span> <span class="nc">HomeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">BoardTopicsTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

<span class="k">class</span> <span class="nc">NewTopicTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_new_topic_view_success_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_new_topic_view_not_found_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">99</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">404</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_new_topic_url_resolves_new_topic_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/boards/1/new/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="p">,</span> <span class="n">new_topic</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_new_topic_view_contains_link_back_to_board_topics_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">new_topic_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">board_topics_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">new_topic_url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="n">response</span><span class="p">,</span> <span class="s">'href="{0}"'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">board_topics_url</span><span class="p">))</span></code></pre></figure>

<p>A quick summary of the tests of our new class <strong>NewTopicTests</strong>:</p>

<ul>
  <li><strong>setUp</strong>: creates a <strong>Board</strong> instance to be used during the tests</li>
  <li><strong>test_new_topic_view_success_status_code</strong>: check if the request to the view is successful</li>
  <li><strong>test_new_topic_view_not_found_status_code</strong>: check if the view is raising a 404 error when the <strong>Board</strong> does not
exist</li>
  <li><strong>test_new_topic_url_resolves_new_topic_view</strong>: check if the right view is being used</li>
  <li><strong>test_new_topic_view_contains_link_back_to_board_topics_view</strong>: ensure the navigation back to the list of topics</li>
</ul>

<p>Run the tests:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
...........
----------------------------------------------------------------------
Ran 11 tests in 0.076s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>Good, now it’s time to start creating the form.</p>

<p><strong>templates/new_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Start a New Topic<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"form-group"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"id_subject"</span><span class="nt">&gt;</span>Subject<span class="nt">&lt;/label&gt;</span>
      <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"text"</span> <span class="na">class=</span><span class="s">"form-control"</span> <span class="na">id=</span><span class="s">"id_subject"</span> <span class="na">name=</span><span class="s">"subject"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"form-group"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"id_message"</span><span class="nt">&gt;</span>Message<span class="nt">&lt;/label&gt;</span>
      <span class="nt">&lt;textarea</span> <span class="na">class=</span><span class="s">"form-control"</span> <span class="na">id=</span><span class="s">"id_message"</span> <span class="na">name=</span><span class="s">"message"</span> <span class="na">rows=</span><span class="s">"5"</span><span class="nt">&gt;&lt;/textarea&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Post<span class="nt">&lt;/button&gt;</span>
  <span class="nt">&lt;/form&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>This is a raw HTML form created by hand using the CSS classes provided by Bootstrap 4. It looks like this:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/start-a-new-topic-form.png" alt="Start a New Topic" /></p>

<p>In the <code class="highlighter-rouge">&lt;form&gt;</code> tag, we have to define the <code class="highlighter-rouge">method</code> attribute. This instructs the browser on how
we want to communicate with the server. The HTTP spec defines several request methods (verbs). But for the most part,
we will only be using <strong>GET</strong> and <strong>POST</strong> request types.</p>

<p><strong>GET</strong> is perhaps the most common request type. It’s used to <em>retrieve</em> data from the server. Every time you click on
a link or type a URL directly into the browser, you are creating a <strong>GET</strong> request.</p>

<p><strong>POST</strong> is used when we want to change data on the server. So, generally speaking, every time we send data to the
server that will result in a change in the state of a resource, we should always send it via <strong>POST</strong> request.</p>

<p>Django protects all <strong>POST</strong> requests using a <strong>CSRF Token</strong> (Cross-Site Request Forgery Token). It’s a security
measure to avoid external sites or applications to submit data to our application. Every time the application receives
a <strong>POST</strong>, it will first look for the <strong>CSRF Token</strong>. If the request has no token, or the token is invalid, it will
discard the posted data.</p>

<p>The result of the <strong>csrf_token</strong> template tag:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span></code></pre></figure>

<p>Is a hidden field that’s submitted along with the other form data:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"hidden"</span> <span class="na">name=</span><span class="s">"csrfmiddlewaretoken"</span> <span class="na">value=</span><span class="s">"jG2o6aWj65YGaqzCpl0TYTg5jn6SctjzRZ9KmluifVx0IVaxlwh97YarZKs54Y32"</span><span class="nt">&gt;</span></code></pre></figure>

<p>Another thing, we have to set the <strong>name</strong> of the HTML inputs. The <strong>name</strong> will be used to retrieve the data on the
server side.</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"text"</span> <span class="na">class=</span><span class="s">"form-control"</span> <span class="na">id=</span><span class="s">"id_subject"</span> <span class="na">name=</span><span class="s">"subject"</span><span class="nt">&gt;</span>
<span class="nt">&lt;textarea</span> <span class="na">class=</span><span class="s">"form-control"</span> <span class="na">id=</span><span class="s">"id_message"</span> <span class="na">name=</span><span class="s">"message"</span> <span class="na">rows=</span><span class="s">"5"</span><span class="nt">&gt;&lt;/textarea&gt;</span></code></pre></figure>

<p>Here is how we retrieve the data:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">subject</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">[</span><span class="s">'subject'</span><span class="p">]</span>
<span class="n">message</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">[</span><span class="s">'message'</span><span class="p">]</span></code></pre></figure>

<p>So, a naïve implementation of a view that grabs the data from the HTML and starts a new topic can be written like this:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span><span class="p">,</span> <span class="n">redirect</span><span class="p">,</span> <span class="n">get_object_or_404</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Topic</span><span class="p">,</span> <span class="n">Post</span>

<span class="k">def</span> <span class="nf">new_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">subject</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">[</span><span class="s">'subject'</span><span class="p">]</span>
        <span class="n">message</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">[</span><span class="s">'message'</span><span class="p">]</span>

        <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">first</span><span class="p">()</span>  <span class="c"># TODO: get the currently logged in user</span>

        <span class="n">topic</span> <span class="o">=</span> <span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span>
            <span class="n">subject</span><span class="o">=</span><span class="n">subject</span><span class="p">,</span>
            <span class="n">board</span><span class="o">=</span><span class="n">board</span><span class="p">,</span>
            <span class="n">starter</span><span class="o">=</span><span class="n">user</span>
        <span class="p">)</span>

        <span class="n">post</span> <span class="o">=</span> <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span>
            <span class="n">message</span><span class="o">=</span><span class="n">message</span><span class="p">,</span>
            <span class="n">topic</span><span class="o">=</span><span class="n">topic</span><span class="p">,</span>
            <span class="n">created_by</span><span class="o">=</span><span class="n">user</span>
        <span class="p">)</span>

        <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span>  <span class="c"># TODO: redirect to the created topic page</span>

    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">})</span></code></pre></figure>

<p>This view is only considering the <em>happy path</em>, which is receiving the data and saving it into the database.
But there are some missing parts. We are not validating the data. The user could submit an empty form or a <strong>subject</strong>
that’s bigger than 255 characters.</p>

<p>So far we are hard-coding the <strong>User</strong> fields because we haven’t implemented the authentication yet. But there’s an
easy way to identify the logged in user. We will get to that part in the next tutorial. Also, we haven’t implemented
the view where we will list all the posts within a topic, so upon success, we are redirecting the user to the page
where we list all the board topics.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/start-a-new-topic-form-submit.png" alt="Start a New Topic" /></p>

<p>Submitted the form clicking on the <strong>Post</strong> button:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/topics-2.png" alt="Topics" /></p>

<p>It looks like it worked. But we haven’t implemented the topics listing yet, so there’s nothing to see here. Let’s edit
the <strong>templates/topics.html</strong> file to do a proper listing:</p>

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
  <span class="nt">&lt;table</span> <span class="na">class=</span><span class="s">"table"</span><span class="nt">&gt;</span>
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
      <span class="cp">{%</span> <span class="k">for</span> <span class="nv">topic</span> <span class="ow">in</span> <span class="nv">board.topics.all</span> <span class="cp">%}</span>
        <span class="nt">&lt;tr&gt;</span>
          <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.starter.username</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td&gt;</span>0<span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td&gt;</span>0<span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;td&gt;</span><span class="cp">{{</span> <span class="nv">topic.last_updated</span> <span class="cp">}}</span><span class="nt">&lt;/td&gt;</span>
        <span class="nt">&lt;/tr&gt;</span>
      <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
    <span class="nt">&lt;/tbody&gt;</span>
  <span class="nt">&lt;/table&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/topics-3.png" alt="Topics" /></p>

<p>Yep! The <strong>Topic</strong> we created is here.</p>

<p>Two new concepts here:</p>

<p>We are using for the first time the <strong>topics</strong> property in the <strong>Board</strong> model. The <strong>topics</strong> property is created
automatically by Django using a reverse relationship. In the previous steps, we created a <strong>Topic</strong> instance:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">def</span> <span class="nf">new_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>

    <span class="c"># ...</span>

    <span class="n">topic</span> <span class="o">=</span> <span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span>
        <span class="n">subject</span><span class="o">=</span><span class="n">subject</span><span class="p">,</span>
        <span class="n">board</span><span class="o">=</span><span class="n">board</span><span class="p">,</span>
        <span class="n">starter</span><span class="o">=</span><span class="n">user</span>
    <span class="p">)</span></code></pre></figure>

<p>In the line <code class="highlighter-rouge">board=board</code>, we set the <strong>board</strong> field in <strong>Topic</strong> model, which is a <code class="highlighter-rouge">ForeignKey(Board)</code>. With that,
now our <strong>Board</strong> instance is aware that it has an <strong>Topic</strong> instance associated with it.</p>

<p>The reason why we used <code class="highlighter-rouge">board.topics.all</code> instead of just <code class="highlighter-rouge">board.topics</code> is because <code class="highlighter-rouge">board.topics</code> is a
<strong>Related Manager</strong>, which is pretty much similar to a <strong>Model Manager</strong>, usually available on the <code class="highlighter-rouge">board.objects</code>
property. So, to return all topics associated with a given board, we have to run <code class="highlighter-rouge">board.topics.all()</code>. To filter some
data, we could do <code class="highlighter-rouge">board.topics.filter(subject__contains='Hello')</code>.</p>

<p>Another important thing to note is that, inside Python code, we have to use parenthesis: <code class="highlighter-rouge">board.topics.all()</code>,
because <code class="highlighter-rouge">all()</code> is a method. When writing code using the Django Template Language, in an HTML template file, we don’t
use parenthesis, so it’s just <code class="highlighter-rouge">board.topics.all</code>.</p>

<p>The second thing is that we are making use of a <code class="highlighter-rouge">ForeignKey</code>:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{{</span> <span class="nv">topic.starter.username</span> <span class="cp">}}</span></code></pre></figure>

<p>Just create a <em>path</em> through the property using dots. We can pretty much access any property of the <strong>User</strong> model.
If we wanted the user’s email, we could use <code class="highlighter-rouge">topic.starter.email</code>.</p>

<p>Since we are already modifying the <strong>topics.html</strong> template, let’s create the button that takes us to the <strong>new topic</strong>
screen:</p>

<p><strong>templates/topics.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'new_topic'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/div&gt;</span>

  <span class="nt">&lt;table</span> <span class="na">class=</span><span class="s">"table"</span><span class="nt">&gt;</span>
    <span class="c">&lt;!-- code suppressed for brevity --&gt;</span>
  <span class="nt">&lt;/table&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/topics-4.png" alt="Topics" /></p>

<p>We can include a test to make sure the user can reach the <strong>New topic</strong> view from this page:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">BoardTopicsTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">test_board_topics_view_contains_navigation_links</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">board_topics_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">homepage_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">)</span>
        <span class="n">new_topic_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>

        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">board_topics_url</span><span class="p">)</span>

        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="n">response</span><span class="p">,</span> <span class="s">'href="{0}"'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">homepage_url</span><span class="p">))</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="n">response</span><span class="p">,</span> <span class="s">'href="{0}"'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">new_topic_url</span><span class="p">))</span></code></pre></figure>

<p>Basically here I renamed the old <strong>test_board_topics_view_contains_link_back_to_homepage</strong> method and add an extra
<code class="highlighter-rouge">assertContains</code>. This test is now responsible for making sure our view contains the required navigation links.</p>

<h5 id="testing-the-form-view">Testing The Form View</h5>

<p>Before we code the previous form example in a Django way, let’s write some tests for the form processing:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="s">''' new imports below '''</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">.views</span> <span class="kn">import</span> <span class="n">new_topic</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Topic</span><span class="p">,</span> <span class="n">Post</span>

<span class="k">class</span> <span class="nc">NewTopicTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'Django board.'</span><span class="p">)</span>
        <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'123'</span><span class="p">)</span>  <span class="c"># &lt;- included this line here</span>

    <span class="c"># ...</span>

    <span class="k">def</span> <span class="nf">test_csrf</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="n">response</span><span class="p">,</span> <span class="s">'csrfmiddlewaretoken'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_new_topic_valid_post_data</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">data</span> <span class="o">=</span> <span class="p">{</span>
            <span class="s">'subject'</span><span class="p">:</span> <span class="s">'Test title'</span><span class="p">,</span>
            <span class="s">'message'</span><span class="p">:</span> <span class="s">'Lorem ipsum dolor sit amet'</span>
        <span class="p">}</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">data</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">exists</span><span class="p">())</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">exists</span><span class="p">())</span>

    <span class="k">def</span> <span class="nf">test_new_topic_invalid_post_data</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        Invalid post data should not redirect
        The expected behavior is to show the form again with validation errors
        '''</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="p">{})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_new_topic_invalid_post_data_empty_fields</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        Invalid post data should not redirect
        The expected behavior is to show the form again with validation errors
        '''</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">data</span> <span class="o">=</span> <span class="p">{</span>
            <span class="s">'subject'</span><span class="p">:</span> <span class="s">''</span><span class="p">,</span>
            <span class="s">'message'</span><span class="p">:</span> <span class="s">''</span>
        <span class="p">}</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">data</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertFalse</span><span class="p">(</span><span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">exists</span><span class="p">())</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertFalse</span><span class="p">(</span><span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">exists</span><span class="p">())</span></code></pre></figure>

<p>First thing, the <strong>tests.py</strong> file is already starting to get big. We will improve it soon, breaking the tests into
several files. But for now, let’s keep working on it.</p>

<ul>
  <li><strong>setUp</strong>: included the <code class="highlighter-rouge">User.objects.create_user</code> to create a <strong>User</strong> instance to be used in the tests</li>
  <li><strong>test_csrf</strong>: since the <strong>CSRF Token</strong> is a fundamental part of processing <strong>POST</strong> requests, we have to make
sure our HTML contains the token.</li>
  <li><strong>test_new_topic_valid_post_data</strong>: sends a valid combination of data and check if the view created a <strong>Topic</strong>
instance and a <strong>Post</strong> instance.</li>
  <li><strong>test_new_topic_invalid_post_data</strong>: here we are sending an empty dictionary to check how the application is
behaving.</li>
  <li><strong>test_new_topic_invalid_post_data_empty_fields</strong>: similar to the previous test, but this time we are sending some
data. The application is expected to validate and reject empty subject and message.</li>
</ul>

<p>Let’s run the tests:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
........EF.....
======================================================================
ERROR: test_new_topic_invalid_post_data (boards.tests.NewTopicTests)
----------------------------------------------------------------------
Traceback (most recent call last):
...
django.utils.datastructures.MultiValueDictKeyError: "'subject'"

======================================================================
FAIL: test_new_topic_invalid_post_data_empty_fields (boards.tests.NewTopicTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/vitorfs/Development/myproject/django-beginners-guide/boards/tests.py", line 115, in test_new_topic_invalid_post_data_empty_fields
    self.assertEquals(response.status_code, 200)
AssertionError: 302 != 200

----------------------------------------------------------------------
Ran 15 tests in 0.512s

FAILED (failures=1, errors=1)
Destroying test database for alias 'default'...</code></pre></figure>

<p>We have one failing test and one error. Both related to invalid user input. Instead of trying to fix it with the
current implementation, let’s make those tests pass using the Django Forms API.</p>

<h4 id="creating-forms-the-right-way">Creating Forms The Right Way</h4>

<p>So, we came a long way since we started working with Forms. Finally, it’s time to use the Forms API.</p>

<p>The Forms API is available in the module <code class="highlighter-rouge">django.forms</code>. Django works with two types of forms: <code class="highlighter-rouge">forms.Form</code> and
<code class="highlighter-rouge">forms.ModelForm</code>. The <code class="highlighter-rouge">Form</code> class is a general purpose form implementation. We can use it to process data that are
not directly associated with a model in our application. A <code class="highlighter-rouge">ModelForm</code> is a subclass of <code class="highlighter-rouge">Form</code>, and it’s associated
with a model class.</p>

<p>Let’s create a new file named <strong>forms.py</strong> inside the <strong>boards</strong>’ folder:</p>

<p><strong>boards/forms.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">forms</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Topic</span>

<span class="k">class</span> <span class="nc">NewTopicForm</span><span class="p">(</span><span class="n">forms</span><span class="o">.</span><span class="n">ModelForm</span><span class="p">):</span>
    <span class="n">message</span> <span class="o">=</span> <span class="n">forms</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">widget</span><span class="o">=</span><span class="n">forms</span><span class="o">.</span><span class="n">Textarea</span><span class="p">(),</span> <span class="n">max_length</span><span class="o">=</span><span class="mi">4000</span><span class="p">)</span>

    <span class="k">class</span> <span class="nc">Meta</span><span class="p">:</span>
        <span class="n">model</span> <span class="o">=</span> <span class="n">Topic</span>
        <span class="n">fields</span> <span class="o">=</span> <span class="p">[</span><span class="s">'subject'</span><span class="p">,</span> <span class="s">'message'</span><span class="p">]</span></code></pre></figure>

<p>This is our first form. It’s a <code class="highlighter-rouge">ModelForm</code> associated with the <strong>Topic</strong> model. The <code class="highlighter-rouge">subject</code> in the <code class="highlighter-rouge">fields</code> list
inside the <strong>Meta</strong> class is referring to the <code class="highlighter-rouge">subject</code> field in the <strong>Topic</strong> class. Now observe that we are
defining an extra field named <code class="highlighter-rouge">message</code>. This refers to the message in the <strong>Post</strong> we want to save.</p>

<p>Now we have to refactor our <strong>views.py</strong>:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">render</span><span class="p">,</span> <span class="n">redirect</span><span class="p">,</span> <span class="n">get_object_or_404</span>
<span class="kn">from</span> <span class="nn">.forms</span> <span class="kn">import</span> <span class="n">NewTopicForm</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Topic</span><span class="p">,</span> <span class="n">Post</span>

<span class="k">def</span> <span class="nf">new_topic</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">first</span><span class="p">()</span>  <span class="c"># TODO: get the currently logged in user</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">NewTopicForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">topic</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="n">commit</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">board</span> <span class="o">=</span> <span class="n">board</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">starter</span> <span class="o">=</span> <span class="n">user</span>
            <span class="n">topic</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="n">post</span> <span class="o">=</span> <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span>
                <span class="n">message</span><span class="o">=</span><span class="n">form</span><span class="o">.</span><span class="n">cleaned_data</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'message'</span><span class="p">),</span>
                <span class="n">topic</span><span class="o">=</span><span class="n">topic</span><span class="p">,</span>
                <span class="n">created_by</span><span class="o">=</span><span class="n">user</span>
            <span class="p">)</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span>  <span class="c"># TODO: redirect to the created topic page</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">NewTopicForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">,</span> <span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>This is how we use the forms in a view. Let me remove the extra noise so we can focus on the core of the form
processing:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
    <span class="n">form</span> <span class="o">=</span> <span class="n">NewTopicForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
        <span class="n">topic</span> <span class="o">=</span> <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'board_topics'</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">board</span><span class="o">.</span><span class="n">pk</span><span class="p">)</span>
<span class="k">else</span><span class="p">:</span>
    <span class="n">form</span> <span class="o">=</span> <span class="n">NewTopicForm</span><span class="p">()</span>
<span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_topic.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>First we check if the request is a <strong>POST</strong> or a <strong>GET</strong>. If the request came from a <strong>POST</strong>, it means the user is
submitting some data to the server. So we instantiate a form instance passing the <strong>POST</strong> data to the form:
<code class="highlighter-rouge">form = NewTopicForm(request.POST)</code>.</p>

<p>Then, we ask Django to verify the data, check if the form is valid if we can save it in the database: <code class="highlighter-rouge">if form.is_valid():</code>.
If the form was valid, we proceed to save the data in the database using <code class="highlighter-rouge">form.save()</code>. The <code class="highlighter-rouge">save()</code> method returns
an instance of the Model saved into the database. So, since this is a <strong>Topic</strong> form, it will return the <strong>Topic</strong>
that was created: <code class="highlighter-rouge">topic = form.save()</code>. After that, the common path is to redirect the user somewhere else, both to
avoid the user re-submitting the form by pressing F5 and also to keep the flow of the application.</p>

<p>Now, if the data was invalid, Django will add a list of errors to the form. After that, the view does nothing and
returns in the last statement: <code class="highlighter-rouge">return render(request, 'new_topic.html', {'form': form})</code>. That means we have to
update the <strong>new_topic.html</strong> to display errors properly.</p>

<p>If the request was a <strong>GET</strong>, we just initialize a new and empty form using <code class="highlighter-rouge">form = NewTopicForm()</code>.</p>

<p>Let’s run the tests and see how is everything:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py <span class="nb">test</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Creating test database for alias 'default'...
System check identified no issues (0 silenced).
...............
----------------------------------------------------------------------
Ran 15 tests in 0.522s

OK
Destroying test database for alias 'default'...</code></pre></figure>

<p>We even fixed the last two tests.</p>

<p>The Django Forms API does much more than processing and validating the data. It also generates the HTML for us.</p>

<p>Let’s update the <strong>new_topic.html</strong> template to fully use the Django Forms API:</p>

<p><strong>templates/new_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Start a New Topic<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
    <span class="cp">{{</span> <span class="nv">form.as_p</span> <span class="cp">}}</span>
    <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Post<span class="nt">&lt;/button&gt;</span>
  <span class="nt">&lt;/form&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>The <code class="highlighter-rouge">form</code> have three rendering options: <code class="highlighter-rouge">form.as_table</code>, <code class="highlighter-rouge">form.as_ul</code>, and <code class="highlighter-rouge">form.as_p</code>. It’s a quick way to render
all the fields of a form. As the name suggests, the <code class="highlighter-rouge">as_table</code> uses table tags to format the inputs, the <code class="highlighter-rouge">as_ul</code>
creates an HTML list of inputs, etc.</p>

<p>Let’s see how it looks like:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/start-a-new-topic-form-django.png" alt="Start a New Topic" /></p>

<p>Well, our previous form was looking better, right? We are going to fix it in a moment.</p>

<p>It can look broken right now but trust me; there’s a lot of things behind it right now. And it’s extremely powerful.
For example, if our form had 50 fields, we could render all the fields just by typing <code class="highlighter-rouge"><span class="p">{</span><span class="err">{</span><span class="w"> </span><span class="err">form.as_p</span><span class="w"> </span><span class="p">}</span><span class="err">}</span></code>.</p>

<p>And more, using the Forms API, Django will validate the data and add error messages to each field. Let’s try submitting
an empty form:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/form-validation.png" alt="Form Validation" /></p>

<div class="info">
  <p>
    <strong><i class="fa fa-info-circle"></i> Note:</strong>
  </p>
  <p>
    If you see something like this:
    <img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/novalidate.png" alt="Please fill out this field." style="display: inline!important;" />
    when you submit the form, that's not Django. It's your browser doing a pre-validation. To disable it add the
    <code>novalidate</code> attribute to your form tag: <code>&lt;form method="post" novalidate&gt;</code>
  </p>
  <p>
    You can keep it; there's no problem with it. It's just because our form is very simple right now, and we don't
    have much data validation to see.
  </p>
  <p>
    Another important thing to note is that: there is no such a thing as "client-side validation." JavaScript
    validation or browser validation is just for <strong>usability</strong> purpose. And also to reduce the number of
    requests to the server. Data validation should always be done on the server side, where we have full control over
    the data.
  </p>
</div>

<p>It also handles help texts, which can be defined both in a <strong>Form</strong> class or in a <strong>Model</strong> class:</p>

<p><strong>boards/forms.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">forms</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Topic</span>

<span class="k">class</span> <span class="nc">NewTopicForm</span><span class="p">(</span><span class="n">forms</span><span class="o">.</span><span class="n">ModelForm</span><span class="p">):</span>
    <span class="n">message</span> <span class="o">=</span> <span class="n">forms</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span>
        <span class="n">widget</span><span class="o">=</span><span class="n">forms</span><span class="o">.</span><span class="n">Textarea</span><span class="p">(),</span>
        <span class="n">max_length</span><span class="o">=</span><span class="mi">4000</span><span class="p">,</span>
        <span class="n">help_text</span><span class="o">=</span><span class="s">'The max length of the text is 4000.'</span>
    <span class="p">)</span>

    <span class="k">class</span> <span class="nc">Meta</span><span class="p">:</span>
        <span class="n">model</span> <span class="o">=</span> <span class="n">Topic</span>
        <span class="n">fields</span> <span class="o">=</span> <span class="p">[</span><span class="s">'subject'</span><span class="p">,</span> <span class="s">'message'</span><span class="p">]</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/help-text.png" alt="Help Text" /></p>

<p>We can also set extra attributes to a form field:</p>

<p><strong>boards/forms.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django</span> <span class="kn">import</span> <span class="n">forms</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Topic</span>

<span class="k">class</span> <span class="nc">NewTopicForm</span><span class="p">(</span><span class="n">forms</span><span class="o">.</span><span class="n">ModelForm</span><span class="p">):</span>
    <span class="n">message</span> <span class="o">=</span> <span class="n">forms</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span>
        <span class="n">widget</span><span class="o">=</span><span class="n">forms</span><span class="o">.</span><span class="n">Textarea</span><span class="p">(</span>
            <span class="n">attrs</span><span class="o">=</span><span class="p">{</span><span class="s">'rows'</span><span class="p">:</span> <span class="mi">5</span><span class="p">,</span> <span class="s">'placeholder'</span><span class="p">:</span> <span class="s">'What is on your mind?'</span><span class="p">}</span>
        <span class="p">),</span>
        <span class="n">max_length</span><span class="o">=</span><span class="mi">4000</span><span class="p">,</span>
        <span class="n">help_text</span><span class="o">=</span><span class="s">'The max length of the text is 4000.'</span>
    <span class="p">)</span>

    <span class="k">class</span> <span class="nc">Meta</span><span class="p">:</span>
        <span class="n">model</span> <span class="o">=</span> <span class="n">Topic</span>
        <span class="n">fields</span> <span class="o">=</span> <span class="p">[</span><span class="s">'subject'</span><span class="p">,</span> <span class="s">'message'</span><span class="p">]</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/form-placeholder.png" alt="Form Placeholder" /></p>

<h5 id="rendering-bootstrap-forms">Rendering Bootstrap Forms</h5>

<p>Alright, so let’s make things pretty again.</p>

<p>When working with Bootstrap or any other Front-End library, I like to use a Django package called
<strong>django-widget-tweaks</strong>. It gives us more control over the rendering process, keeping the defaults and just adding
extra customizations on top of it.</p>

<p>Let’s start off by installing it:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">pip install django-widget-tweaks</code></pre></figure>

<p>Now add it to the <code class="highlighter-rouge">INSTALLED_APPS</code>:</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">INSTALLED_APPS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="s">'django.contrib.admin'</span><span class="p">,</span>
    <span class="s">'django.contrib.auth'</span><span class="p">,</span>
    <span class="s">'django.contrib.contenttypes'</span><span class="p">,</span>
    <span class="s">'django.contrib.sessions'</span><span class="p">,</span>
    <span class="s">'django.contrib.messages'</span><span class="p">,</span>
    <span class="s">'django.contrib.staticfiles'</span><span class="p">,</span>

    <span class="s">'widget_tweaks'</span><span class="p">,</span>

    <span class="s">'boards'</span><span class="p">,</span>
<span class="p">]</span></code></pre></figure>

<p>Now let’s take it into use:</p>

<p><strong>templates/new_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="nv">load</span> <span class="nv">widget_tweaks</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Start a New Topic<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>

    <span class="cp">{%</span> <span class="k">for</span> <span class="nv">field</span> <span class="ow">in</span> <span class="nv">form</span> <span class="cp">%}</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"form-group"</span><span class="nt">&gt;</span>
        <span class="cp">{{</span> <span class="nv">field.label_tag</span> <span class="cp">}}</span>

        <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control"</span> <span class="cp">%}</span>

        <span class="cp">{%</span> <span class="k">if</span> <span class="nv">field.help_text</span> <span class="cp">%}</span>
          <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"form-text text-muted"</span><span class="nt">&gt;</span>
            <span class="cp">{{</span> <span class="nv">field.help_text</span> <span class="cp">}}</span>
          <span class="nt">&lt;/small&gt;</span>
        <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

    <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Post<span class="nt">&lt;/button&gt;</span>
  <span class="nt">&lt;/form&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/bootstrap-form.png" alt="Bootstrap Form" /></p>

<p>There it is! So, here we are using the <strong>django-widget-tweaks</strong>. First, we load it in the template by using the
<code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">load</span><span class="w"> </span><span class="err">widget_tweaks</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> template tag. Then the usage:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control"</span> <span class="cp">%}</span></code></pre></figure>

<p>The <code class="highlighter-rouge">render_field</code> tag is not part of Django; it lives inside the package we installed. To use it we have to pass a
form field instance as the first parameter, and then after we can add arbitrary HTML attributes to complement it. It
will be useful because then we can assign classes based on certain conditions.</p>

<p>Some examples of the <code class="highlighter-rouge">render_field</code> template tag:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">form.subject</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control"</span> <span class="cp">%}</span>
<span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">form.message</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control"</span> <span class="nv">placeholder</span><span class="err">=</span><span class="nv">form.message.label</span> <span class="cp">%}</span>
<span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control"</span> <span class="nv">placeholder</span><span class="err">=</span><span class="s2">"Write a message!"</span> <span class="cp">%}</span>
<span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">style</span><span class="err">=</span><span class="s2">"font-size: 20px"</span> <span class="cp">%}</span></code></pre></figure>

<p>Now to implement the Bootstrap 4 validation tags, we can change the <strong>new_topic.html</strong> template:</p>

<p><strong>templates/new_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
  <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>

  <span class="cp">{%</span> <span class="k">for</span> <span class="nv">field</span> <span class="ow">in</span> <span class="nv">form</span> <span class="cp">%}</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"form-group"</span><span class="nt">&gt;</span>
      <span class="cp">{{</span> <span class="nv">field.label_tag</span> <span class="cp">}}</span>

      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">form.is_bound</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">if</span> <span class="nv">field.errors</span> <span class="cp">%}</span>

          <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control is-invalid"</span> <span class="cp">%}</span>
          <span class="cp">{%</span> <span class="k">for</span> <span class="nv">error</span> <span class="ow">in</span> <span class="nv">field.errors</span> <span class="cp">%}</span>
            <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"invalid-feedback"</span><span class="nt">&gt;</span>
              <span class="cp">{{</span> <span class="nv">error</span> <span class="cp">}}</span>
            <span class="nt">&lt;/div&gt;</span>
          <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

        <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
          <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control is-valid"</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control"</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">field.help_text</span> <span class="cp">%}</span>
        <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"form-text text-muted"</span><span class="nt">&gt;</span>
          <span class="cp">{{</span> <span class="nv">field.help_text</span> <span class="cp">}}</span>
        <span class="nt">&lt;/small&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

  <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Post<span class="nt">&lt;/button&gt;</span>
<span class="nt">&lt;/form&gt;</span></code></pre></figure>

<p>The result is this:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/bootstrap-invalid-1.png" alt="Bootstrap Form Invalid" /></p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-3/bootstrap-invalid-2.png" alt="Bootstrap Form Partially Valid" /></p>

<p>So, we have three different rendering states:</p>

<ul>
  <li><strong>Initial state</strong>: the form has no data (is not bound)</li>
  <li><strong>Invalid</strong>: we add the <code class="highlighter-rouge">.is-invalid</code> CSS class and add error messages in an element with a class
<code class="highlighter-rouge">.invalid-feedback</code>. The form field and the messages are rendered in red.</li>
  <li><strong>Valid</strong>: we add the <code class="highlighter-rouge">.is-valid</code> CSS class so to paint the form field in green, giving feedback to the user that
this field is good to go.</li>
</ul>

<h5 id="reusable-forms-templates">Reusable Forms Templates</h5>

<p>The template code looks a little bit complicated, right? Well, the good news is that we can reuse this snippet across
the project.</p>

<p>In the <strong>templates</strong> folder, create a new folder named <strong>includes</strong>:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- boards/
 |    |-- myproject/
 |    |-- templates/
 |    |    |-- includes/    &lt;-- here!
 |    |    |-- base.html
 |    |    |-- home.html
 |    |    |-- new_topic.html
 |    |    +-- topics.html
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>Now inside the <strong>includes</strong> folder, create a file named <strong>form.html</strong>:</p>

<p><strong>templates/includes/form.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">widget_tweaks</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">for</span> <span class="nv">field</span> <span class="ow">in</span> <span class="nv">form</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"form-group"</span><span class="nt">&gt;</span>
    <span class="cp">{{</span> <span class="nv">field.label_tag</span> <span class="cp">}}</span>

    <span class="cp">{%</span> <span class="k">if</span> <span class="nv">form.is_bound</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">field.errors</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control is-invalid"</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">for</span> <span class="nv">error</span> <span class="ow">in</span> <span class="nv">field.errors</span> <span class="cp">%}</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"invalid-feedback"</span><span class="nt">&gt;</span>
            <span class="cp">{{</span> <span class="nv">error</span> <span class="cp">}}</span>
          <span class="nt">&lt;/div&gt;</span>
        <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control is-valid"</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="nv">render_field</span> <span class="nv">field</span> <span class="nv">class</span><span class="err">=</span><span class="s2">"form-control"</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

    <span class="cp">{%</span> <span class="k">if</span> <span class="nv">field.help_text</span> <span class="cp">%}</span>
      <span class="nt">&lt;small</span> <span class="na">class=</span><span class="s">"form-text text-muted"</span><span class="nt">&gt;</span>
        <span class="cp">{{</span> <span class="nv">field.help_text</span> <span class="cp">}}</span>
      <span class="nt">&lt;/small&gt;</span>
    <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span></code></pre></figure>

<p>Now we change our <strong>new_topic.html</strong> template:</p>

<p><strong>templates/new_topic.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Start a New Topic<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Boards<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'board_topics'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">board.name</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
    <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
    <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Post<span class="nt">&lt;/button&gt;</span>
  <span class="nt">&lt;/form&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>As the name suggests, the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">include</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> is used to <em>include</em> HTML templates in another
template. It’s a very useful way to reuse HTML components in a project.</p>

<p>The next form we implement, we can simply use <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">include</span><span class="w"> </span><span class="err">'includes/form.html'</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code> to render it.</p>

<h5 id="adding-more-tests">Adding More Tests</h5>

<p>Now we are using Django Forms; we can add more tests to make sure it is running smoothly:</p>

<p><strong>boards/tests.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="c"># ... other imports</span>
<span class="kn">from</span> <span class="nn">.forms</span> <span class="kn">import</span> <span class="n">NewTopicForm</span>

<span class="k">class</span> <span class="nc">NewTopicTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ... other tests</span>

    <span class="k">def</span> <span class="nf">test_contains_form</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>  <span class="c"># &lt;- new test</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIsInstance</span><span class="p">(</span><span class="n">form</span><span class="p">,</span> <span class="n">NewTopicForm</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_new_topic_invalid_post_data</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>  <span class="c"># &lt;- updated this one</span>
        <span class="s">'''
        Invalid post data should not redirect
        The expected behavior is to show the form again with validation errors
        '''</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'new_topic'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'pk'</span><span class="p">:</span> <span class="mi">1</span><span class="p">})</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="p">{})</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">form</span><span class="o">.</span><span class="n">errors</span><span class="p">)</span></code></pre></figure>

<p>Now we are using the <code class="highlighter-rouge">assertIsInstance</code> method for the first time. Basically we are grabbing the form instance in the
context data, and checking if it is a <code class="highlighter-rouge">NewTopicForm</code>. In the last test, we added the <code class="highlighter-rouge">self.assertTrue(form.errors)</code>
to make sure the form is showing errors when the data is invalid.</p>

<hr />
