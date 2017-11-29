<h4 id="password-change-view">Password Change View</h4>

<p>This view is meant to be used by logged in users that want to change their password. Usually, those forms are composed
of three fields: old password, new password, and new password confirmation.</p>

<p><strong>myproject/urls.py</strong>
<small><a href="https://gist.github.com/vitorfs/0927898f37831cad0d6a4ec538b8a002#file-urls-py-L31" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">url</span><span class="p">(</span><span class="s">r'^settings/password/$'</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordChangeView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(</span><span class="n">template_name</span><span class="o">=</span><span class="s">'password_change.html'</span><span class="p">),</span>
    <span class="n">name</span><span class="o">=</span><span class="s">'password_change'</span><span class="p">),</span>
<span class="n">url</span><span class="p">(</span><span class="s">r'^settings/password/done/$'</span><span class="p">,</span> <span class="n">auth_views</span><span class="o">.</span><span class="n">PasswordChangeDoneView</span><span class="o">.</span><span class="n">as_view</span><span class="p">(</span><span class="n">template_name</span><span class="o">=</span><span class="s">'password_change_done.html'</span><span class="p">),</span>
    <span class="n">name</span><span class="o">=</span><span class="s">'password_change_done'</span><span class="p">),</span></code></pre></figure>

<p>Those views only works for logged in users. They make use of a view decorator named <code class="highlighter-rouge">@login_required</code>. This decorator
prevents non-authorized users to access this page. If the user is not logged in, Django will redirect them to the login
page.</p>

<p>Now we have to define what is the login URL of our application in the <strong>settings.py</strong>:</p>

<p><strong>myproject/settings.py</strong>
<small><a href="https://gist.github.com/vitorfs/2d3a2c45df7deb025b8206c5a9b55e12#file-settings-py-L133" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="n">LOGIN_URL</span> <span class="o">=</span> <span class="s">'login'</span></code></pre></figure>

<p><strong>templates/password_change.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Change password<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>Change password<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"col-lg-6 col-md-8 col-sm-10"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;form</span> <span class="na">method=</span><span class="s">"post"</span> <span class="na">novalidate</span><span class="nt">&gt;</span>
        <span class="cp">{%</span> <span class="nv">csrf_token</span> <span class="cp">%}</span>
        <span class="cp">{%</span> <span class="k">include</span> <span class="s1">'includes/form.html'</span> <span class="cp">%}</span>
        <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"btn btn-success"</span><span class="nt">&gt;</span>Change password<span class="nt">&lt;/button&gt;</span>
      <span class="nt">&lt;/form&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_change.png" alt="Change Password" /></p>

<p><strong>templates/password_change_done.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="k">extends</span> <span class="s1">'base.html'</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Change password successful<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">breadcrumb</span> <span class="cp">%}</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item"</span><span class="nt">&gt;&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'password_change'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Change password<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
  <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"breadcrumb-item active"</span><span class="nt">&gt;</span>Success<span class="nt">&lt;/li&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span>

<span class="cp">{%</span> <span class="k">block</span> <span class="nv">content</span> <span class="cp">%}</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"alert alert-success"</span> <span class="na">role=</span><span class="s">"alert"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;strong&gt;</span>Success!<span class="nt">&lt;/strong&gt;</span> Your password has been changed!
  <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-secondary"</span><span class="nt">&gt;</span>Return to home page<span class="nt">&lt;/a&gt;</span>
<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_change_done.png" alt="Change Password Successful" /></p>

<p>Regarding the password change view, we can implement similar test cases as we have already been doing so far. Create
a new test file named <strong>test_view_password_change.py</strong>.</p>

<p>I will list below new types of tests. You can check all the tests I wrote for the password change view clicking in the
<em>view complete file contents</em> link next to the code snippet. Most of the tests are similar to what we have been
doing so far. I moved to an external file to avoid being too repetitive.</p>

<p><strong>accounts/tests/test_view_password_change.py</strong>
<small><a href="https://gist.github.com/vitorfs/03e2f20a4c1e693c9b22b343503fb461#file-test_view_password_change-py-L40" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">LoginRequiredPasswordChangeTests</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_change'</span><span class="p">)</span>
        <span class="n">login_url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'login'</span><span class="p">)</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="n">response</span><span class="p">,</span> <span class="n">f</span><span class="s">'{login_url}?next={url}'</span><span class="p">)</span></code></pre></figure>

<p>The test above tries to access the <strong>password_change</strong> view without being logged in. The expected behavior is to
redirect the user to the login page.</p>

