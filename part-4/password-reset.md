<h4 id="password-reset">Password Reset</h4>

<p>The password reset process involves some nasty URL patterns. But as we discussed in the previous tutorial, we don’t
need to be an expert in regular expressions. It’s just a matter of knowing the common ones.</p>

<p>Another important thing before we start is that, for the password reset process, we need to send emails. It’s a little
bit complicated in the beginning because we need an external service. For now, we won’t be configuring a production
quality email service. In fact, during the development phase, we can use Django’s debug tools to check if the emails
are being sent correctly.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Email.png" alt="Comic Email" /></p>

<h5 id="console-email-backend">Console Email Backend</h5>

<p>The idea is during the development of the project, instead of sending real emails, we just log them. There are
two options: writing all emails in a text file or simply displaying them in the console. I find the latter option more
convenient because we are already using a console to run the development server and the setup is a bit easier.</p>

<p>Edit the <strong>settings.py</strong> module and add the <code class="highlighter-rouge">EMAIL_BACKEND</code> variable to the end of the file:</p>

<p><strong>myproject/settings.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">EMAIL_BACKEND</span> <span class="o">=</span> <span class="s">'django.core.mail.backends.console.EmailBackend'</span></code></pre></figure>

<h5 id="configuring-the-routes">Configuring the Routes</h5>

<p>The password reset process requires four views:</p>

<ul>
  <li>A page with a form to start the reset process;</li>
  <li>A success page saying the process initiated, instructing the user to check their spam folders, etc.;</li>
  <li>A page to check the token sent via email;</li>
  <li>A page to tell the user if the reset was successful or not.</li>
</ul>

<p>The views are built-in, we don’t need to implement anything. All we need to do is add the routes to the <strong>urls.py</strong>
and create the templates.</p>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/117e300e00d5685f7186e09260f82736#file-urls-py-L14" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">url</span><span class="p">(</span><span class="s">r'^reset/$'</span><span class="p">,</span>
    <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(</span>
        <span class="n">template_name</span><span class="o">=</span><span class="s">'password_reset.html'</span><span class="p">,</span>
        <span class="n">email_template_name</span><span class="o">=</span><span class="s">'password_reset_email.html'</span><span class="p">,</span>
        <span class="n">subject_template_name</span><span class="o">=</span><span class="s">'password_reset_subject.txt'</span>
    <span class="p">),</span>
    <span class="n">name</span><span class="o">=</span><span class="s">'password_reset'</span><span class="p">),</span>
<span class="n">url</span><span class="p">(</span><span class="s">r'^reset/done/$'</span><span class="p">,</span>
    <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetDoneView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(</span><span class="n">template_name</span><span class="o">=</span><span class="s">'password_reset_done.html'</span><span class="p">),</span>
    <span class="n">name</span><span class="o">=</span><span class="s">'password_reset_done'</span><span class="p">),</span>
<span class="n">url</span><span class="p">(</span><span class="s">r'^reset/(?P&lt;uidb64&gt;[0-9A-Za-z_</span><span class="err">\</span><span class="s">-]+)/(?P&lt;token&gt;[0-9A-Za-z]{1,13}-[0-9A-Za-z]{1,20})/$'</span><span class="p">,</span>
    <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetConfirmView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(</span><span class="n">template_name</span><span class="o">=</span><span class="s">'password_reset_confirm.html'</span><span class="p">),</span>
    <span class="n">name</span><span class="o">=</span><span class="s">'password_reset_confirm'</span><span class="p">),</span>
<span class="n">url</span><span class="p">(</span><span class="s">r'^reset/complete/$'</span><span class="p">,</span>
    <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetCompleteView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(</span><span class="n">template_name</span><span class="o">=</span><span class="s">'password_reset_complete.html'</span><span class="p">),</span>
    <span class="n">name</span><span class="o">=</span><span class="s">'password_reset_complete'</span><span class="p">),</span>
