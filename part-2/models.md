Models

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html

<p>The models are basically a representation of your application’s database layout. What we are going to do in this
section is create the Django representation of the classes we modeled in the previous section: <strong>Board</strong>, <strong>Topic</strong>,
and <strong>Post</strong>. The <strong>User</strong> model is already defined inside a built-in app named <strong>auth</strong>, which is listed in our
<code class="highlighter-rouge">INSTALLED_APPS</code> configuration under the namespace <strong>django.contrib.auth</strong>.</p>

<p>We will do all the work inside the <strong>boards/models.py</strong> file. Here is how we represent our class diagram (see
<a href="#figure-4">Figure 4</a>). in a Django application:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.db</span> <span class="kn">import</span> <span class="n">models</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>


<span class="k">class</span> <span class="nc">Board</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">name</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">30</span><span class="p">,</span> <span class="n">unique</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">description</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">100</span><span class="p">)</span>


<span class="k">class</span> <span class="nc">Topic</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">subject</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">255</span><span class="p">)</span>
    <span class="n">last_updated</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">DateTimeField</span><span class="p">(</span><span class="n">auto_now_add</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">board</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">Board</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'topics'</span><span class="p">)</span>
    <span class="n">starter</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">User</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'topics'</span><span class="p">)</span>


<span class="k">class</span> <span class="nc">Post</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">message</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">TextField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">4000</span><span class="p">)</span>
    <span class="n">topic</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">Topic</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'posts'</span><span class="p">)</span>
    <span class="n">created_at</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">DateTimeField</span><span class="p">(</span><span class="n">auto_now_add</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">updated_at</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">DateTimeField</span><span class="p">(</span><span class="n">null</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">created_by</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">User</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'posts'</span><span class="p">)</span>
    <span class="n">updated_by</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">ForeignKey</span><span class="p">(</span><span class="n">User</span><span class="p">,</span> <span class="n">null</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">related_name</span><span class="o">=</span><span class="s">'+'</span><span class="p">)</span></code></pre></figure>

<p>All models are subclass of the <strong>django.db.models.Model</strong> class. Each class will be transformed into <em>database tables</em>.
Each field is represented by instances of <strong>django.db.models.Field</strong> subclasses (built-in Django core) and will be
translated into <em>database columns</em>.</p>

<p>The fields <code class="highlighter-rouge">CharField</code>, <code class="highlighter-rouge">DateTimeField</code>, etc., are all subclasses of <strong>django.db.models.Field</strong> and they come included
in the Django core – ready to be used.</p>

<p>Here we are only using <code class="highlighter-rouge">CharField</code>, <code class="highlighter-rouge">TextField</code>, <code class="highlighter-rouge">DateTimeField</code>, and <code class="highlighter-rouge">ForeignKey</code> fields to define our models. But
Django offers a wide range of options to represent different types of data, such as <code class="highlighter-rouge">IntegerField</code>, <code class="highlighter-rouge">BooleanField</code>,
<code class="highlighter-rouge">DecimalField</code>, and many others. We will refer to them as we need.</p>

<p>Some fields have required arguments, such as the <code class="highlighter-rouge">CharField</code>. We should always set a <code class="highlighter-rouge">max_length</code>. This information
will be used to create the database column. Django needs to know how big the database column needs to be. The
<code class="highlighter-rouge">max_length</code> parameter will also be used by the Django Forms API, to validate user input. More on that later.</p>

<p>In the <code class="highlighter-rouge">Board</code> model definition, more specifically in the <code class="highlighter-rouge">name</code> field, we are also setting the parameter
<code class="highlighter-rouge">unique=True</code>, as the name suggests, it will enforce the uniqueness of the field at the database level.</p>

<p>In the <code class="highlighter-rouge">Post</code> model, the <code class="highlighter-rouge">created_at</code> field has an optional parameter, the <code class="highlighter-rouge">auto_now_add</code> set to <code class="highlighter-rouge">True</code>. This will
instruct Django to set the current date and time when a <code class="highlighter-rouge">Post</code> object is created.</p>

