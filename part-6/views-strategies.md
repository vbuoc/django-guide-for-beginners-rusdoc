<h4 id="views-strategies">Views Strategies</h4>

<p>At the end of the day, all Django views are <em>functions</em>. Even class-based views (CBV). Behind the scenes, it does all
the magic and ends up returning a view function.</p>

<p>Class-based views were introduced to make it easier for developers to reuse and extend views. There are many benefits
of using them, such as the extendability, the ability to use O.O. techniques such as multiple inheritances, the handling
of HTTP methods are done in separate methods, rather than using conditional branching, and there are also the
Generic Class-Based Views (GCBV).</p>

<p>Before we move forward, let’s clarify what those three terms mean:</p>

<ul>
  <li>Function-Based Views (FBV)</li>
  <li>Class-Based Views (CBV)</li>
  <li>Generic Class-Based Views (GCBV)</li>
</ul>

<p>A FBV is the simplest representation of a Django view: it’s just a function that receives an <strong>HttpRequest</strong> object and
returns an <strong>HttpResponse</strong>.</p>

<p>A CBV is every Django view defined as a Python class that extends the <code class="highlighter-rouge">django.views.generic.View</code> abstract class. A
CBV essentially is a class that wraps a FBV. CBVs are great to extend and reuse code.</p>

<p>GCBVs are built-in CBVs that solve specific problems such as listing views, create, update, and delete views.</p>

<p>Below we are going to explore some examples of the different implementation strategies.</p>

<h5 id="function-based-view">Function-Based View</h5>

<p><strong>views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">def</span> <span class="nf">new_post</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">request</span><span class="o">.</span><span class="n">method</span> <span class="o">==</span> <span class="s">'POST'</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'post_list'</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_post.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p><strong>urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^new_post/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">new_post</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="s">'new_post'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<h5 id="class-based-view">Class-Based View</h5>

<p>A CBV is a view that extends the <strong>View</strong> class. The main difference here is that the requests are handled inside class
methods named after the HTTP methods, such as <strong>get</strong>, <strong>post</strong>, <strong>put</strong>, <strong>head</strong>, etc.</p>

<p>So, here we don’t need to do a conditional to check if the request is a <strong>POST</strong> or if it’s a <strong>GET</strong>. The code goes
straight to the right method. This logic is handled internally in the <strong>View</strong> class.</p>

<p><strong>views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">View</span>

<span class="k">class</span> <span class="nc">NewPostView</span><span class="p">(</span><span class="n">View</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">post</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">request</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'post_list'</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_post.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span>

    <span class="k">def</span> <span class="nf">get</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">request</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_post.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="n">form</span><span class="p">})</span></code></pre></figure>

<p>The way we refer to the CBVs in the <strong>urls.py</strong> module is a little bit different:</p>

<p><strong>urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^new_post/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">NewPostView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'new_post'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Here we need to use the <code class="highlighter-rouge">as_view()</code> class method, which returns a view function to the url patterns. In some cases, we
can also feed the <code class="highlighter-rouge">as_view()</code> with some keyword arguments, so to customize the behavior of the CBV, just like we did
with some of the authentication views to customize the templates.</p>

<p>Anyway, the good thing about CBV is that we can add more methods, and perhaps do something like this:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">View</span>

<span class="k">class</span> <span class="nc">NewPostView</span><span class="p">(</span><span class="n">View</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">render</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">request</span><span class="p">):</span>
        <span class="k">return</span> <span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="s">'new_post.html'</span><span class="p">,</span> <span class="p">{</span><span class="s">'form'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">form</span><span class="p">})</span>

    <span class="k">def</span> <span class="nf">post</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">request</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">POST</span><span class="p">)</span>
        <span class="k">if</span> <span class="bp">self</span><span class="o">.</span><span class="n">form</span><span class="o">.</span><span class="n">is_valid</span><span class="p">():</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">form</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>
            <span class="k">return</span> <span class="n">redirect</span><span class="p">(</span><span class="s">'post_list'</span><span class="p">)</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">get</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">request</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">form</span> <span class="o">=</span> <span class="n">PostForm</span><span class="p">()</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">render</span><span class="p">(</span><span class="n">request</span><span class="p">)</span></code></pre></figure>

<p>It’s also possible to create some generic views that accomplish some tasks so that we can reuse it across the project.</p>

<p>But that’s pretty much all you need to know about CBVs. Simple as that.</p>

<h5 id="generic-class-based-view">Generic Class-Based View</h5>

<p>Now about the GCBV. That’s a different story. As I mentioned earlier, those views are built-in CBVs for common use
cases. Their implementation makes heavy usage of multiple inheritances (mixins) and other O.O. strategies.</p>

<p>They are very flexible and can save many hours of work. But in the beginning, it can be difficult to work with them.</p>

<p>When I first started working with Django, I found GCBV hard to work with. At first, it’s hard to tell what is going on,
because the code flow is not obvious, as there is good chunk of code hidden in the parent classes. The documentation
is a little bit challenging to follow too, mostly because the attributes and methods are sometimes spread across eight
parent classes. When working with GCBV, it’s always good to have the <a href="https://ccbv.co.uk/" target="_blank" rel="noopener">ccbv.co.uk</a>
opened for quick reference. No worries, we are going to explore it together.</p>

<p>Now let’s see a GCBV example.</p>

<p><strong>views.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.views.generic</span> <span class="kn">import</span> <span class="n">CreateView</span>

<span class="k">class</span> <span class="nc">NewPostView</span><span class="p">(</span><span class="n">CreateView</span><span class="p">):</span>
    <span class="n">model</span> <span class="o">=</span> <span class="n">Post</span>
    <span class="n">form_class</span> <span class="o">=</span> <span class="n">PostForm</span>
    <span class="n">success_url</span> <span class="o">=</span> <span class="n">reverse_lazy</span><span class="p">(</span><span class="s">'post_list'</span><span class="p">)</span>
    <span class="n">template_name</span> <span class="o">=</span> <span class="s">'new_post.html'</span></code></pre></figure>

<p>Here we are using a generic view used to create model objects. It does all the form processing and save the object
if the form is valid.</p>

<p>Since it’s a CBV, we refer to it in the <strong>urls.py</strong> the same way as any other CBV:</p>

<p><strong>urls.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">urlpatterns</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">url</span><span class="p">(</span><span class="s">r'^new_post/$'</span><span class="p">,</span> <span class="n">views</span><span class="o">.</span><span class="n">NewPostView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(),</span> <span class="n">name</span><span class="o">=</span><span class="s">'new_post'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>Other examples of GCBVs are <strong>DetailView</strong>, <strong>DeleteView</strong>, <strong>FormView</strong>, <strong>UpdateView</strong>, <strong>ListView</strong>.</p>

<hr />
