<h4 id="list-view">List View</h4>

<p>We could refactor some of our existing views to take advantage of the CBV capabilities. Take the home page for example.
We are just grabbing all the boards from the database and listing it in the HTML:</p>

<p><strong>boards/views.py</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django">from django.shortcuts import render
from .models import Board

def home(request):
    boards = Board.objects.all()
    return render(request, 'home.html', {'boards': boards})</code></pre></figure>

<p>Here is how we could rewrite it using a GCBV for models listing:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/5e248d9a4499e2a796c6ffee9cbb1125#file-views-py-L12" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">ListView</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">class</span> <span class="nc">BoardListView</span><span class="p">(</span><span class="n">ListView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Board</span>
    <span class="n">context_object_name</span> <span class="o">=</span> <span class="s">'boards'</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'home.html'</span></code></pre></figure>

<p>Then we have to change the reference in the <strong>urls.py</strong> module:</p>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/a0a826233d95a4cc53a75afc441db1e9#file-urls-py-L10" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">BoardListView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'home'</span><span class="p">),</span>
    <span class="c"># ...</span>
<span class="p">]</span></code></pre></figure>

<p>If we check the homepage we will see that nothing really changed, everything is working as expected. But we have to
tweak our tests a little bit because now we are dealing with a class-based view:</p>

<p><strong>boards/tests/test_view_home.py</strong>
<small><a href="https://gist.github.com/vitorfs/e17216ce3d92110cf7e005ce3288c587" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">BoardListView</span>

<span class="k">class</span> <span class="nc">HomeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>
    <span class="k">def</span> <span class="nf">test_home_url_resolves_home_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="o">.</span><span class="n">view_class</span><span class="p">,</span> <span class="n">BoardListView</span><span class="p">)</span></code></pre></figure>

<h5 id="pagination">Pagination</h5>

<p>We can very easily implement pagination with class-based views. But first I wanted to do a pagination by hand, so that
we can explore better the mechanics behind it, so it doesn’t look like magic.</p>

<p>It wouldn’t really make sense to paginate the boards listing view because we do not expect to have many boards. But
definitely the topics listing and the posts listing need some pagination.</p>

<p>From now on, we will be working on the <strong>board_topics</strong> view.</p>

<p>First, let’s add some volume of posts. We could just use the application’s user interface and add several posts, or
open the Python shell and write a small script to do it for us:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py shell</code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">boards.models</span> <span class="kn">import</span> <span class="n">Board</span><span class="p">,</span> <span class="n">Topic</span><span class="p">,</span> <span class="n">Post</span>

<span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">first</span><span class="p">()</span>

<span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">)</span>

<span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="mi">100</span><span class="p">):</span>
    <span class="n">subject</span> <span class="o">=</span> <span class="s">'Topic test #{}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
    <span class="n">topic</span> <span class="o">=</span> <span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">subject</span><span class="o">=</span><span class="n">subject</span><span class="p">,</span> <span class="n">board</span><span class="o">=</span><span class="n">board</span><span class="p">,</span> <span class="n">starter</span><span class="o">=</span><span class="n">user</span><span class="p">)</span>
    <span class="n">Post</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">message</span><span class="o">=</span><span class="s">'Lorem ipsum...'</span><span class="p">,</span> <span class="n">topic</span><span class="o">=</span><span class="n">topic</span><span class="p">,</span> <span class="n">created_by</span><span class="o">=</span><span class="n">user</span><span class="p">)</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/topics.png" alt="Topics" /></p>

<p>Good, now we have some data to play with.</p>

<p>Before we jump into the code, let’s experiment a little bit more with the Python shell:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">python manage.py shell</code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">boards.models</span> <span class="kn">import</span> <span class="n">Topic</span>

<span class="c"># All the topics in the app</span>
<span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">107</span>

<span class="c"># Just the topics in the Django board</span>
<span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">board__name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">)</span><span class="o">.</span><span class="n">count</span><span class="p">()</span>
<span class="mi">104</span>

