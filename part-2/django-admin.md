Introduction to Django Admin

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html


<p>When we start a new project, Django already comes configured with the <strong>Django Admin</strong> listed in the <code class="highlighter-rouge">INSTALLED_APPS</code>.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Django_Admin.png" alt="Django Admin Comic" /></p>

<p>A good use case of the Django Admin is for example in a blog; it can be used by the authors to write and publish
articles. Another example is an e-commerce website, where the staff members can create, edit, delete products.</p>

<p>For now, we are going to configure the Django Admin to maintain our application’s boards.</p>

<p>Let’s start by creating an administrator account:</p>

<figure class="highlight"><pre><code class="language-bash" data-lang="bash">python manage.py createsuperuser</code></pre></figure>

<p>Follow the instructions:</p>

<figure class="highlight"><pre><code class="language-text" data-lang="text">Username (leave blank to use 'vitorfs'): admin
Email address: admin@example.com
Password:
Password (again):
Superuser created successfully.</code></pre></figure>

<p>Now open the URL in a web browser: <strong>http://127.0.0.1:8000/admin/</strong></p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-login.png" alt="Django Admin Login" /></p>

<p>Enter the <strong>username</strong> and <strong>password</strong> to log into the administration interface:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin.png" alt="Django Admin" /></p>

<p>It already comes with some features configured. Here we can add <strong>Users</strong> and <strong>Groups</strong> to manage permissions. We will
explore more of those concepts later on.</p>

<p>To add the <strong>Board</strong> model is very straightforward. Open the <strong>admin.py</strong> file in the <strong>boards</strong> directory, and add
the following code:</p>

<p><strong>boards/admin.py</strong></p>

<figure class="highlight"><pre><code class="language-django" data-lang="django">from django.contrib import admin
from .models import Board

admin.site.register(Board)</code></pre></figure>

<p>Save the <strong>admin.py</strong> file, and refresh the page on your web browser:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards.png" alt="Django Admin Boards" /></p>

<p>And that’s it! It’s ready to be used. Click on the <strong>Boards</strong> link to see the list of existing boards:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-list.png" alt="Django Admin Boards List" /></p>

<p>We can add a new board by clicking on the <strong>Add Board</strong> button:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-add.png" alt="Django Admin Boards Add" /></p>

<p>Click on the <strong>save</strong> button:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-list-2.png" alt="Django Admin Boards List" /></p>

<p>We can check if everything is working be opening the <strong>http://127.0.0.1:8000</strong> URL:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-bootstrap-3.png" alt="Boards Homepage" /></p>

<hr />
