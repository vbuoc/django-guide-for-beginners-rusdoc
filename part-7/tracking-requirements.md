<h4 id="tracking-requirements">Tracking Requirements</h4>

<p>It’s a good practice to keep track of the project’s dependencies, so to be easier to install it on another machine.</p>

<p>We can check the currently installed Python libraries by running the command:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">pip freeze

dj-database-url==0.4.2
Django==1.11.6
django-widget-tweaks==1.4.1
Markdown==2.6.9
python-decouple==3.1
pytz==2017.2</code></pre></figure>

<p>Create a file named <strong>requirements.txt</strong> in the project root, and add the dependencies there:</p>

<p><strong>requirements.txt</strong></p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">dj-database-url==0.4.2
Django==1.11.6
django-widget-tweaks==1.4.1
Markdown==2.6.9
python-decouple==3.1</code></pre></figure>

<p>I kept the <strong>pytz==2017.2</strong> out, because it is automatically installed by Django.</p>

<p>You can update your source code repository:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">git add .
git commit -m "Add requirements.txt file"
git push origin master</code></pre></figure>

<hr />