<span class="c"># Let's save this queryset into a variable to paginate it</span>
<span class="n">queryset</span> <span class="o">=</span> <span class="n">Topic</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">filter</span><span class="p">(</span><span class="n">board__name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">)</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-last_updated'</span><span class="p">)</span></code></pre></figure>

<p>It’s very important always define an ordering to a <strong>QuerySet</strong> you are going to paginate! Otherwise, it can give you
inconsistent results.</p>

<p>Now let’s import the <strong>Paginator</strong> utility:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.core.paginator</span> <span class="kn">import</span> <span class="n">Paginator</span>

<span class="n">paginator</span> <span class="o">=</span> <span class="n">Paginator</span><span class="p">(</span><span class="n">queryset</span><span class="p">,</span> <span class="mi">20</span><span class="p">)</span></code></pre></figure>

<p>Here we are telling Django to paginate our <strong>QuerySet</strong> in pages of 20 each. Now let’s explore some of the paginator
properties:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="c"># count the number of elements in the paginator</span>
<span class="n">paginator</span><span class="o">.</span><span class="n">count</span>
<span class="mi">104</span>

<span class="c"># total number of pages</span>
<span class="c"># 104 elements, paginating 20 per page gives you 6 pages</span>
<span class="c"># where the last page will have only 4 elements</span>
<span class="n">paginator</span><span class="o">.</span><span class="n">num_pages</span>
<span class="mi">6</span>

<span class="c"># range of pages that can be used to iterate and create the</span>
<span class="c"># links to the pages in the template</span>
<span class="n">paginator</span><span class="o">.</span><span class="n">page_range</span>
<span class="nb">range</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">7</span><span class="p">)</span>

<span class="c"># returns a Page instance</span>
<span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span>
<span class="o">&lt;</span><span class="n">Page</span> <span class="mi">2</span> <span class="n">of</span> <span class="mi">6</span><span class="o">&gt;</span>

<span class="n">page</span> <span class="o">=</span> <span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span>

<span class="nb">type</span><span class="p">(</span><span class="n">page</span><span class="p">)</span>
<span class="n">django</span><span class="o">.</span><span class="n">core</span><span class="o">.</span><span class="n">paginator</span><span class="o">.</span><span class="n">Page</span>

<span class="nb">type</span><span class="p">(</span><span class="n">paginator</span><span class="p">)</span>
<span class="n">django</span><span class="o">.</span><span class="n">core</span><span class="o">.</span><span class="n">paginator</span><span class="o">.</span><span class="n">Paginator</span></code></pre></figure>

<p>Here we have to pay attention because if we try to get a page that doesn’t exist, the Paginator will throw an
exception:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="mi">7</span><span class="p">)</span>
<span class="n">EmptyPage</span><span class="p">:</span> <span class="n">That</span> <span class="n">page</span> <span class="n">contains</span> <span class="n">no</span> <span class="n">results</span></code></pre></figure>

<p>Or if we try to pass an arbitrary parameter, which is not a page number:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="s">'abc'</span><span class="p">)</span>
<span class="n">PageNotAnInteger</span><span class="p">:</span> <span class="n">That</span> <span class="n">page</span> <span class="n">number</span> <span class="ow">is</span> <span class="ow">not</span> <span class="n">an</span> <span class="n">integer</span></code></pre></figure>

<p>We have to keep those details in mind when designing the user interface.</p>

<p>Now let’s explore the attributes and methods offered by the <strong>Page</strong> class a little bit:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">page</span> <span class="o">=</span> <span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>

<span class="c"># Check if there is another page after this one</span>
<span class="n">page</span><span class="o">.</span><span class="n">has_next</span><span class="p">()</span>
<span class="bp">True</span>

<span class="c"># If there is no previous page, that means this one is the first page</span>
<span class="n">page</span><span class="o">.</span><span class="n">has_previous</span><span class="p">()</span>
<span class="bp">False</span>

<span class="n">page</span><span class="o">.</span><span class="n">has_other_pages</span><span class="p">()</span>
<span class="bp">True</span>

<span class="n">page</span><span class="o">.</span><span class="n">next_page_number</span><span class="p">()</span>
<span class="mi">2</span>