<span class="p">]</span></code></pre></figure>

<p>The <code class="highlighter-rouge">template_name</code> parameter in the password reset views are optional. But I thought it would be a good idea to
re-define it, so the link between the view and the template be more obvious than just using the defaults.</p>

<p>Inside the <strong>templates</strong> folder, the following template files:</p>

<ul>
  <li><strong>password_reset.html</strong></li>
  <li><strong>password_reset_email.html</strong>: this template is the body of the email message sent to the user</li>
  <li><strong>password_reset_subject.txt</strong>: this template is the subject line of the email, it should be a single line file</li>
  <li><strong>password_reset_done.html</strong></li>
  <li><strong>password_reset_confirm.html</strong></li>
  <li><strong>password_reset_complete.html</strong></li>
</ul>

<p>Before we start implementing the templates, let’s prepare a new test file.</p>

<p>We can add just some basic tests because those views and forms are already tested in the Django code. We are going to
test just the specifics of our application.</p>

<p>Create a new test file named <strong>test_view_password_reset.py</strong> inside the <strong>accounts/tests</strong> folder.</p>

<h5 id="password-reset-view">Password Reset View</h5>

<p><strong>templates/password_reset.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base_accounts.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Reset your password<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row justify-content-center"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-4 col-md-6 col-sm-8"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Reset your password<span class="nt">&lt;/h3&gt;</span>
          <span class="nt">&lt;p&gt;</span>Enter your email address and we will send you a link to reset your password.<span class="nt">&lt;/p&gt;</span>
          <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
            <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
            <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
            <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-primary btn-block"</span><span class="nt">&gt;</span>Send password reset email<span class="nt">&lt;/button&gt;</span>
          <span class="nt">&lt;/form&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset.jpg" alt="Password Reset" /></p>

<p><strong>accounts/tests/test_view_password_reset.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">auth_views</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">PasswordResetForm</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.core</span> <span class="kn">import</span> <span class="n">mail</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>


<span class="k">class</span> <span class="nc">PasswordResetTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_view_function</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/reset/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="o">.</span><span class="n">view_class</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetView</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_csrf</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'csrfmiddlewaretoken'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_contains_form</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIsInstance</span><span class="p">(</span><span class="n">form</span><span class="p">,</span> <span class="n">PasswordResetForm</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_form_inputs</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        The view must contain two inputs: csrf and email
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'&lt;input'</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'type="email"'</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>


<span class="k">class</span> <span class="nc">SuccessfulPasswordResetTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">email</span> <span class="o">=</span> <span class="s">'john@doe.com'</span>
        <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="n">email</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'123abcdef'</span><span class="p">)</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="p">{</span><span class="s">'email'</span><span class="p">:</span> <span class="n">email</span><span class="p">})</span>

    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        A valid form submission should redirect the user to `password_reset_done` view
        '''</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset_done'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_send_password_reset_email</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEqual</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">mail</span><span class="o">.</span><span class="n">outbox</span><span class="p">))</span>


<span class="k">class</span> <span class="nc">InvalidPasswordResetTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="p">{</span><span class="s">'email'</span><span class="p">:</span> <span class="s">'donotexist@email.com'</span><span class="p">})</span>

    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        Even invalid emails in the database should
        redirect the user to `password_reset_done` view
        '''</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset_done'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_no_reset_email_sent</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEqual</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nb">len</span><span class="p">(</span><span class="n">mail</span><span class="o">.</span><span class="n">outbox</span><span class="p">))</span></code></pre></figure>

<p><strong>templates/password_reset_subject.txt</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django">[Django Boards] Please reset your password</code></pre></figure>

<p><strong>templates/password_reset_email.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django">Hi there,

