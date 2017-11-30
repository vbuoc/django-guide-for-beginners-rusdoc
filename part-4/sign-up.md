#### Sign Up

![Sign Up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Sign_Up_View.png)

Let’s start by creating the sign up view. First thing, create a new route in the **urls.py** file:

**myproject/urls.py**

```

    from django.conf.urls import url
    from django.contrib import admin

    from accounts import views as accounts_views
    from boards import views

    urlpatterns = [
        url(r'^/figure>, views.home, name='home'),
        url(r'^signup//figure>, accounts_views.signup, name='signup'),
        url(r'^boards/(?P<pk>\d+)//figure>, views.board_topics, name='board_topics'),
        url(r'^boards/(?P<pk>\d+)/new//figure>, views.new_topic, name='new_topic'),
        url(r'^admin/', admin.site.urls),
    ]

```

Notice how we are importing the **views** module from the **accounts** app in a different way:

```

    from accounts import views as accounts_views

```

We are giving an alias because otherwise, it would clash with the **boards’** views. We can improve the **urls.py** design later on. But for now, let’s focus on the authentication features.

Now edit the **views.py** inside the **accounts** app and create a new view named **signup**:

**accounts/views.py**

```

    from django.shortcuts import render

    def signup(request):
        return render(request, 'signup.html')

```

Create the new template, named **signup.html**:

**templates/signup.html**

```

    {% extends 'base.html' %}

    {% block content %}
      <h2>Sign up</h2>
    {% endblock %}

```

Open the URL **http://127.0.0.1:8000/signup/** in the browser, check if everything is working:

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup.png)

Time to write some tests:

**accounts/tests.py**

```

    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase
    from .views import signup

    class SignUpTests(TestCase):
        def test_signup_status_code(self):
            url = reverse('signup')
            response = self.client.get(url)
            self.assertEquals(response.status_code, 200)

        def test_signup_url_resolves_signup_view(self):
            view = resolve('/signup/')
            self.assertEquals(view.func, signup)

```