<span class="c"># Take care here, since there is no previous page,</span>
<span class="c"># if we call the method `previous_page_number() we will get an exception:</span>
<span class="n">page</span><span class="o">.</span><span class="n">previous_page_number</span><span class="p">()</span>
<span class="n">EmptyPage</span><span class="p">:</span> <span class="n">That</span> <span class="n">page</span> <span class="n">number</span> <span class="ow">is</span> <span class="n">less</span> <span class="n">than</span> <span class="mi">1</span></code></pre></figure>

<h5 id="fbv-pagination">FBV Pagination</h5>

<p>Here is how we do it using regular function-based views:</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/16f0ac257439245fa6645af259d8846f#file-views-py-L19" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.db.models</span> <span class="kn">import</span> <span class="n">Count</span>
<span class="kn">from</span> <span class="nn">django.core.paginator</span> <span class="kn">import</span> <span class="n">Paginator</span><span class="p">,</span> <span class="n">EmptyPage</span><span class="p">,</span> <span class="n">PageNotAnInteger</span>
<span class="kn">from</span> <span class="nn">django.shortcuts</span> <span class="kn">import</span> <span class="n">get_object_or_404</span><span class="p">,</span> <span class="n">render</span>
<span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">ListView</span>
<span class="kn">from</span> <span class="nn">.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="k">def</span> <span class="nf">board_topics</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">pk</span><span class="p">):</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="n">pk</span><span class="p">)</span>
    <span class="n">queryset</span> <span class="o">=</span> <span class="n">board</span><span class="o">.</span><span class="n">topics</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-last_updated'</span><span class="p">)</span><span class="o">.</span><span class="n">annotate</span><span class="p">(</span><span class="n">replies</span><span class="o">=</span><span class="n">Count</span><span class="p">(</span><span class="s">'posts'</span><span class="p">)</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">page</span> <span class="o">=</span> <span class="n">request</span><span class="o">.</span><span class="n">GET</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'page'</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>

    <span class="n">paginator</span> <span class="o">=</span> <span class="n">Paginator</span><span class="p">(</span><span class="n">queryset</span><span class="p">,</span> <span class="mi">20</span><span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="n">topics</span> <span class="o">=</span> <span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="n">page</span><span class="p">)</span>
    <span class="k">except</span> <span class="n">PageNotAnInteger</span><span class="p">:</span>
        <span class="c"># fallback to the first page</span>
        <span class="n">topics</span> <span class="o">=</span> <span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
    <span class="k">except</span> <span class="n">EmptyPage</span><span class="p">:</span>
        <span class="c"># probably the user tried to add a page number</span>
        <span class="c"># in the url, so we fallback to the last page</span>
        <span class="n">topics</span> <span class="o">=</span> <span class="n">paginator</span><span class="o">.</span><span class="n">page</span><span class="p">(</span><span class="n">paginator</span><span class="o">.</span><span class="n">num_pages</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'topics.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'board'</span><span class="p">:</span> <span class="n">board</span><span class="p">,</span> <span class="s">'topics'</span><span class="p">:</span> <span class="n">topics</span><span class="p">})</span></code></pre></figure>

<p>Now the trick part is to render the pages correctly using the Bootstrap 4 pagination component. But take the time
to read the code and see if it makes sense for you. We are using here all the methods we played with before. And here
in that context, <code class="highlighter-rouge">topics</code> is no longer a QuerySet but a <code class="highlighter-rouge">paginator.Page</code> instance.</p>

<p>Right after the topics HTML table, we can render the pagination component:</p>

<p><strong>templates/topics.html</strong>
<small><a href="https://gist.github.com/vitorfs/3101a1bd72125aeb45829659a5532bc6" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">if</span> <span class="nv">topics.has_other_pages</span> <span class="cp">%}</span>
  <span class="nt">&lt;nav</span> <span class="na">aria-label=</span><span class="s">"Topics pagination"</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;ul</span> <span class="na">class=</span><span class="s">"pagination"</span><span class="nt">&gt;</span>
      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">topics.has_previous</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=</span><span class="cp">{{</span> <span class="nv">topics.previous_page_number</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span>Previous<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item disabled"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>Previous<span class="nt">&lt;/span&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

      <span class="cp">{%</span> <span class="k">for</span> <span class="nv">page_num</span> <span class="ow">in</span> <span class="nv">topics.paginator.page_range</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">if</span> <span class="nv">topics.</span><span class="nb">number</span> <span class="o">==</span> <span class="nv">page_num</span> <span class="cp">%}</span>
          <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item active"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>
              <span class="cp">{{</span> <span class="nv">page_num</span> <span class="cp">}}</span>
              <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"sr-only"</span><span class="nt">&gt;</span>(current)<span class="nt">&lt;/span&gt;</span>
            <span class="nt">&lt;/span&gt;</span>
          <span class="nt">&lt;/li&gt;</span>
        <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
          <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=</span><span class="cp">{{</span> <span class="nv">page_num</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">page_num</span> <span class="cp">}}</span><span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;/li&gt;</span>
        <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
      <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">topics.has_next</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"page-link"</span> <span class="na">href=</span><span class="s">"?page=</span><span class="cp">{{</span> <span class="nv">topics.next_page_number</span> <span class="cp">}}</span><span class="s">"</span><span class="nt">&gt;</span>Next<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"page-item disabled"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"page-link"</span><span class="nt">&gt;</span>Next<span class="nt">&lt;/span&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
    <span class="nt">&lt;/ul&gt;</span>
  <span class="nt">&lt;/nav&gt;</span>
<span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/paginate.png" alt="Pagination" /></p>

<h5 id="gcbv-pagination">GCBV Pagination</h5>

<p>Below, the same implementation but this time using the <strong>ListView</strong>.</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/323173bc56dec48fd83caf983d459421#file-views-py-L19" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">TopicListView</span><span class="p">(</span><span class="n">ListView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Topic</span>
    <span class="n">context_object_name</span> <span class="o">=</span> <span class="s">'topics'</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'topics.html'</span>
    <span class="n">paginate_by</span> <span class="o">=</span> <span class="mi">20</span>

    <span class="k">def</span> <span class="nf">get_context_data</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="n">kwargs</span><span class="p">[</span><span class="s">'board'</span><span class="p">]</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">board</span>
        <span class="k">return</span> <span class="nb">super</span><span class="p">()</span><span class="o">.</span><span class="n">get_context_data</span><span class="p">(</span><span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get_queryset</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">board</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">pk</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">kwargs</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'pk'</span><span class="p">))</span>
        <span class="n">queryset</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">board</span><span class="o">.</span><span class="n">topics</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'-last_updated'</span><span class="p">)</span><span class="o">.</span><span class="n">annotate</span><span class="p">(</span><span class="n">replies</span><span class="o">=</span><span class="n">Count</span><span class="p">(</span><span class="s">'posts'</span><span class="p">)</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">queryset</span></code></pre></figure>

<p>While using pagination with class-based views, the way we interact with the paginator in the template is a little bit
different. It will make available the following variables in the template: <strong>paginator</strong>, <strong>page_obj</strong>,
<strong>is_paginated</strong>, <strong>object_list</strong>, and also a variable with the name we defined in the <strong>context_object_name</strong>. In our
case this extra variable will be named <strong>topics</strong>, and it will be equivalent to <strong>object_list</strong>.</p>

<p>Now about the whole <strong>get_context_data</strong> thing, well, that’s how we add stuff to the request context when extending a
GCBV.</p>

<p>But the main point here is the <strong>paginate_by</strong> attribute. In some cases, just by adding it will be enough.</p>

<p>Remember to update the <strong>urls.py</strong>:</p>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/61f5345b7cf8b006b2901a61b8f8e348#file-urls-py-L37" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c"># ...</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">TopicListView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'board_topics'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Now let’s fix the template:</p>

<p><strong>templates/topics.html</strong>
<small><a href="https://gist.github.com/vitorfs/65095aa3eda78bafd22d5e2f94086d40#file-topics-html-L40" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'new_topic'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/div&gt;</span>

  <span class="nt">&lt;table</span> <span class="na">class=</span><span class="s">"table mb-4"</span><span class="nt">&gt;</span>
    <span class="c">&lt;!-- table content suppressed --&gt;</span>
  <span class="nt">&lt;/table&gt;</span>

  <span class="cp">{%</span> <span class="k">if</span> <span class="nv">is_paginated</span> <span class="cp">%}</span>
    <span class="nt">&lt;nav</span> <span class="na">aria-label=</span><span class="s">"Topics pagination"</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;ul</span> <span class="na">class=</span><span class="s">"pagination"</span><span class="nt">&gt;</span>
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
          <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
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
      <span class="nt">&lt;/ul&gt;</span>
    <span class="nt">&lt;/nav&gt;</span>
  <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Now take the time to run the tests and fix if needed.</p>

<p><strong>boards/tests/test_view_board_topics.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">TopicListView</span>

<span class="k">class</span> <span class="nc">BoardTopicsTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>
    <span class="k">def</span> <span class="nf">test_board_topics_url_resolves_board_topics_view</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/boards/1/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="o">.</span><span class="n">view_class</span><span class="p">,</span> <span class="n">TopicListView</span><span class="p">)</span></code></pre></figure>

<h5 id="reusable-pagination-template">Reusable Pagination Template</h5>

<p>Just like we did with the <strong>form.html</strong> partial template, we can also create something similar for the pagination
HTML snippet.</p>

<p>Let’s paginate the topic posts page, and then find a way to reuse the pagination component.</p>

<p><strong>boards/views.py</strong>
<small><a href="https://gist.github.com/vitorfs/53139ca0fd7c01b8459c2ff62828f963" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">PostListView</span><span class="p">(</span><span class="n">ListView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Post</span>
    <span class="n">context_object_name</span> <span class="o">=</span> <span class="s">'posts'</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'topic_posts.html'</span>
    <span class="n">paginate_by</span> <span class="o">=</span> <span class="mi">2</span>

    <span class="k">def</span> <span class="nf">get_context_data</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">views</span> <span class="o">+=</span> <span class="mi">1</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
        <span class="n">kwargs</span><span class="p">[</span><span class="s">'topic'</span><span class="p">]</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">topic</span>
        <span class="k">return</span> <span class="nb">super</span><span class="p">()</span><span class="o">.</span><span class="n">get_context_data</span><span class="p">(</span><span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get_queryset</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">topic</span> <span class="o">=</span> <span class="n">get_object_or_404</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">board__pk</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">kwargs</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'pk'</span><span class="p">),</span> <span class="n">pk</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">kwargs</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'topic_pk'</span><span class="p">))</span>
        <span class="n">queryset</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">topic</span><span class="o">.</span><span class="n">posts</span><span class="o">.</span><span class="n">order_by</span><span class="p">(</span><span class="s">'created_at'</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">queryset</span></code></pre></figure>

<p>Now update the <strong>urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/428d58dbb61f9c2601bb7434150ea37f" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.conf.urls</span> <span class="kn">import</span> <span class="n">url</span>
<span class="kn">from</span> <span class="nn">boards</span> <span class="kn">import</span> <span class="n">views</span>

<span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c"># ...</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^boards/(?P&lt;pk&gt;</span><span class="err">\</span><span class="s">d+)/topics/(?P&lt;topic_pk&gt;</span><span class="err">\</span><span class="s">d+)/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">PostListView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'topic_posts'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Now we grab that pagination HTML snippet from the <strong>topics.html</strong> template, and create a new file named
<strong>pagination.html</strong> inside the <strong>templates/includes</strong> folder, alongside with the <strong>forms.html</strong> file:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |-- myproject/
 |    |-- static/
 |    |-- templates/
 |    |    |-- includes/
 |    |    |    |-- form.html
 |    |    |    +-- pagination.html  &lt;-- here!
 |    |    +-- ...
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p><strong>templates/includes/pagination.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">if</span> <span class="nv">is_paginated</span> <span class="cp">%}</span>
  <span class="nt">&lt;nav</span> <span class="na">aria-label=</span><span class="s">"Topics pagination"</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;ul</span> <span class="na">class=</span><span class="s">"pagination"</span><span class="nt">&gt;</span>
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
        <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
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
    <span class="nt">&lt;/ul&gt;</span>
  <span class="nt">&lt;/nav&gt;</span>
<span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span></code></pre></figure>

<p>Now in the <strong>topic_posts.html</strong> template we use it:</p>

<p><strong>templates/topic_posts.html</strong>
<small><a href="https://gist.github.com/vitorfs/df5b16bb16c1134ba4e03218dce250d7" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>

  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'reply_topic'</span> <span class="nv">topic.board.pk</span> <span class="nv">topic.pk</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span> <span class="na">role=</span><span class="s">"button"</span><span class="nt">&gt;</span>Reply<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/div&gt;</span>

  <span class="cp">{%</span> <span class="k">for</span> <span class="nv">post</span> <span class="ow">in</span> <span class="nv">posts</span> <span class="cp">%}</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card </span><span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.last</span> <span class="cp">%}</span><span class="s">mb-4</span><span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span><span class="s">mb-2</span><span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span><span class="s"> </span><span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.first</span> <span class="cp">%}</span><span class="s">border-dark</span><span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">forloop.first</span> <span class="cp">%}</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-header text-white bg-dark py-2 px-3"</span><span class="nt">&gt;</span><span class="cp">{{</span> <span class="nv">topic.subject</span> <span class="cp">}}</span><span class="nt">&lt;/div&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
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
                <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'edit_post'</span> <span class="nv">post.topic.board.pk</span> <span class="nv">post.topic.pk</span> <span class="nv">post.pk</span> <span class="cp">%}</span><span class="s">"</span>
                   <span class="na">class=</span><span class="s">"btn btn-primary btn-sm"</span>
                   <span class="na">role=</span><span class="s">"button"</span><span class="nt">&gt;</span>Edit<span class="nt">&lt;/a&gt;</span>
              <span class="nt">&lt;/div&gt;</span>
            <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
          <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="cp">{%</span> <span class="k">endfor</span> <span class="cp">%}</span>

  <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/pagination.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Don’t forget to change the main forloop to <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">for</span><span class="w"> </span><span class="err">post</span><span class="w"> </span><span class="err">in</span><span class="w"> </span><span class="err">posts</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>.</p>