<p>One way to create a relationship between the models is by using the <code class="highlighter-rouge">ForeignKey</code> field. It will create a link between
the models and create a proper relationship at the database level. The <code class="highlighter-rouge">ForeignKey</code> field expects a positional
parameter with the reference to the model it will relate to.</p>

<p>For example, in the <code class="highlighter-rouge">Topic</code> model, the <code class="highlighter-rouge">board</code> field is a <code class="highlighter-rouge">ForeignKey</code> to the <code class="highlighter-rouge">Board</code> model. It is telling Django that
a <code class="highlighter-rouge">Topic</code> instance relates to only one <code class="highlighter-rouge">Board</code> instance. The <code class="highlighter-rouge">related_name</code> parameter will be used to create a
<em>reverse relationship</em> where the <code class="highlighter-rouge">Board</code> instances will have access a list of <code class="highlighter-rouge">Topic</code> instances that belong to it.</p>

<p>Django automatically creates this reverse relationship – the <code class="highlighter-rouge">related_name</code> is optional. But if we don’t set a name for
it, Django will generate it with the name: <code class="highlighter-rouge">(class_name)_set</code>. For example, in the <code class="highlighter-rouge">Board</code> model, the <code class="highlighter-rouge">Topic</code> instances
would be available under the <code class="highlighter-rouge">topic_set</code> property. Instead, we simply renamed it to <code class="highlighter-rouge">topics</code>, to make it feel more
natural.</p>

<p>In the <code class="highlighter-rouge">Post</code> model, the <code class="highlighter-rouge">updated_by</code> field sets the <code class="highlighter-rouge">related_name='+'</code>. This instructs Django that we don’t need this
reverse relationship, so it will ignore it.</p>

<p>Below you can see the comparison between the class diagram and the source code to generate the models with Django. The
green lines represent how we are handling the reverse relationships.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-django-models.png" alt="Class Diagram Models Definition" /></p>

<p>At this point, you may be asking yourself: “what about primary keys/IDs”? If we don’t specify a primary key for a model,
Django will automatically generate it for us. So we are good for now. In the next section, you will see better how it
works.</p>

<h5 id="migrating-the-models">Migrating the Models</h5>

<p>The next step is to tell Django to create the database so we can start using it.</p>

<p>Open the <span class="platform-mac platform-linux">Terminal</span> <span class="platform-windows">Command Line Tools</span>,
activate the virtual environment, go to the folder where the <strong>manage.py</strong> file is, and run the commands below:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py makemigrations</code></pre></figure>

<p>As an output you will get something like this:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">Migrations <span class="k">for</span> <span class="s1">'boards'</span>:
  boards/migrations/0001_initial.py
    - Create model Board
    - Create model Post
    - Create model Topic
    - Add field topic to post
    - Add field updated_by to post</code></pre></figure>

<p>At this point, Django created a file named <strong>0001_initial.py</strong> inside the <strong>boards/migrations</strong> directory. It represents
the current state of our application’s models. In the next step, Django will use this file to create the tables and
columns.</p>

<p>The migration files are translated into SQL statements. If you are familiar with SQL, you can run the following command
to inspect the SQL instructions that will be executed in the database:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py sqlmigrate boards 0001</code></pre></figure>

<p>If you’re not familiar with SQL, don’t worry. We won’t be working directly with SQL in this tutorial series. All the
work will be done using just the Django ORM, which is an abstraction layer that communicates with the database.</p>

<p>The next step now is to <em>apply</em> the migration we generated to the database:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py migrate</code></pre></figure>

<p>The output should be something like this:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">Operations to perform:
  Apply all migrations: admin, auth, boards, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying boards.0001_initial... OK
  Applying sessions.0001_initial... OK</code></pre></figure>

<p>Because this is the first time we are migrating the database, the <code class="highlighter-rouge">migrate</code> command also applied the existing migration
files from the Django contrib apps, listed in the <code class="highlighter-rouge">INSTALLED_APPS</code>. This is expected.</p>