Testing the status code (200 = success) and if the URL **/signup/** is returning the correct view function.

```

    python manage.py test

```

```

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    ..................
    ----------------------------------------------------------------------
    Ran 18 tests in 0.652s

    OK
    Destroying test database for alias 'default'...

```

For the authentication views (sign up, log in, password reset, etc.) we won’t use the top bar or the breadcrumb. We can still use the **base.html** template. It just needs some tweaks:

**templates/base.html**

```

    {% load static %}<!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>{% block title %}Django Boards{% endblock %}</title>
        <link href="https://fonts.googleapis.com/css?family=Peralta" rel="stylesheet">
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
        <link rel="stylesheet" href="{% static 'css/app.css' %}">
        {% block stylesheet %}{% endblock %}  <!-- HERE -->
      </head>
      <body>
        {% block body %}  <!-- HERE -->
          <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
            <div class="container">
              <a class="navbar-brand" href="{% url 'home' %}">Django Boards</a>
            </div>
          </nav>
          <div class="container">
            <ol class="breadcrumb my-4">
              {% block breadcrumb %}
              {% endblock %}
            </ol>
            {% block content %}
            {% endblock %}
          </div>
        {% endblock body %}  <!-- AND HERE -->
      </body>
    </html>

```

I marked with comments the new bits in the **base.html** template. The block `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">stylesheet</span> <span class="w"></span> <span class="err">%</span><span class="p">}{</span><span class="err">%</span> <span class="w"></span> <span class="err">endblock</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` will be used to add extra CSS, specific to some pages.

The block `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">body</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` is wrapping the whole HTML document. We can use it to have an empty document taking advantage of the head of the **base.html**. Notice how we named the end block `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">endblock</span> <span class="w"></span> <span class="err">body</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>`. In cases like this, it’s a good practice to name the closing tag, so it’s easier to identify where it ends.

Now on the **signup.html** template, instead of using the `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">content</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>`, we can use the `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">block</span> <span class="w"></span> <span class="err">body</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>`.

**templates/signup.html**

```

    {% extends 'base.html' %}

    {% block body %}
      <h2>Sign up</h2>
    {% endblock %}

```

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-2.png)

![Too Empty](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Empty.png)

Time to create the sign up form. Django has a built-in form named **UserCreationForm**. Let’s use it:

**accounts/views.py**

```

    from django.contrib.auth.forms import UserCreationForm
    from django.shortcuts import render

    def signup(request):
        form = UserCreationForm()
        return render(request, 'signup.html', {'form': form})

```

**templates/signup.html**

```

    {% extends 'base.html' %}

    {% block body %}
      <div class="container">
        <h2>Sign up</h2>
        <form method="post" novalidate>
          {% csrf_token %}
          {{ form.as_p }}
          <button type="submit" class="btn btn-primary">Create an account</button>
        </form>
      </div>
    {% endblock %}

```

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-3.png)

Looking a little bit messy, right? We can use our **form.html** template to make it look better:

**templates/signup.html**

```

    {% extends 'base.html' %}

    {% block body %}
      <div class="container">
        <h2>Sign up</h2>
        <form method="post" novalidate>
          {% csrf_token %}
          {% include 'includes/form.html' %}
          <button type="submit" class="btn btn-primary">Create an account</button>
        </form>
      </div>
    {% endblock %}

```

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-4.png)

Uh, almost there. Currently, our **form.html** partial template is displaying some raw HTML. It’s a security feature. By default Django treats all strings as unsafe, escaping all the special characters that may cause trouble. But in this case, we can trust it.

**templates/includes/form.html**

```

    {% load widget_tweaks %}

    {% for field in form %}
      <div class="form-group">
        {{ field.label_tag }}

        <!-- code suppressed for brevity -->

        {% if field.help_text %}
          <small class="form-text text-muted">
            {{ field.help_text|safe }}  <!-- new code here -->
          </small>
        {% endif %}
      </div>
    {% endfor %}

```

Basically, in the previous template we added the option `safe` to the `field.help_text`: `<span class="p">{</span><span class="err">{</span> <span class="w"></span> <span class="err">field.help_text|safe</span> <span class="w"></span> <span class="p">}</span><span class="err">}</span>`.

Save the **form.html** file, and check the sign up page again:

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-5.png)

Now let’s implement the business logic in the **signup** view:

**accounts/views.py**

```

    from django.contrib.auth import login as auth_login
    from django.contrib.auth.forms import UserCreationForm
    from django.shortcuts import render, redirect

    def signup(request):
        if request.method == 'POST':
            form = UserCreationForm(request.POST)
            if form.is_valid():
                user = form.save()
                auth_login(request, user)
                return redirect('home')
        else:
            form = UserCreationForm()
        return render(request, 'signup.html', {'form': form})

```

A basic form processing with a small detail: the **login** function (renamed to **auth_login** to avoid clashing with the built-in login view).

<div class="info">

**Note:** I renamed the `login` function to `auth_login`, but later I realized that Django 1.11 has a class-based view for the login view, `LoginView`, so there was no risk of clashing the names.

On the older versions there was a `auth.login` and `auth.view.login`, which used to cause some confusion, because one was the function that logs the user in, and the other was the view.

Long story short: you can import it just as `login` if you want, it will not cause any problem.

</div>

If the form is valid, a **User** instance is created with the `user = form.save()`. The created user is then passed as an argument to the **auth_login** function, manually authenticating the user. After that, the view redirects the user to the homepage, keeping the flow of the application.

Let’s try it. First, submit some invalid data. Either an empty form, non-matching fields, or an existing username:

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-6.png)

Now fill the form and submit it, check if the user is created and redirected to the homepage:

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-7.png)

##### Referencing the Authenticated User in the Template

How can we know if it worked? Well, we can edit the **base.html** template to add the name of the user on the top bar:

**templates/base.html**

```

    {% block body %}
      <nav class="navbar navbar-expand-sm navbar-dark bg-dark">
        <div class="container">
          <a class="navbar-brand" href="{% url 'home' %}">Django Boards</a>
          <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#mainMenu" aria-controls="mainMenu" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="mainMenu">
            <ul class="navbar-nav ml-auto">
              <li class="nav-item">
                <a class="nav-link" href="#">{{ user.username }}</a>
              </li>
            </ul>
          </div>
        </div>
      </nav>

      <div class="container">
        <ol class="breadcrumb my-4">
          {% block breadcrumb %}
          {% endblock %}
        </ol>
        {% block content %}
        {% endblock %}
      </div>
    {% endblock body %}

```

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-8.png)

##### Testing the Sign Up View

Let’s now improve our test cases:

**accounts/tests.py**

```

    from django.contrib.auth.forms import UserCreationForm
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase
    from .views import signup

    class SignUpTests(TestCase):
        def setUp(self):
            url = reverse('signup')
            self.response = self.client.get(url)

        def test_signup_status_code(self):
            self.assertEquals(self.response.status_code, 200)

        def test_signup_url_resolves_signup_view(self):
            view = resolve('/signup/')
            self.assertEquals(view.func, signup)

        def test_csrf(self):
            self.assertContains(self.response, 'csrfmiddlewaretoken')

        def test_contains_form(self):
            form = self.response.context.get('form')
            self.assertIsInstance(form, UserCreationForm)

```

We changed a little bit the **SignUpTests** class. Defined a **setUp** method, moved the response object to there. Then now we are also testing if there are a form and the CSRF token in the response.

Now we are going to test a successful sign up. This time, let’s create a new class to organize better the tests:

**accounts/tests.py**

```

    from django.contrib.auth.models import User
    from django.contrib.auth.forms import UserCreationForm
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase
    from .views import signup

    class SignUpTests(TestCase):
        # code suppressed...

    class SuccessfulSignUpTests(TestCase):
        def setUp(self):
            url = reverse('signup')
            data = {
                'username': 'john',
                'password1': 'abcdef123456',
                'password2': 'abcdef123456'
            }
            self.response = self.client.post(url, data)
            self.home_url = reverse('home')

        def test_redirection(self):
            '''
            A valid form submission should redirect the user to the home page
            '''
            self.assertRedirects(self.response, self.home_url)

        def test_user_creation(self):
            self.assertTrue(User.objects.exists())

        def test_user_authentication(self):
            '''
            Create a new request to an arbitrary page.
            The resulting response should now have a `user` to its context,
            after a successful sign up.
            '''
            response = self.client.get(self.home_url)
            user = response.context.get('user')
            self.assertTrue(user.is_authenticated)

```

Run the tests.

Using a similar strategy, now let’s create a new class for sign up tests when the data is invalid:

```

    from django.contrib.auth.models import User
    from django.contrib.auth.forms import UserCreationForm
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase
    from .views import signup

    class SignUpTests(TestCase):
        # code suppressed...

    class SuccessfulSignUpTests(TestCase):
        # code suppressed...

    class InvalidSignUpTests(TestCase):
        def setUp(self):
            url = reverse('signup')
            self.response = self.client.post(url, {})  # submit an empty dictionary

        def test_signup_status_code(self):
            '''
            An invalid form submission should return to the same page
            '''
            self.assertEquals(self.response.status_code, 200)

        def test_form_errors(self):
            form = self.response.context.get('form')
            self.assertTrue(form.errors)

        def test_dont_create_user(self):
            self.assertFalse(User.objects.exists())

```

##### Adding the Email Field to the Form

Everything is working, but… The **email address** field is missing. Well, the **UserCreationForm** does not provide an **email** field. But we can extend it.

Create a file named **forms.py** inside the **accounts** folder:

**accounts/forms.py**

```

    from django import forms
    from django.contrib.auth.forms import UserCreationForm
    from django.contrib.auth.models import User

    class SignUpForm(UserCreationForm):
        email = forms.CharField(max_length=254, required=True, widget=forms.EmailInput())
        class Meta:
            model = User
            fields = ('username', 'email', 'password1', 'password2')

```

Now, instead of using the **UserCreationForm** in our **views.py**, let’s import the new form, **SignUpForm**, and use it instead:

**accounts/views.py**

```

    from django.contrib.auth import login as auth_login
    from django.shortcuts import render, redirect

    from .forms import SignUpForm

    def signup(request):
        if request.method == 'POST':
            form = SignUpForm(request.POST)
            if form.is_valid():
                user = form.save()
                auth_login(request, user)
                return redirect('home')
        else:
            form = SignUpForm()
        return render(request, 'signup.html', {'form': form})

```

Just with this small change, everything is already working:

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-9.png)

Remember to change the test case to use the **SignUpForm** instead of **UserCreationForm**:

```

    from .forms import SignUpForm

    class SignUpTests(TestCase):
        # ...

        def test_contains_form(self):
            form = self.response.context.get('form')
            self.assertIsInstance(form, SignUpForm)

    class SuccessfulSignUpTests(TestCase):
        def setUp(self):
            url = reverse('signup')
            data = {
                'username': 'john',
                'email': 'john@doe.com',
                'password1': 'abcdef123456',
                'password2': 'abcdef123456'
            }
            self.response = self.client.post(url, data)
            self.home_url = reverse('home')

        # ...

```

The previous test case would still pass because since **SignUpForm** extends the **UserCreationForm**, _it is_ an instance of **UserCreationForm**.

Now let’s think about what happened for a moment. We added a new form field:

```

    fields = ('username', 'email', 'password1', 'password2')

```

And it automatically reflected in the HTML template. It’s good, right? Well, depends. What if in the future, a new developer wanted to re-use the **SignUpForm** for something else, and add some extra fields to it. Then those new fields would also show up in the **signup.html**, which may not be the desired behavior. This change could pass unnoticed, and we don’t want any surprises.

So let’s create a new test, that verifies the HTML inputs in the template:

**accounts/tests.py**

```

    class SignUpTests(TestCase):
        # ...

        def test_form_inputs(self):
            '''
            The view must contain five inputs: csrf, username, email,
            password1, password2
            '''
            self.assertContains(self.response, '<input', 5)
            self.assertContains(self.response, 'type="text"', 1)
            self.assertContains(self.response, 'type="email"', 1)
            self.assertContains(self.response, 'type="password"', 2)

```

##### Improving the Tests Layout

Alright, so we are testing the inputs and everything, but we still have to test the form itself. Instead of just keep adding tests to the **accounts/tests.py** file, let’s improve the project design a little bit.

Create a new folder named **tests** within the **accounts** folder. Then, inside the **tests** folder, create an empty file named **__init__.py**.

Now, move the **tests.py** file to inside the **tests** folder, and rename it to **test_view_signup.py**.

The final result should be the following:

```

    myproject/
     |-- myproject/
     |    |-- accounts/
     |    |    |-- migrations/
     |    |    |-- tests/
     |    |    |    |-- __init__.py
     |    |    |    +-- test_view_signup.py
     |    |    |-- __init__.py
     |    |    |-- admin.py
     |    |    |-- apps.py
     |    |    |-- models.py
     |    |    +-- views.py
     |    |-- boards/
     |    |-- myproject/
     |    |-- static/
     |    |-- templates/
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

```

Note that since we are using relative import within the context of the apps, we need to fix the imports in the new **test_view_signup.py**:

**accounts/tests/test_view_signup.py**

```

    from django.contrib.auth.models import User
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase

    from ..views import signup
    from ..forms import SignUpForm

```

We are using relative imports inside the app modules so we can have the freedom to rename the Django app later on, without having to fix all the absolute imports.

Now let’s create a new test file, to test the **SignUpForm**. Add a new test file named **test_form_signup.py**:

**accounts/tests/test_form_signup.py**

```

    from django.test import TestCase
    from ..forms import SignUpForm

    class SignUpFormTest(TestCase):
        def test_form_has_fields(self):
            form = SignUpForm()
            expected = ['username', 'email', 'password1', 'password2',]
            actual = list(form.fields)
            self.assertSequenceEqual(expected, actual)

```

It looks very strict, right? For example, if in the future we have to change the **SignUpForm**, to include the user’s first and last name, we will probably end up having to fix a few test cases, even if we didn’t break anything.

![Test Case Alert](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Test_Alerts.png)

Those alerts are useful because they help to bring awareness, especially for newcomers touching the code for the first time. It helps them code with confidence.

##### Improving the Sign Up Template

Let’s work a little bit on it. Here we can use Bootstrap 4 cards components to make it look good.

Go to [https://www.toptal.com/designers/subtlepatterns/](https://www.toptal.com/designers/subtlepatterns/) and find a nice background pattern to use as a background of the accounts pages. Download it, create a new folder named **img** inside the **static** folder, and place the image there.

Then after that, create a new CSS file named **accounts.css** in the **static/css**. The result should be the following:

```

    myproject/
     |-- myproject/
     |    |-- accounts/
     |    |-- boards/
     |    |-- myproject/
     |    |-- static/
     |    |    |-- css/
     |    |    |    |-- accounts.css  <-- here
     |    |    |    |-- app.css
     |    |    |    +-- bootstrap.min.css
     |    |    +-- img/
     |    |    |    +-- shattered.png  <-- here (the name may be different, depending on the patter you downloaded)
     |    |-- templates/
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

```

Now edit the **accounts.css** file:

**static/css/accounts.css**

```

    body {
      background-image: url(../img/shattered.png);
    }

    .logo {
      font-family: 'Peralta', cursive;
    }

    .logo a {
      color: rgba(0,0,0,.9);
    }

    .logo a:hover,
    .logo a:active {
      text-decoration: none;
    }

```

In the **signup.html** template, we can change it to make use of the new CSS and also take the Bootstrap 4 card components into use:

**templates/signup.html**

```

    {% extends 'base.html' %}

    {% load static %}

    {% block stylesheet %}
      <link rel="stylesheet" href="{% static 'css/accounts.css' %}">
    {% endblock %}

    {% block body %}
      <div class="container">
        <h1 class="text-center logo my-4">
          <a href="{% url 'home' %}">Django Boards</a>
        </h1>
        <div class="row justify-content-center">
          <div class="col-lg-8 col-md-10 col-sm-12">
            <div class="card">
              <div class="card-body">
                <h3 class="card-title">Sign up</h3>
                <form method="post" novalidate>
                  {% csrf_token %}
                  {% include 'includes/form.html' %}
                  <button type="submit" class="btn btn-primary btn-block">Create an account</button>
                </form>
              </div>
              <div class="card-footer text-muted text-center">
                Already have an account? <a href="#">Log in</a>
              </div>
            </div>
          </div>
        </div>
      </div>
    {% endblock %}

```

With that, this should be our sign up page right now:

![Sign up](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/signup-10.jpg)

* * *