Someone asked for a password reset for the email address <span class="cp">{{</span> <span class="nv">email</span> <span class="cp">}}</span>.
Follow the link below:
<span class="cp">{{</span> <span class="nv">protocol</span> <span class="cp">}}</span>://<span class="cp">{{</span> <span class="nv">domain</span> <span class="cp">}}{%</span> <span class="nv">url</span> <span class="s1">'password_reset_confirm'</span> <span class="nv">uidb64</span><span class="err">=</span><span class="nv">uid</span> <span class="nv">token</span><span class="err">=</span><span class="nv">token</span> <span class="cp">%}</span>

In case you forgot your Django Boards username: <span class="cp">{{</span> <span class="nv">user.username</span> <span class="cp">}}</span>

If clicking the link above doesn't work, please copy and paste the URL
in a new browser window instead.

If you've received this mail in error, it's likely that another user entered
your email address by mistake while trying to reset a password. If you didn't
initiate the request, you don't need to take any further action and can safely
disregard this email.

Thanks,

The Django Boards Team</code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_email.png" alt="Password Reset Email" /></p>

<p>We can create a specific file to test the email message. Create a new file named <strong>test_mail_password_reset.py</strong>
inside the <strong>accounts/tests</strong> folder:</p>

<p><strong>accounts/tests/test_mail_password_reset.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.core</span> <span class="kn">import</span> <span class="n">mail</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>