<p>The line <code class="highlighter-rouge">Applying boards.0001_initial... OK</code> is the migration we generated in the previous step.</p>

<p>That’s it! Our database is ready to be used.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_SQLite.png" alt="SQLite" /></p>

<div class="info">
  <p>
    <strong><i class="fa fa-info-circle"></i> Note:</strong>
    It's important to note that <strong>SQLite</strong> is a production-quality database. SQLite is used by many companies across
    thousands of products, like all Android and iOS devices, all major Web browsers, Windows 10, macOS, etc.
  </p>
  <p>
    It's just not suitable for all cases. SQLite doesn't compare with databases like MySQL, PostgreSQL or Oracle.
    High-volume websites, write-intensive applications, very large datasets, high concurrency, are some situations that
    will eventually result in a problem by using SQLite.
  </p>
  <p>
    We are going to use SQLite during the development of our project because it's convenient and we won't need to
    install anything else. When we deploy our project to production, we will switch to PostgreSQL. For simple websites
    this work fine. But for complex websites, it's advisable to use the same database for development and production.
  </p>
</div>

<h5 id="experimenting-with-the-models-api">Experimenting with the Models API</h5>

<p>One of the great advantages of developing with Python is the interactive shell. I use it all the time. It’s a quick way
to try things out and experiment libraries and APIs.</p>

<p>You can start a Python shell with our project loaded using the <strong>manage.py</strong> utility:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py shell</code></pre></figure>

<div class="platform-mac">

  <figure class="highlight"><pre><code class="language-bash" data-lang="bash">Python 3.6.2 <span class="o">(</span>default, Jul 17 2017, 16:44:45<span class="o">)</span>
<span class="o">[</span>GCC 4.2.1 Compatible Apple LLVM 8.1.0 <span class="o">(</span>clang-802.0.42<span class="o">)]</span> on darwin
Type <span class="s2">"help"</span>, <span class="s2">"copyright"</span>, <span class="s2">"credits"</span> or <span class="s2">"license"</span> <span class="k">for </span>more information.
<span class="o">(</span>InteractiveConsole<span class="o">)</span>
&gt;&gt;&gt;</code></pre></figure>

</div>

<div class="platform-windows">

  <figure class="highlight"><pre><code class="language-bash" data-lang="bash">Python 3.6.2 <span class="o">(</span>v3.6.2:5fd33b5, Jul  8 2017, 04:57:36<span class="o">)</span> <span class="o">[</span>MSC v.1900 64 bit <span class="o">(</span>AMD64<span class="o">)]</span> on win32
Type <span class="s2">"help"</span>, <span class="s2">"copyright"</span>, <span class="s2">"credits"</span> or <span class="s2">"license"</span> <span class="k">for </span>more information.
<span class="o">(</span>InteractiveConsole<span class="o">)</span>
&gt;&gt;&gt;</code></pre></figure>

</div>

<div class="platform-linux">

  <figure class="highlight"><pre><code class="language-bash" data-lang="bash">Python 3.6.2 <span class="o">(</span>default, Jul 17 2017, 23:14:31<span class="o">)</span>
<span class="o">[</span>GCC 5.4.0 20160609] on linux
Type <span class="s2">"help"</span>, <span class="s2">"copyright"</span>, <span class="s2">"credits"</span> or <span class="s2">"license"</span> <span class="k">for </span>more information.
<span class="o">(</span>InteractiveConsole<span class="o">)</span>
&gt;&gt;&gt;</code></pre></figure>

</div>

<p>This is very similar to calling the interactive console just by typing <code class="highlighter-rouge">python</code>, except when we use
<code class="highlighter-rouge">python manage.py shell</code>, we are adding our project to the <code class="highlighter-rouge">sys.path</code> and loading Django. That means we can import
our models and any other resource within the project and play with it.</p>

