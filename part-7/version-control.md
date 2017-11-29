<h4 id="version-control">Version Control</h4>

<p>Version control is an extremely important topic in software development. Especially when working with teams and
maintaining production code at the same time, several features are being developed in parallel. No matter if it’s
a one developer project or a multiple developers project, every project should use version control.</p>

<p>There are several options of version control systems out there. Perhaps because of the popularity of GitHub, <strong>Git</strong>
become the <em>de facto</em> standard in version control. So if you are not familiar version control, Git is a good place to
start. There are many tutorials, courses, and resources in general so that it’s easy to find help.</p>

<p>GitHub and Code School have a great <a href="https://try.github.io" target="_blank" rel="noopener">interactive tutorial about Git</a>,
which I used years ago when I started moving from SVN to Git. It’s a very good introduction.</p>

<p>This is such an important topic that I probably should have brought it up since the first tutorial. But the truth is I
wanted the focus of this tutorial series to be on Django. If all this is new for you, don’t worry. It’s important to
take one step at a time. Your first project won’t be perfect. It’s important to keep learning and evolving your skills
slowly but with constancy.</p>

<p>A very good thing about Git is that it’s much more than just a version control system. There’s a rich ecosystem of
tools and services built around it. Some good examples are continuous integration, deployment, code review,
code quality, and project management.</p>

<p>Using Git to support the deployment process of Django projects works very well. It’s a convenient way to pull the
latest version from the source code repository or to rollback to a specific version in case of a problem. There are
many services that integrate with Git so to automate test execution and deployment for example.</p>

<p>If you don’t have Git installed on your local machine, grab the installed from <a href="https://git-scm.com/downloads" target="_blank" rel="noopener">https://git-scm.com/downloads</a>.</p>

<h5 id="basic-setup">Basic Setup</h5>

<p>First thing, set your identity:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">git config --global user.name <span class="s2">"Vitor Freitas"</span>
git config --global user.email vitor@simpleisbetterthancomplex.com</code></pre></figure>

<p>In the project root (the same directory as <strong>manage.py</strong> is), initialize a git repository:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git init</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Initialized empty Git repository in /Users/vitorfs/Development/myproject/.git/</code></pre></figure>

<p>Check the status of the repository:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git status</code></pre></figure>

<figure class="highlight"><pre><code class="language-text" data-lang="text">On branch master

Initial commit

Untracked files:
  (use "git add &lt;file&gt;..." to include in what will be committed)

  accounts/
  boards/
  manage.py
  myproject/
  requirements.txt
  static/
  templates/

nothing added to commit but untracked files present (use "git add" to track)</code></pre></figure>

<p>Before we proceed in adding the source files, create a new file named <strong>.gitignore</strong> in the project root. This special
file will help us keep the repository clean, without unnecessary files like cache files or logs for example.</p>

<p>You can grab a <a href="https://github.com/github/gitignore/blob/master/Python.gitignore" target="_blank" rel="noopener">generic .gitignore file for Python projects</a>
from GitHub.</p>

<p>Make sure to rename it from <strong>Python.gitignore</strong> to just <strong>.gitignore</strong> (the dot is important!).</p>

<p>You can complement the <strong>.gitignore</strong> file telling it to ignore SQLite database files for example:</p>

<p><strong>.gitignore</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">__pycache__/
*.py[cod]
.env
venv/


# SQLite database files

*.sqlite3</code></pre></figure>

<p>Now add the files to the repository:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git add .</code></pre></figure>

<p>Notice the dot here. The command above is telling Git to add <em>all</em> untracked files within the current directory.</p>

<p>Now make the first commit:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git commit -m "Initial commit"</code></pre></figure>

<p>Always write a comment telling what this commit is about, briefly describing what have you changed.</p>

<h5 id="remote-repository">Remote Repository</h5>

<p>Now let’s setup <a href="https://github.com" target="_blank" rel="noopener">GitHub</a> as a remote repository. First, create
a free account on GitHub, then confirm your email address. After that, you will be able to create public repositories.</p>

<p>For now, just pick a name for the repository, don’t initialize it with a README, or add a .gitignore or add a license
so far. Make sure you start the repository empty:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/github1.png" alt="GitHub" /></p>

<p>After you create the repository you should see something like this:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/github2.png" alt="GitHub" /></p>

<p>Now let’s configure it as our remote repository:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git remote add origin git@github.com:sibtc/django-boards.git</code></pre></figure>

<p>Now push the code to the remote server, that is, to the GitHub repository:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git push origin master

Counting objects: 84, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (81/81), done.
Writing objects: 100% (84/84), 319.70 KiB | 0 bytes/s, done.
Total 84 (delta 10), reused 0 (delta 0)
remote: Resolving deltas: 100% (10/10), done.
To git@github.com:sibtc/django-boards.git
 * [new branch]      master -&gt; master</code></pre></figure>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/github3.png" alt="GitHub" /></p>

<p>I create this repository just to demonstrate the process to create a remote repository with an existing code base. The
source code of the project is officially hosted in this repository:
<a href="https://github.com/sibtc/django-beginners-guide" target="_blank" rel="noopener">https://github.com/sibtc/django-beginners-guide</a>.</p>

<hr />
