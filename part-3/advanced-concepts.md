# Продвинутые концепции

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/18/a-complete-beginners-guide-to-django-part-3.html

<h4 id="introduction">Introduction</h4>

<p>In this tutorial, we are going to dive deep into two fundamental concepts: URLs and Forms. In the process, we are going
to explore many other concepts like creating reusable templates and installing third-party libraries. We are also
going to write plenty of unit tests.</p>

<p>If you are following this tutorial series since the first part, coding your project and following the tutorial step by
step, you may need to update your <strong>models.py</strong> before starting:</p>

<p><strong>boards/models.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">Topic</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># other fields...</span>
    <span class="c"># Add `auto_now_add=True` to the `last_updated` field</span>
    <span class="n">last_updated</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">DateTimeField</span><span class="p">(</span><span class="n">auto_now_add</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">Post</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="c"># other fields...</span>
    <span class="c"># Add `null=True` to the `updated_by` field</span>
    <span class="n">updated_by</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">User</span><span class="p">,</span> <span class="n">null</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'+'</span><span class="p">)</span></code></pre></figure>

<p>Now run the commands with the virtualenv activated:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py makemigrations
python manage.py migrate</code></pre></figure>

<p>If you already have <code class="highlighter-rouge">null=True</code> in the <code class="highlighter-rouge">updated_by</code> field and the <code class="highlighter-rouge">auto_now_add=True</code> in the <code class="highlighter-rouge">last_updated</code> field,
you can safely ignore the instructions above.</p>

<p>If you prefer to use my source code as a starting point, you can grab it on GitHub.</p>

<p>The current state of the project can be found under the release tag <strong>v0.2-lw</strong>. The link below will take you to the
right place:</p>

<p><a href="https://github.com/sibtc/django-beginners-guide/tree/v0.2-lw" target="_blank">https://github.com/sibtc/django-beginners-guide/tree/v0.2-lw</a></p>

<p>The development will follow from here.</p>

<hr />



h4 id="conclusions">Conclusions</h4>

<p>In this tutorial, we focused on URLs, Reusable Templates, and Forms. As usual, we also implement several test cases.
That’s how we develop with confidence.</p>

<p>Our tests file is starting to get big, so in the next tutorial, we are going to refactor it to improve the
maintainability so to sustain the growth of our code base.</p>

<p>We are also reaching a point where we need to interact with the logged in user. In the next tutorial, we are going to
learn everything about authentication and how to protect our views and resources.</p>

<p>I hope you enjoyed the third part of this tutorial series! The fourth part is coming out next week, on Sep 25, 2017.
If you would like to get notified when the fourth part is out, you can <a href="http://eepurl.com/b0gR51" target="_blank">subscribe to our mailing list</a>.</p>

<p>The source code of the project is available on GitHub. The current state of the project can be found under the release
tag <strong>v0.3-lw</strong>. The link below will take you to the right place:</p>

<p><a href="https://github.com/sibtc/django-beginners-guide/tree/v0.3-lw" target="_blank">https://github.com/sibtc/django-beginners-guide/tree/v0.3-lw</a></p>