<p>We can also update the previous template, the <strong>topics.html</strong> template to use the pagination partial template too:</p>

<p><strong>templates/topics.html</strong>
<small><a href="https://gist.github.com/vitorfs/9198ad8f91cd889f315ade7e4eb62710#file-topics-html-L40" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"mb-4"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'new_topic'</span> <span class="nv">board.pk</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-primary"</span><span class="nt">&gt;</span>New topic<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;/div&gt;</span>

  <span class="nt">&lt;table</span> <span class="na">class=</span><span class="s">"table mb-4"</span><span class="nt">&gt;</span>
    <span class="c">&lt;!-- table code suppressed --&gt;</span>
  <span class="nt">&lt;/table&gt;</span>

  <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/pagination.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>Just for testing purpose, you could just add a few posts (or create some using the Python Shell) and change the
<strong>paginate_by</strong> to a low number, say 2, and see how it’s looking like:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-6/topics-1.png" alt="Topics" />
<small><a href="https://gist.github.com/vitorfs/6b3cd0769f805ab38626f5bd97b4e5e3" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<p>Update the test cases:</p>

<p><strong>boards/tests/test_view_topic_posts.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">..views</span> <span class="kn">import</span> <span class="n">PostListView</span>

<span class="k">class</span> <span class="nc">TopicPostsTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="c"># ...</span>
    <span class="k">def</span> <span class="nf">test_view_function</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/boards/1/topics/1/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="o">.</span><span class="n">view_class</span><span class="p">,</span> <span class="n">PostListView</span><span class="p">)</span></code></pre></figure>

<hr />