<p><strong>accounts/tests/test_view_password_change.py</strong>
<small><a href="https://gist.github.com/vitorfs/03e2f20a4c1e693c9b22b343503fb461#file-test_view_password_change-py-L48" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">PasswordChangeTestCase</span><span class="p">(</span><span class="n">TestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">data</span><span class="o">=</span><span class="p">{}):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">user</span> <span class="o">=</span> <span class="n">User</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">create_user</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">email</span><span class="o">=</span><span class="s">'john@doe.com'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'old_password'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">url</span> <span class="o">=</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_change'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">login</span><span class="p">(</span><span class="n">username</span><span class="o">=</span><span class="s">'john'</span><span class="p">,</span> <span class="n">password</span><span class="o">=</span><span class="s">'old_password'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">post</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">url</span><span class="p">,</span> <span class="n">data</span><span class="p">)</span></code></pre></figure>

<p>Here we defined a new class named <strong>PasswordChangeTestCase</strong>. It does a basic setup, creating a user and making a
<strong>POST</strong> request to the <strong>password_change</strong> view. In the next set of test cases, we are going to use this class
instead of the <strong>TestCase</strong> class and test a successful request and an invalid request:</p>

<p><strong>accounts/tests/test_view_password_change.py</strong>
<small><a href="https://gist.github.com/vitorfs/03e2f20a4c1e693c9b22b343503fb461#file-test_view_password_change-py-L60" target="_blank" rel="noopener nofollow">(view complete file contents)</a></small></p>

<figure class="highlight"><pre><code class="language-python" data-lang="python"><span class="k">class</span> <span class="nc">SuccessfulPasswordChangeTests</span><span class="p">(</span><span class="n">PasswordChangeTestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">setUp</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="nb">super</span><span class="p">()</span><span class="o">.</span><span class="n">setUp</span><span class="p">({</span>
            <span class="s">'old_password'</span><span class="p">:</span> <span class="s">'old_password'</span><span class="p">,</span>
            <span class="s">'new_password1'</span><span class="p">:</span> <span class="s">'new_password'</span><span class="p">,</span>
            <span class="s">'new_password2'</span><span class="p">:</span> <span class="s">'new_password'</span><span class="p">,</span>
        <span class="p">})</span>

    <span class="k">def</span> <span class="nf">test_redirection</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        A valid form submission should redirect the user
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertRedirects</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="p">,</span> <span class="n">reverse</span><span class="p">(</span><span class="s">'password_change_done'</span><span class="p">))</span>

    <span class="k">def</span> <span class="nf">test_password_changed</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        refresh the user instance from database to get the new password
        hash updated by the change password view.
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">user</span><span class="o">.</span><span class="n">refresh_from_db</span><span class="p">()</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">user</span><span class="o">.</span><span class="n">check_password</span><span class="p">(</span><span class="s">'new_password'</span><span class="p">))</span>

    <span class="k">def</span> <span class="nf">test_user_authentication</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        Create a new request to an arbitrary page.
        The resulting response should now have an `user` to its context, after a successful sign up.
        '''</span>
        <span class="n">response</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">reverse</span><span class="p">(</span><span class="s">'home'</span><span class="p">))</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'user'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">user</span><span class="o">.</span><span class="n">is_authenticated</span><span class="p">)</span>


<span class="k">class</span> <span class="nc">InvalidPasswordChangeTests</span><span class="p">(</span><span class="n">PasswordChangeTestCase</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">test_status_code</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        An invalid form submission should return to the same page
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertEquals</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">status_code</span><span class="p">,</span> <span class="mi">200</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_form_errors</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">form</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">response</span><span class="o">.</span><span class="n">context</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="s">'form'</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="n">form</span><span class="o">.</span><span class="n">errors</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">test_didnt_change_password</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="s">'''
        refresh the user instance from the database to make
        sure we have the latest data.
        '''</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">user</span><span class="o">.</span><span class="n">refresh_from_db</span><span class="p">()</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">assertTrue</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">user</span><span class="o">.</span><span class="n">check_password</span><span class="p">(</span><span class="s">'old_password'</span><span class="p">))</span></code></pre></figure>

<p>The <code class="highlighter-rouge">refresh_from_db()</code> method make sure we have the latest state of the data. It forces Django to query the database
again to update the data. We have to do it because the <strong>change_password</strong> view update the password in the database.
So to test if the password <em>really</em> changed, we have to grab the latest data from the database.</p>

<hr />