<p>Let’s start by importing the <strong>Board</strong> class:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">boards.models</span> <span class="kn">import</span> <span class="n">Board</span></code></pre></figure>

<p>To create a new board object, we can do the following:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'This is a board about Django.'</span><span class="p">)</span></code></pre></figure>

<p>To persist this object in the database, we have to call the <code class="highlighter-rouge">save</code> method:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="n">save</span><span class="p">()</span></code></pre></figure>

<p>The <code class="highlighter-rouge">save</code> method is used both to <em>create</em> and <em>update</em> objects. Here Django created a new object because the <strong>Board</strong>
instance had no <strong>id</strong>. After saving it for the first time, Django will set the id automatically:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="nb">id</span>
<span class="mi">1</span></code></pre></figure>

<p>You can access the rest of the fields as Python attributes:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="n">name</span>
<span class="s">'Django'</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="n">description</span>
<span class="s">'This is a board about Django.'</span></code></pre></figure>

<p>To update a value we could do:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="n">description</span> <span class="o">=</span> <span class="s">'Django discussion board.'</span>
<span class="n">board</span><span class="o">.</span><span class="n">save</span><span class="p">()</span></code></pre></figure>

<p>Every Django model comes with a special attribute; we call it a <strong>Model Manager</strong>. You can access it via the Python
attribute <code class="highlighter-rouge">objects</code>. It is used mainly to execute queries in the database. For example, we could use it to directly
create a new <strong>Board</strong> object:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Python'</span><span class="p">,</span> <span class="n">description</span><span class="o">=</span><span class="s">'General discussion about Python.'</span><span class="p">)</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="nb">id</span>
<span class="mi">2</span></code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span><span class="o">.</span><span class="n">name</span>
<span class="s">'Python'</span></code></pre></figure>

<p>So, right now we have two boards. We can use the <code class="highlighter-rouge">objects</code> to list all existing boards in the database:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">all</span><span class="p">()</span>
<span class="o">&lt;</span><span class="n">QuerySet</span> <span class="p">[</span><span class="o">&lt;</span><span class="n">Board</span><span class="p">:</span> <span class="n">Board</span> <span class="nb">object</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Board</span><span class="p">:</span> <span class="n">Board</span> <span class="nb">object</span><span class="o">&gt;</span><span class="p">]</span><span class="o">&gt;</span></code></pre></figure>

<p>The result was a <strong>QuerySet</strong>. We will learn more about that later on. Basically, it’s a list of objects from the
database. We can see that we have two objects, but we can only read <strong>Board object</strong>. That’s because we haven’t defined
the <code class="highlighter-rouge">__str__</code> method in the <strong>Board</strong> model.</p>

<p>The <code class="highlighter-rouge">__str__</code> method is a String representation of an object. We can use the board name to represent it.</p>

<p>First, exit the interactive console:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="nb">exit</span><span class="p">()</span></code></pre></figure>

<p>Now edit the <strong>models.py</strong> file inside the <strong>boards</strong> app:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">Board</span><span class="p">(</span><span class="n">models</span><span class="o">.</span><span class="n">Model</span><span class="p">):</span>
    <span class="n">name</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">30</span><span class="p">,</span> <span class="n">unique</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="n">description</span> <span class="o">=</span> <span class="n">models</span><span class="o">.</span><span class="n">CharField</span><span class="p">(</span><span class="n">max_length</span><span class="o">=</span><span class="mi">100</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">__str__</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="k">return</span> <span class="bp">self</span><span class="o">.</span><span class="n">name</span></code></pre></figure>

<p>Let’s try the query again. Open the interactive console again:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py shell</code></pre></figure>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">boards.models</span> <span class="kn">import</span> <span class="n">Board</span>

<span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">all</span><span class="p">()</span>
<span class="o">&lt;</span><span class="n">QuerySet</span> <span class="p">[</span><span class="o">&lt;</span><span class="n">Board</span><span class="p">:</span> <span class="n">Django</span><span class="o">&gt;</span><span class="p">,</span> <span class="o">&lt;</span><span class="n">Board</span><span class="p">:</span> <span class="n">Python</span><span class="o">&gt;</span><span class="p">]</span><span class="o">&gt;</span></code></pre></figure>

