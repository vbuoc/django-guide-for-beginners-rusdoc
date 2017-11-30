#### Login

First thing, add a new URL route:

**myproject/urls.py**

```

    from django.conf.urls import url
    from django.contrib import admin
    from django.contrib.auth import views as auth_views

    from accounts import views as accounts_views
    from boards import views

    urlpatterns = [
        url(r'^/figure>, views.home, name='home'),
        url(r'^signup//figure>, accounts_views.signup, name='signup'),
        url(r'^login//figure>, auth_views.LoginView.as_view(template_name='login.html'), name='login'),
        url(r'^logout//figure>, auth_views.LogoutView.as_view(), name='logout'),
        url(r'^boards/(?P<pk>\d+)//figure>, views.board_topics, name='board_topics'),
        url(r'^boards/(?P<pk>\d+)/new//figure>, views.new_topic, name='new_topic'),
        url(r'^admin/', admin.site.urls),
    ]

```

Inside the `as_view()` we can pass some extra parameters, so to override the defaults. In this case, we are instructing the **LoginView** to look for a template at **login.html**.

Edit the **settings.py** and add the following configuration:

**myproject/settings.py**

```

    LOGIN_REDIRECT_URL = 'home'

```

This configuration is telling Django where to redirect the user after a successful login.

Finally, add the login URL to the **base.html** template:

**templates/base.html**

```

    <a href="{% url 'login' %}" class="btn btn-outline-secondary">Log in</a>

```

We can create a template similar to the sign up page. Create a new file named **login.html**:

**templates/login.html**

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
          <div class="col-lg-4 col-md-6 col-sm-8">
            <div class="card">
              <div class="card-body">
                <h3 class="card-title">Log in</h3>
                <form method="post" novalidate>
                  {% csrf_token %}
                  {% include 'includes/form.html' %}
                  <button type="submit" class="btn btn-primary btn-block">Log in</button>
                </form>
              </div>
              <div class="card-footer text-muted text-center">
                New to Django Boards? <a href="{% url 'signup' %}">Sign up</a>
              </div>
            </div>
            <div class="text-center py-2">
              <small>
                <a href="#" class="text-muted">Forgot your password?</a>
              </small>
            </div>
          </div>
        </div>
      </div>
    {% endblock %}

```

![Login](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login.jpg)

And we are repeating HTML templates. Let’s refactor it.

Create a new template named **base_accounts.html**:

**templates/base_accounts.html**

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
        {% block content %}
        {% endblock %}
      </div>
    {% endblock %}

```

Now use it on both **signup.html** and **login.html**:

**templates/login.html**

```

    {% extends 'base_accounts.html' %}

    {% block title %}Log in to Django Boards{% endblock %}

    {% block content %}
      <div class="row justify-content-center">
        <div class="col-lg-4 col-md-6 col-sm-8">
          <div class="card">
            <div class="card-body">
              <h3 class="card-title">Log in</h3>
              <form method="post" novalidate>
                {% csrf_token %}
                {% include 'includes/form.html' %}
                <button type="submit" class="btn btn-primary btn-block">Log in</button>
              </form>
            </div>
            <div class="card-footer text-muted text-center">
              New to Django Boards? <a href="{% url 'signup' %}">Sign up</a>
            </div>
          </div>
          <div class="text-center py-2">
            <small>
              <a href="#" class="text-muted">Forgot your password?</a>
            </small>
          </div>
        </div>
      </div>
    {% endblock %}

```

We still don’t have the password reset URL, so let’s leave it as `#` for now.

**templates/signup.html**

```

    {% extends 'base_accounts.html' %}

    {% block title %}Sign up to Django Boards{% endblock %}

    {% block content %}
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
              Already have an account? <a href="{% url 'login' %}">Log in</a>
            </div>
          </div>
        </div>
      </div>
    {% endblock %}

```

Notice that we added the log in URL: `<a href="{% url 'login' %}">Log in</a>`.

##### Log In Non Field Errors

If we submit the log in form empty, we get some nice error messages:

![Login](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-1.jpg)

But if we submit an username that doesn’t exist or an invalid password, right now that’s what’s going to happen:

![Login](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-2.jpg)

A little bit misleading. The fields are showing green, suggesting they are okay. Also, there’s no message saying anything.

That’s because forms have a special type of error, which is called **non-field errors**. It’s a collection of errors that are not related to a specific field. Let’s refactor the **form.html** partial template to display those errors as well:

**templates/includes/form.html**

```

    {% load widget_tweaks %}

    {% if form.non_field_errors %}
      <div class="alert alert-danger" role="alert">
        {% for error in form.non_field_errors %}
          <p{% if forloop.last %} class="mb-0"{% endif %}>{{ error }}</p>
        {% endfor %}
      </div>
    {% endif %}

    {% for field in form %}
      <!-- code suppressed -->
    {% endfor %}

```

The `<span class="p">{</span><span class="err">%</span> <span class="w"></span> <span class="err">if</span> <span class="w"></span> <span class="err">forloop.last</span> <span class="w"></span> <span class="err">%</span><span class="p">}</span>` is just a minor thing. Because the `p` tag has a `margin-bottom`. And a form may have several non-field errors. For each non-field error, we render a `p` tag with the error. Then I’m checking if it’s the last error to render. If so, we add a Bootstrap 4 CSS class `mb-0` which stands for “margin bottom = 0”. Then the alert doesn’t look weird, with some extra space. Again, just a very minor detail. I did that just to keep the consistency of the spacing.

![Login](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-3.jpg)

We still have to deal with the password field though. The thing is, Django never returned the data of password fields to the client. So, instead of trying to do something smart, let’s just ignore the `is-valid` and `is-invalid` CSS classes in some cases. But our form template already looks complicated. We can move some of the code to a **template tag**.

