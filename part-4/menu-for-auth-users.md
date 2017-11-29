<h4 id="displaying-menu-for-authenticated-users">Displaying Menu For Authenticated Users</h4>

<p>Now we will need to do some tweaks in our <strong>base.html</strong> template. We have to add a dropdown menu with the logout link.</p>

<p>The Bootstrap 4 dropdown component needs jQuery to work.</p>

<p>First, go to <a href="https://jquery.com/download/" target="_blank">jquery.com/download/</a> and download the
<strong>compressed, production jQuery 3.2.1</strong> version.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/jquery-download.jpg" alt="jQuery Download" /></p>

<p>Inside the <strong>static</strong> folder, create a new folder named <strong>js</strong>. Copy the <strong>jquery-3.2.1.min.js</strong> file to there.</p>

<p>Bootstrap 4 also needs a library called <strong>Popper</strong> to work. Go to <a href="https://popper.js.org/" target="_blank">popper.js.org</a>
and download the latest version.</p>

<p>Inside the <strong>popper.js-1.12.5</strong> folder, go to <strong>dist/umd</strong> and copy the file <strong>popper.min.js</strong> to our <strong>js</strong> folder.
Pay attention here; Bootstrap 4 will only work with the <strong>umd/popper.min.js</strong>. So make sure you are copying the right
file.</p>

<p>If you no longer have all the Bootstrap 4 files, download it again from <a href="http://getbootstrap.com/" target="_blank">getbootstrap.com</a>.</p>

<p>Similarly, copy the <strong>bootstrap.min.js</strong> file to our <strong>js</strong> folder as well.</p>

<p>The final result should be:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">myproject/
 |-- myproject/
 |    |-- accounts/
 |    |-- boards/
 |    |-- myproject/
 |    |-- static/
 |    |    |-- css/
 |    |    +-- js/
 |    |         |-- bootstrap.min.js
 |    |         |-- jquery-3.2.1.min.js
 |    |         +-- popper.min.js
 |    |-- templates/
 |    |-- db.sqlite3
 |    +-- manage.py
 +-- venv/</code></pre></figure>

<p>In the bottom of the <strong>base.html</strong> file, add the scripts <em>after</em> the <code class="highlighter-rouge"><span class="p">{</span><span class="err">%</span><span class="w"> </span><span class="err">endblock</span><span class="w"> </span><span class="err">body</span><span class="w"> </span><span class="err">%</span><span class="p">}</span></code>:</p>

<p><strong>templates/base.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="cp">{%</span> <span class="nv">load</span> <span class="nv">static</span> <span class="cp">%}&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;title&gt;</span><span class="cp">{%</span> <span class="k">block</span> <span class="nv">title</span> <span class="cp">%}</span>Django Boards<span class="cp">{%</span> <span class="k">endblock</span> <span class="cp">%}</span><span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">href=</span><span class="s">"https://fonts.googleapis.com/css?family=Peralta"</span> <span class="na">rel=</span><span class="s">"stylesheet"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/bootstrap.min.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'css/app.css'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>
    <span class="cp">{%</span> <span class="k">block</span> <span class="nv">stylesheet</span> <span class="cp">%}{%</span> <span class="k">endblock</span> <span class="cp">%}</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="cp">{%</span> <span class="k">block</span> <span class="nv">body</span> <span class="cp">%}</span>
    <span class="c">&lt;!-- code suppressed for brevity --&gt;</span>
    <span class="cp">{%</span> <span class="k">endblock</span> <span class="nv">body</span> <span class="cp">%}</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/jquery-3.2.1.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/popper.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">static</span> <span class="s1">'js/bootstrap.min.js'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;&lt;/script&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span></code></pre></figure>

<p>If you found the instructions confusing, just download the files using the direct links below:</p>

<ul>
  <li><a href="https://code.jquery.com/jquery-3.2.1.min.js" target="_blank">https://code.jquery.com/jquery-3.2.1.min.js</a></li>
  <li><a href="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.11.0/umd/popper.min.js" target="_blank">https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.11.0/umd/popper.min.js</a></li>
  <li><a href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/js/bootstrap.min.js" target="_blank">https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/js/bootstrap.min.js</a></li>
</ul>

<p>Right-click and <strong>*Save link as…</strong>.</p>

<p>Now we can add the Bootstrap 4 dropdown menu:</p>

<p><strong>templates/base.html</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;nav</span> <span class="na">class=</span><span class="s">"navbar navbar-expand-sm navbar-dark bg-dark"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"navbar-brand"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Django Boards<span class="nt">&lt;/a&gt;</span>
    <span class="nt">&lt;button</span> <span class="na">class=</span><span class="s">"navbar-toggler"</span> <span class="na">type=</span><span class="s">"button"</span> <span class="na">data-toggle=</span><span class="s">"collapse"</span> <span class="na">data-target=</span><span class="s">"#mainMenu"</span> <span class="na">aria-controls=</span><span class="s">"mainMenu"</span> <span class="na">aria-expanded=</span><span class="s">"false"</span> <span class="na">aria-label=</span><span class="s">"Toggle navigation"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"navbar-toggler-icon"</span><span class="nt">&gt;&lt;/span&gt;</span>
    <span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"collapse navbar-collapse"</span> <span class="na">id=</span><span class="s">"mainMenu"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;ul</span> <span class="na">class=</span><span class="s">"navbar-nav ml-auto"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"nav-item dropdown"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"nav-link dropdown-toggle"</span> <span class="na">href=</span><span class="s">"#"</span> <span class="na">id=</span><span class="s">"userMenu"</span> <span class="na">data-toggle=</span><span class="s">"dropdown"</span> <span class="na">aria-haspopup=</span><span class="s">"true"</span> <span class="na">aria-expanded=</span><span class="s">"false"</span><span class="nt">&gt;</span>
            <span class="cp">{{</span> <span class="nv">user.username</span> <span class="cp">}}</span>
          <span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"dropdown-menu dropdown-menu-right"</span> <span class="na">aria-labelledby=</span><span class="s">"userMenu"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"dropdown-item"</span> <span class="na">href=</span><span class="s">"#"</span><span class="nt">&gt;</span>My account<span class="nt">&lt;/a&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"dropdown-item"</span> <span class="na">href=</span><span class="s">"#"</span><span class="nt">&gt;</span>Change password<span class="nt">&lt;/a&gt;</span>
            <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"dropdown-divider"</span><span class="nt">&gt;&lt;/div&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"dropdown-item"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'logout'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Log out<span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;/li&gt;</span>
      <span class="nt">&lt;/ul&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="nt">&lt;/nav&gt;</span></code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/dropdown.png" alt="Dropdown menu" /></p>