<span class="k">class</span> <span class="nc">PasswordResetMailTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'123'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset'</span><span class="p">),</span> <span class="p">{</span> <span class="s">'email'</span><span class="p">:</span> <span class="s">'john@doe.com'</span> <span class="p">})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">email</span> <span class="o">=</span> <span class="n">mail</span><span class="o">.</span><span class="n">outbox</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>

    <span class="k">def</span> <span class="nf">test_email_subject</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEqual</span><span class="p">(</span><span class="s">'[Django Boards] Please reset your password'</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">email</span><span class="o">.</span><span class="n">subject</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_email_body</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">context</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">context</span>
        <span class="n">token</span> <span class="o">=</span> <span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'token'</span><span class="p">)</span>
        <span class="n">uid</span> <span class="o">=</span> <span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'uid'</span><span class="p">)</span>
        <span class="n">password_reset_token_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset_confirm'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span>
            <span class="s">'uidb64'</span><span class="p">:</span> <span class="n">uid</span><span class="p">,</span>
            <span class="s">'token'</span><span class="p">:</span> <span class="n">token</span>
        <span class="p">})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIn</span><span class="p">(</span><span class="n">password_reset_token_url</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">email</span><span class="o">.</span><span class="n">body</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIn</span><span class="p">(</span><span class="s">'john'</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">email</span><span class="o">.</span><span class="n">body</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIn</span><span class="p">(</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">email</span><span class="o">.</span><span class="n">body</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_email_to</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEqual</span><span class="p">([</span><span class="s">'john@doe.com'</span><span class="p">,],</span> <span class="bp">self</span><span class="o">.</span><span class="n">email</span><span class="o">.</span><span class="n">to</span><span class="p">)</span></code></pre></figure>

<p>This test case grabs the email sent by the application, and examine the subject line, the body contents, and to who
was the email sent to.</p>

<h5 id="password-reset-done-view">Password Reset Done View</h5>

<p><strong>templates/password_reset_done.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base_accounts.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Reset your password<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row justify-content-center"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-4 col-md-6 col-sm-8"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Reset your password<span class="nt">&lt;/h3&gt;</span>
          <span class="nt">&lt;p&gt;</span>Check your email for a link to reset your password. If it doesn't appear within a few minutes, check your spam folder.<span class="nt">&lt;/p&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'login'</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-secondary btn-block"</span><span class="nt">&gt;</span>Return to log in<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_done.jpg" alt="Password Reset Done" /></p>

<p><strong>accounts/tests/test_view_password_reset.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">auth_views</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>

<span class="k">class</span> <span class="nc">PasswordResetDoneTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset_done'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_view_function</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/reset/done/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="o">.</span><span class="n">view_class</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetDoneView</span><span class="p">)</span></code></pre></figure>

<h5 id="password-reset-confirm-view">Password Reset Confirm View</h5>

<p><strong>templates/password_reset_confirm.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base_accounts.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>
  <span class="cp">{%</span> <span class="k">if</span> <span class="nv">validlink</span> <span class="cp">%}</span>
    Change password for <span class="cp">{{</span> <span class="nv">form.user.username</span> <span class="cp">}}</span>
  <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
    Reset your password
  <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row justify-content-center"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-6 col-md-8 col-sm-10"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body"</span><span class="nt">&gt;</span>
          <span class="cp">{%</span> <span class="k">if</span> <span class="nv">validlink</span> <span class="cp">%}</span>
            <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Change password for @<span class="cp">{{</span> <span class="nv">form.user.username</span> <span class="cp">}}</span><span class="nt">&lt;/h3&gt;</span>
            <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
              <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
              <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
              <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success btn-block"</span><span class="nt">&gt;</span>Change password<span class="nt">&lt;/button&gt;</span>
            <span class="nt">&lt;/form&gt;</span>
          <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
            <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Reset your password<span class="nt">&lt;/h3&gt;</span>
            <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"alert alert-danger"</span> <span class="na">role=</span><span class="s">"alert"</span><span class="nt">&gt;</span>
              It looks like you clicked on an invalid password reset link. Please try again.
            <span class="nt">&lt;/div&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'password_reset'</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-secondary btn-block"</span><span class="nt">&gt;</span>Request a new password reset link<span class="nt">&lt;/a&gt;</span>
          <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p>This page can only be accessed with the link sent in the email. It looks like this: <strong>http://127.0.0.1:8000/reset/Mw/4po-2b5f2d47c19966e294a1/</strong></p>

<p>During the development phase, grab this link from the email in the console.</p>

<p>If the link is valid:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_confirm_valid.jpg" alt="Password Reset Confirm Valid" /></p>

<p>Or if the link has already been used:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_confirm_invalid.jpg" alt="Password Reset Confirm Invalid" /></p>

<p><strong>accounts/tests/test_view_password_reset.py</strong></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth.tokens</span> <span class="kn">import</span> <span class="n">default_token_generator</span>
<span class="kn">from</span> <span class="nn">django.utils.encoding</span> <span class="kn">import</span> <span class="n">force_bytes</span>
<span class="kn">from</span> <span class="nn">django.utils.http</span> <span class="kn">import</span> <span class="n">urlsafe_base64_encode</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">auth_views</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.forms</span> <span class="kn">import</span> <span class="n">SetPasswordForm</span>
<span class="kn">from</span> <span class="nn">django.contrib.auth.models</span> <span class="kn">import</span> <span class="n">User</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>


<span class="k">class</span> <span class="nc">PasswordResetConfirmTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'123abcdef'</span><span class="p">)</span>

        <span class="s">'''
        create a valid password reset token
        based on how django creates the token internally:
        https://github.com/django/django/blob/1.11.5/django/contrib/auth/forms.py#L280
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">uid</span> <span class="o">=</span> <span class="n">urlsafe_base64_encode</span><span class="p">(</span><span class="n">force_bytes</span><span class="p">(</span><span class="n">user</span><span class="o">.</span><span class="n">pk</span><span class="p">))</span><span class="o">.</span><span class="n">decode</span><span class="p">()</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">token</span> <span class="o">=</span> <span class="n">default_token_generator</span><span class="o">.</span><span class="n">make_token</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>

        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset_confirm'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'uidb64'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">uid</span><span class="p">,</span> <span class="s">'token'</span><span class="p">:</span> <span class="bp">self</span><span class="o">.</span><span class="n">token</span><span class="p">})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">follow</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_view_function</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/reset/{uidb64}/{token}/'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">uidb64</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">uid</span><span class="p">,</span> <span class="n">token</span><span class="o">=</span><span class="bp">self</span><span class="o">.</span><span class="n">token</span><span class="p">))</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="o">.</span><span class="n">view_class</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetConfirmView</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_csrf</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'csrfmiddlewaretoken'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_contains_form</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertIsInstance</span><span class="p">(</span><span class="n">form</span><span class="p">,</span> <span class="n">SetPasswordForm</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_form_inputs</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        The view must contain two inputs: csrf and two password fields
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'&lt;input'</span><span class="p">,</span> <span class="mi">3</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'type="password"'</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>


<span class="k">class</span> <span class="nc">InvalidPasswordResetConfirmTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'123abcdef'</span><span class="p">)</span>
        <span class="n">uid</span> <span class="o">=</span> <span class="n">urlsafe_base64_encode</span><span class="p">(</span><span class="n">force_bytes</span><span class="p">(</span><span class="n">user</span><span class="o">.</span><span class="n">pk</span><span class="p">))</span><span class="o">.</span><span class="n">decode</span><span class="p">()</span>
        <span class="n">token</span> <span class="o">=</span> <span class="n">default_token_generator</span><span class="o">.</span><span class="n">make_token</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>

        <span class="s">'''
        invalidate the token by changing the password
        '''</span>
        <span class="n">user</span><span class="o">.</span><span class="n">set_password</span><span class="p">(</span><span class="s">'abcdef123'</span><span class="p">)</span>
        <span class="n">user</span><span class="o">.</span><span class="n">save</span><span class="p">()</span>

        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset_confirm'</span><span class="p">,</span> <span class="n">kwargs</span><span class="o">=</span><span class="p">{</span><span class="s">'uidb64'</span><span class="p">:</span> <span class="n">uid</span><span class="p">,</span> <span class="s">'token'</span><span class="p">:</span> <span class="n">token</span><span class="p">})</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_html</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">password_reset_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'invalid password reset link'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertContains</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="s">'href="{0}"'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">password_reset_url</span><span class="p">))</span></code></pre></figure>

<h5 id="password-reset-complete-view">Password Reset Complete View</h5>

<p><strong>templates/password_reset_complete.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base_accounts.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Password changed!<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row justify-content-center"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-6 col-md-8 col-sm-10"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"card-body"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;h3</span> <span class="na">class=</span><span class="s">"card-title"</span><span class="nt">&gt;</span>Password changed!<span class="nt">&lt;/h3&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"alert alert-success"</span> <span class="na">role=</span><span class="s">"alert"</span><span class="nt">&gt;</span>
            You have successfully changed your password! You may now proceed to log in.
          <span class="nt">&lt;/div&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'login'</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-secondary btn-block"</span><span class="nt">&gt;</span>Return to log in<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_complete.jpg" alt="Password Reset Complete" /></p>

<p><strong>accounts/tests/test_view_password_reset.py</strong>
<small><a href="https://gist.github.com/vitorfs/c9657d39d28c2a0cfb0933e715bfc9cf#file-test_view_password_reset-py-L149" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="kn">from</span> <span class="nn">django.contrib.auth</span> <span class="kn">import</span> <span class="n">views</span> <span class="k">as</span> <span class="n">auth_views</span>
<span class="kn">from</span> <span class="nn">django.core.urlresolvers</span> <span class="kn">import</span> <span class="n">reverse</span>
<span class="kn">from</span> <span class="nn">django.urls</span> <span class="kn">import</span> <span class="n">resolve</span>
<span class="kn">from</span> <span class="nn">django.test</span> <span class="kn">import</span> <span class="n">TestCase</span>

<span class="k">class</span> <span class="nc">PasswordResetCompleteTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_reset_complete'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_view_function</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">view</span> <span class="o">=</span> <span class="n">resolve</span><span class="p">(</span><span class="s">'/reset/complete/'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="n">view</span><span class="o">.</span><span class="n">func</span><span class="o">.</span><span class="n">view_class</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordResetCompleteView</span><span class="p">)</span></code></pre></figure>

<hr />