##### Creating Custom Template Tags

Inside the **boards** app, create a new folder named **templatetags**. Then inside this folder, create two empty files named **__init__.py** and **form_tags.py**.

The structure should be the following:

```

    myproject/
     |-- myproject/
     |    |-- accounts/
     |    |-- boards/
     |    |    |-- migrations/
     |    |    |-- templatetags/        <-- here
     |    |    |    |-- __init__.py
     |    |    |    +-- form_tags.py
     |    |    |-- __init__.py
     |    |    |-- admin.py
     |    |    |-- apps.py
     |    |    |-- models.py
     |    |    |-- tests.py
     |    |    +-- views.py
     |    |-- myproject/
     |    |-- static/
     |    |-- templates/
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

```

In the **form_tags.py** file, let’s create two template tags:

**boards/templatetags/form_tags.py**

```

    from django import template

    register = template.Library()

    @register.filter
    def field_type(bound_field):
        return bound_field.field.widget.__class__.__name__

    @register.filter
    def input_class(bound_field):
        css_class = ''
        if bound_field.form.is_bound:
            if bound_field.errors:
                css_class = 'is-invalid'
            elif field_type(bound_field) != 'PasswordInput':
                css_class = 'is-valid'
        return 'form-control {}'.format(css_class)

```

Those are _template filters_. They work like this:

First, we load it in a template as we do with the **widget_tweaks** or **static** template tags. Note that after creating this file, you will have to manually stop the development server and start it again so Django can identify the new template tags.

```

    {% load form_tags %}

```

Then after that, we can use them in a template:

```

    {{ form.username|field_type }}

```

Will return:

```

    'TextInput'

```

Or in case of the **input_class**:

```

    {{ form.username|input_class }}

    <!-- if the form is not bound, it will simply return: -->
    'form-control '

    <!-- if the form is bound and valid: -->
    'form-control is-valid'

    <!-- if the form is bound and invalid: -->
    'form-control is-invalid'

```

Now update the **form.html** to use the new template tags:

**templates/includes/form.html**

```

    {% load form_tags widget_tweaks %}

    {% if form.non_field_errors %}
      <div class="alert alert-danger" role="alert">
        {% for error in form.non_field_errors %}
          <p{% if forloop.last %} class="mb-0"{% endif %}>{{ error }}</p>
        {% endfor %}
      </div>
    {% endif %}

    {% for field in form %}
      <div class="form-group">
        {{ field.label_tag }}
        {% render_field field class=field|input_class %}
        {% for error in field.errors %}
          <div class="invalid-feedback">
            {{ error }}
          </div>
        {% endfor %}
        {% if field.help_text %}
          <small class="form-text text-muted">
            {{ field.help_text|safe }}
          </small>
        {% endif %}
      </div>
    {% endfor %}

```

Much better, right? Reduced the complexity of the template. It looks cleaner now. And it also solved the problem with the password field displaying a green border:

![Login](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/login-4.jpg)

##### Testing the Template Tags

First, let’s just organize the **boards’** tests a little bit. Like we did with the **accounts** app, create a new folder named **tests**, add a **__init__.py**, copy the **tests.py** and rename it to just **test_views.py** for now.

Add a new empty file named **test_templatetags.py**.

```

    myproject/
     |-- myproject/
     |    |-- accounts/
     |    |-- boards/
     |    |    |-- migrations/
     |    |    |-- templatetags/
     |    |    |-- tests/
     |    |    |    |-- __init__.py
     |    |    |    |-- test_templatetags.py  <-- new file, empty for now
     |    |    |    +-- test_views.py  <-- our old file with all the tests
     |    |    |-- __init__.py
     |    |    |-- admin.py
     |    |    |-- apps.py
     |    |    |-- models.py
     |    |    +-- views.py
     |    |-- myproject/
     |    |-- static/
     |    |-- templates/
     |    |-- db.sqlite3
     |    +-- manage.py
     +-- venv/

```

Fix the **test_views.py** imports:

**boards/tests/test_views.py**

```

    from ..views import home, board_topics, new_topic
    from ..models import Board, Topic, Post
    from ..forms import NewTopicForm

```

Execute the tests just to make sure everything is working.

**boards/tests/test_templatetags.py**

```

    from django import forms
    from django.test import TestCase
    from ..templatetags.form_tags import field_type, input_class

    class ExampleForm(forms.Form):
        name = forms.CharField()
        password = forms.CharField(widget=forms.PasswordInput())
        class Meta:
            fields = ('name', 'password')

    class FieldTypeTests(TestCase):
        def test_field_widget_type(self):
            form = ExampleForm()
            self.assertEquals('TextInput', field_type(form['name']))
            self.assertEquals('PasswordInput', field_type(form['password']))

    class InputClassTests(TestCase):
        def test_unbound_field_initial_state(self):
            form = ExampleForm()  # unbound form
            self.assertEquals('form-control ', input_class(form['name']))

        def test_valid_bound_field(self):
            form = ExampleForm({'name': 'john', 'password': '123'})  # bound form (field + data)
            self.assertEquals('form-control is-valid', input_class(form['name']))
            self.assertEquals('form-control ', input_class(form['password']))

        def test_invalid_bound_field(self):
            form = ExampleForm({'name': '', 'password': '123'})  # bound form (field + data)
            self.assertEquals('form-control is-invalid', input_class(form['name']))

```

We created a form class to be used in the tests then added test cases covering the possible scenarios in the two template tags.

```

    python manage.py test

```

```

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    ................................
    ----------------------------------------------------------------------
    Ran 32 tests in 0.846s

    OK
    Destroying test database for alias 'default'...

```

* * *