<p>Let’s try it. Click on logout:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/logout.png" alt="Logout" /></p>

<p>It’s working. But the dropdown is showing regardless of the user being logged in or not. The difference is that now the
username is empty, and we can only see an arrow.</p>

<p>We can improve it a little bit:</p>

<figure class="highlight"><pre><code class="language-django" data-lang="django"><span class="nt">&lt;nav</span> <span class="na">class=</span><span class="s">"navbar navbar-expand-sm navbar-dark bg-dark"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"container"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"navbar-brand"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'home'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Django Boards<span class="nt">&lt;/a&gt;</span>
    <span class="nt">&lt;button</span> <span class="na">class=</span><span class="s">"navbar-toggler"</span> <span class="na">type=</span><span class="s">"button"</span> <span class="na">data-toggle=</span><span class="s">"collapse"</span> <span class="na">data-target=</span><span class="s">"#mainMenu"</span> <span class="na">aria-controls=</span><span class="s">"mainMenu"</span> <span class="na">aria-expanded=</span><span class="s">"false"</span> <span class="na">aria-label=</span><span class="s">"Toggle navigation"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"navbar-toggler-icon"</span><span class="nt">&gt;&lt;/span&gt;</span>
    <span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"collapse navbar-collapse"</span> <span class="na">id=</span><span class="s">"mainMenu"</span><span class="nt">&gt;</span>
      <span class="cp">{%</span> <span class="k">if</span> <span class="nv">user.is_authenticated</span> <span class="cp">%}</span>
        <span class="nt">&lt;ul</span> <span class="na">class=</span><span class="s">"navbar-nav ml-auto"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;li</span> <span class="na">class=</span><span class="s">"nav-item dropdown"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"nav-link dropdown-toggle"</span> <span class="na">href=</span><span class="s">"#"</span> <span class="na">id=</span><span class="s">"userMenu"</span> <span class="na">data-toggle=</span><span class="s">"dropdown"</span> <span class="na">aria-haspopup=</span><span class="s">"true"</span> <span class="na">aria-expanded=</span><span class="s">"false"</span><span class="nt">&gt;</span>
              <span class="cp">{{</span> <span class="nv">user.username</span> <span class="cp">}}</span>
            <span class="nt">&lt;/a&gt;</span>
            <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"dropdown-menu dropdown-menu-right"</span> <span class="na">aria-labelledby=</span><span class="s">"userMenu"</span><span class="nt">&gt;</span>
              <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"dropdown-item"</span> <span class="na">href=</span><span class="s">"#"</span><span class="nt">&gt;</span>My account<span class="nt">&lt;/a&gt;</span>
              <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"dropdown-item"</span> <span class="na">href=</span><span class="s">"#"</span><span class="nt">&gt;</span>Change password<span class="nt">&lt;/a&gt;</span>
              <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"dropdown-divider"</span><span class="nt">&gt;&lt;/div&gt;</span>
              <span class="nt">&lt;a</span> <span class="na">class=</span><span class="s">"dropdown-item"</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'logout'</span> <span class="cp">%}</span><span class="s">"</span><span class="nt">&gt;</span>Log out<span class="nt">&lt;/a&gt;</span>
            <span class="nt">&lt;/div&gt;</span>
          <span class="nt">&lt;/li&gt;</span>
        <span class="nt">&lt;/ul&gt;</span>
      <span class="cp">{%</span> <span class="k">else</span> <span class="cp">%}</span>
        <span class="nt">&lt;form</span> <span class="na">class=</span><span class="s">"form-inline ml-auto"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"#"</span> <span class="na">class=</span><span class="s">"btn btn-outline-secondary"</span><span class="nt">&gt;</span>Log in<span class="nt">&lt;/a&gt;</span>
          <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"</span><span class="cp">{%</span> <span class="nv">url</span> <span class="s1">'signup'</span> <span class="cp">%}</span><span class="s">"</span> <span class="na">class=</span><span class="s">"btn btn-primary ml-2"</span><span class="nt">&gt;</span>Sign up<span class="nt">&lt;/a&gt;</span>
        <span class="nt">&lt;/form&gt;</span>
      <span class="cp">{%</span> <span class="k">endif</span> <span class="cp">%}</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="nt">&lt;/nav&gt;</span></code></pre></figure>

<p>Now we are telling Django to show the dropdown menu if the user is logged in, and if not, show the log in and sign up
buttons:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/mainmenu.png" alt="Main Menu" /></p>

<hr />