<p>Much better, right?</p>

<p>We can treat this <strong>QuerySet</strong> like a list. Let’s say we wanted to iterate over it and print the description of each
board:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">boards_list</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="nb">all</span><span class="p">()</span>
<span class="k">for</span> <span class="n">board</span> <span class="ow">in</span> <span class="n">boards_list</span><span class="p">:</span>
    <span class="k">print</span><span class="p">(</span><span class="n">board</span><span class="o">.</span><span class="n">description</span><span class="p">)</span></code></pre></figure>

<p>The result would be:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Django discussion board.
General discussion about Python.</code></pre></figure>

<p>Similarly, we can use the model <strong>Manager</strong> to query the database and return a single object. For that we use the <code class="highlighter-rouge">get</code>
method:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">django_board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>

<span class="n">django_board</span><span class="o">.</span><span class="n">name</span>
<span class="s">'Django'</span></code></pre></figure>

<p>But we have to be careful with this kind of operation. If we try to get an object that doesn’t exist, for example, a
board with <code class="highlighter-rouge">id=3</code>, it will raise an exception:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">board</span> <span class="o">=</span> <span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="mi">3</span><span class="p">)</span>

<span class="n">boards</span><span class="o">.</span><span class="n">models</span><span class="o">.</span><span class="n">DoesNotExist</span><span class="p">:</span> <span class="n">Board</span> <span class="n">matching</span> <span class="n">query</span> <span class="n">does</span> <span class="ow">not</span> <span class="n">exist</span><span class="o">.</span></code></pre></figure>

<p>We can use the <code class="highlighter-rouge">get</code> method with any model field, but preferably use a field that can uniquely identify an object.
Otherwise, the query may return more than one object, which will cause an exception.</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'Django'</span><span class="p">)</span>
<span class="o">&lt;</span><span class="n">Board</span><span class="p">:</span> <span class="n">Django</span><span class="o">&gt;</span></code></pre></figure>

<p>Note that the query is <em>case sensitive</em>, a lower case “django” would not match:</p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">Board</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s">'django'</span><span class="p">)</span>
<span class="n">boards</span><span class="o">.</span><span class="n">models</span><span class="o">.</span><span class="n">DoesNotExist</span><span class="p">:</span> <span class="n">Board</span> <span class="n">matching</span> <span class="n">query</span> <span class="n">does</span> <span class="ow">not</span> <span class="n">exist</span><span class="o">.</span></code></pre></figure>

<h5 id="summary-of-models-operations">Summary of Model’s Operations</h5>

<p>Find below a summary of the methods and operations we learned in this section, using the <strong>Board</strong> model as a reference.
Uppercase <strong>Board</strong> refers to the class, lowercase <strong>board</strong> refers to an instance (or object) of the <strong>Board</strong> model
class:</p>

<table>
  <thead>
    <tr>
      <th>Operation</th>
      <th>Code sample</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Create an object without saving</td>
      <td><code class="highlighter-rouge">board = Board()</code></td>
    </tr>
    <tr>
      <td>Save an object (create or update)</td>
      <td><code class="highlighter-rouge">board.save()</code></td>
    </tr>
    <tr>
      <td>Create and save an object in the database</td>
      <td><code class="highlighter-rouge">Board.objects.create(name='...', description='...')</code></td>
    </tr>
    <tr>
      <td>List all objects</td>
      <td><code class="highlighter-rouge">Board.objects.all()</code></td>
    </tr>
    <tr>
      <td>Get a single object, identified by a field</td>
      <td><code class="highlighter-rouge">Board.objects.get(id=1)</code></td>
    </tr>
  </tbody>
</table>

<p>In the next section, we are going to start writing views and displaying our boards in HTML pages.</p>

<hr />
