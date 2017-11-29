#### Password Reset

The password reset process involves some nasty URL patterns. But as we discussed in the previous tutorial, we don’t need to be an expert in regular expressions. It’s just a matter of knowing the common ones.

Another important thing before we start is that, for the password reset process, we need to send emails. It’s a little bit complicated in the beginning because we need an external service. For now, we won’t be configuring a production quality email service. In fact, during the development phase, we can use Django’s debug tools to check if the emails are being sent correctly.

![Comic Email](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Email.png)

##### Console Email Backend

The idea is during the development of the project, instead of sending real emails, we just log them. There are two options: writing all emails in a text file or simply displaying them in the console. I find the latter option more convenient because we are already using a console to run the development server and the setup is a bit easier.

Edit the **settings.py** module and add the `EMAIL_BACKEND` variable to the end of the file:

**myproject/settings.py**

<figure class="highlight">

    EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

</figure>

##### Configuring the Routes

The password reset process requires four views:

*   A page with a form to start the reset process;
*   A success page saying the process initiated, instructing the user to check their spam folders, etc.;
*   A page to check the token sent via email;
*   A page to tell the user if the reset was successful or not.

The views are built-in, we don’t need to implement anything. All we need to do is add the routes to the **urls.py** and create the templates.

**myproject/urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/117e300e00d5685f7186e09260f82736#file-urls-py-L14)</small>

<figure class="highlight">

    url(r'^reset//figure>,
        auth_views.PasswordResetView.as_view(
            template_name='password_reset.html',
            email_template_name='password_reset_email.html',
            subject_template_name='password_reset_subject.txt'
        ),
        name='password_reset'),
    url(r'^reset/done//figure>,
        auth_views.PasswordResetDoneView.as_view(template_name='password_reset_done.html'),
        name='password_reset_done'),
    url(r'^reset/(?P<uidb64>[0-9A-Za-z_\-]+)/(?P<token>[0-9A-Za-z]{1,13}-[0-9A-Za-z]{1,20})//figure>,
        auth_views.PasswordResetConfirmView.as_view(template_name='password_reset_confirm.html'),
        name='password_reset_confirm'),
    url(r'^reset/complete//figure>,
        auth_views.PasswordResetCompleteView.as_view(template_name='password_reset_complete.html'),
        name='password_reset_complete'),
    ]

</figure>

The `template_name` parameter in the password reset views are optional. But I thought it would be a good idea to re-define it, so the link between the view and the template be more obvious than just using the defaults.

Inside the **templates** folder, the following template files:

*   **password_reset.html**
*   **password_reset_email.html**: this template is the body of the email message sent to the user
*   **password_reset_subject.txt**: this template is the subject line of the email, it should be a single line file
*   **password_reset_done.html**
*   **password_reset_confirm.html**
*   **password_reset_complete.html**

Before we start implementing the templates, let’s prepare a new test file.

We can add just some basic tests because those views and forms are already tested in the Django code. We are going to test just the specifics of our application.

Create a new test file named **test_view_password_reset.py** inside the **accounts/tests** folder.

##### Password Reset View

**templates/password_reset.html**

<figure class="highlight">

    {% extends 'base_accounts.html' %}

    {% block title %}Reset your password{% endblock %}

    {% block content %}
      <div class="row justify-content-center">
        <div class="col-lg-4 col-md-6 col-sm-8">
          <div class="card">
            <div class="card-body">
              <h3 class="card-title">Reset your password</h3>
              <p>Enter your email address and we will send you a link to reset your password.</p>
              <form method="post" novalidate>
                {% csrf_token %}
                {% include 'includes/form.html' %}
                <button type="submit" class="btn btn-primary btn-block">Send password reset email</button>
              </form>
            </div>
          </div>
        </div>
      </div>
    {% endblock %}

</figure>

![Password Reset](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset.jpg)

**accounts/tests/test_view_password_reset.py**

<figure class="highlight">

    from django.contrib.auth import views as auth_views
    from django.contrib.auth.forms import PasswordResetForm
    from django.contrib.auth.models import User
    from django.core import mail
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase

    class PasswordResetTests(TestCase):
        def setUp(self):
            url = reverse('password_reset')
            self.response = self.client.get(url)

        def test_status_code(self):
            self.assertEquals(self.response.status_code, 200)

        def test_view_function(self):
            view = resolve('/reset/')
            self.assertEquals(view.func.view_class, auth_views.PasswordResetView)

        def test_csrf(self):
            self.assertContains(self.response, 'csrfmiddlewaretoken')

        def test_contains_form(self):
            form = self.response.context.get('form')
            self.assertIsInstance(form, PasswordResetForm)

        def test_form_inputs(self):
            '''
            The view must contain two inputs: csrf and email
            '''
            self.assertContains(self.response, '<input', 2)
            self.assertContains(self.response, 'type="email"', 1)

    class SuccessfulPasswordResetTests(TestCase):
        def setUp(self):
            email = 'john@doe.com'
            User.objects.create_user(username='john', email=email, password='123abcdef')
            url = reverse('password_reset')
            self.response = self.client.post(url, {'email': email})

        def test_redirection(self):
            '''
            A valid form submission should redirect the user to `password_reset_done` view
            '''
            url = reverse('password_reset_done')
            self.assertRedirects(self.response, url)

        def test_send_password_reset_email(self):
            self.assertEqual(1, len(mail.outbox))

    class InvalidPasswordResetTests(TestCase):
        def setUp(self):
            url = reverse('password_reset')
            self.response = self.client.post(url, {'email': 'donotexist@email.com'})

        def test_redirection(self):
            '''
            Even invalid emails in the database should
            redirect the user to `password_reset_done` view
            '''
            url = reverse('password_reset_done')
            self.assertRedirects(self.response, url)

        def test_no_reset_email_sent(self):
            self.assertEqual(0, len(mail.outbox))

</figure>

**templates/password_reset_subject.txt**

<figure class="highlight">

    [Django Boards] Please reset your password

</figure>

**templates/password_reset_email.html**

<figure class="highlight">

    Hi there,

    Someone asked for a password reset for the email address {{ email }}.
    Follow the link below:
    {{ protocol }}://{{ domain }}{% url 'password_reset_confirm' uidb64=uid token=token %}

    In case you forgot your Django Boards username: {{ user.username }}

    If clicking the link above doesn't work, please copy and paste the URL
    in a new browser window instead.

    If you've received this mail in error, it's likely that another user entered
    your email address by mistake while trying to reset a password. If you didn't
    initiate the request, you don't need to take any further action and can safely
    disregard this email.

    Thanks,

    The Django Boards Team

</figure>

![Password Reset Email](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_email.png)

We can create a specific file to test the email message. Create a new file named **test_mail_password_reset.py** inside the **accounts/tests** folder:

**accounts/tests/test_mail_password_reset.py**

<figure class="highlight">

    from django.core import mail
    from django.contrib.auth.models import User
    from django.urls import reverse
    from django.test import TestCase

    class PasswordResetMailTests(TestCase):
        def setUp(self):
            User.objects.create_user(username='john', email='john@doe.com', password='123')
            self.response = self.client.post(reverse('password_reset'), { 'email': 'john@doe.com' })
            self.email = mail.outbox[0]

        def test_email_subject(self):
            self.assertEqual('[Django Boards] Please reset your password', self.email.subject)

        def test_email_body(self):
            context = self.response.context
            token = context.get('token')
            uid = context.get('uid')
            password_reset_token_url = reverse('password_reset_confirm', kwargs={
                'uidb64': uid,
                'token': token
            })
            self.assertIn(password_reset_token_url, self.email.body)
            self.assertIn('john', self.email.body)
            self.assertIn('john@doe.com', self.email.body)

        def test_email_to(self):
            self.assertEqual(['john@doe.com',], self.email.to)

</figure>

This test case grabs the email sent by the application, and examine the subject line, the body contents, and to who was the email sent to.

##### Password Reset Done View

**templates/password_reset_done.html**

<figure class="highlight">

    {% extends 'base_accounts.html' %}

    {% block title %}Reset your password{% endblock %}

    {% block content %}
      <div class="row justify-content-center">
        <div class="col-lg-4 col-md-6 col-sm-8">
          <div class="card">
            <div class="card-body">
              <h3 class="card-title">Reset your password</h3>
              <p>Check your email for a link to reset your password. If it doesn't appear within a few minutes, check your spam folder.</p>
              <a href="{% url 'login' %}" class="btn btn-secondary btn-block">Return to log in</a>
            </div>
          </div>
        </div>
      </div>
    {% endblock %}

</figure>

![Password Reset Done](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_done.jpg)

**accounts/tests/test_view_password_reset.py**

<figure class="highlight">

    from django.contrib.auth import views as auth_views
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase

    class PasswordResetDoneTests(TestCase):
        def setUp(self):
            url = reverse('password_reset_done')
            self.response = self.client.get(url)

        def test_status_code(self):
            self.assertEquals(self.response.status_code, 200)

        def test_view_function(self):
            view = resolve('/reset/done/')
            self.assertEquals(view.func.view_class, auth_views.PasswordResetDoneView)

</figure>

##### Password Reset Confirm View

**templates/password_reset_confirm.html**

<figure class="highlight">

    {% extends 'base_accounts.html' %}

    {% block title %}
      {% if validlink %}
        Change password for {{ form.user.username }}
      {% else %}
        Reset your password
      {% endif %}
    {% endblock %}

    {% block content %}
      <div class="row justify-content-center">
        <div class="col-lg-6 col-md-8 col-sm-10">
          <div class="card">
            <div class="card-body">
              {% if validlink %}
                <h3 class="card-title">Change password for @{{ form.user.username }}</h3>
                <form method="post" novalidate>
                  {% csrf_token %}
                  {% include 'includes/form.html' %}
                  <button type="submit" class="btn btn-success btn-block">Change password</button>
                </form>
              {% else %}
                <h3 class="card-title">Reset your password</h3>
                <div class="alert alert-danger" role="alert">
                  It looks like you clicked on an invalid password reset link. Please try again.
                </div>
                <a href="{% url 'password_reset' %}" class="btn btn-secondary btn-block">Request a new password reset link</a>
              {% endif %}
            </div>
          </div>
        </div>
      </div>
    {% endblock %}

</figure>

This page can only be accessed with the link sent in the email. It looks like this: **http://127.0.0.1:8000/reset/Mw/4po-2b5f2d47c19966e294a1/**

During the development phase, grab this link from the email in the console.

If the link is valid:

![Password Reset Confirm Valid](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_confirm_valid.jpg)

Or if the link has already been used:

![Password Reset Confirm Invalid](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_confirm_invalid.jpg)

**accounts/tests/test_view_password_reset.py**

<figure class="highlight">

    from django.contrib.auth.tokens import default_token_generator
    from django.utils.encoding import force_bytes
    from django.utils.http import urlsafe_base64_encode
    from django.contrib.auth import views as auth_views
    from django.contrib.auth.forms import SetPasswordForm
    from django.contrib.auth.models import User
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase

    class PasswordResetConfirmTests(TestCase):
        def setUp(self):
            user = User.objects.create_user(username='john', email='john@doe.com', password='123abcdef')

            '''
            create a valid password reset token
            based on how django creates the token internally:
            https://github.com/django/django/blob/1.11.5/django/contrib/auth/forms.py#L280
            '''
            self.uid = urlsafe_base64_encode(force_bytes(user.pk)).decode()
            self.token = default_token_generator.make_token(user)

            url = reverse('password_reset_confirm', kwargs={'uidb64': self.uid, 'token': self.token})
            self.response = self.client.get(url, follow=True)

        def test_status_code(self):
            self.assertEquals(self.response.status_code, 200)

        def test_view_function(self):
            view = resolve('/reset/{uidb64}/{token}/'.format(uidb64=self.uid, token=self.token))
            self.assertEquals(view.func.view_class, auth_views.PasswordResetConfirmView)

        def test_csrf(self):
            self.assertContains(self.response, 'csrfmiddlewaretoken')

        def test_contains_form(self):
            form = self.response.context.get('form')
            self.assertIsInstance(form, SetPasswordForm)

        def test_form_inputs(self):
            '''
            The view must contain two inputs: csrf and two password fields
            '''
            self.assertContains(self.response, '<input', 3)
            self.assertContains(self.response, 'type="password"', 2)

    class InvalidPasswordResetConfirmTests(TestCase):
        def setUp(self):
            user = User.objects.create_user(username='john', email='john@doe.com', password='123abcdef')
            uid = urlsafe_base64_encode(force_bytes(user.pk)).decode()
            token = default_token_generator.make_token(user)

            '''
            invalidate the token by changing the password
            '''
            user.set_password('abcdef123')
            user.save()

            url = reverse('password_reset_confirm', kwargs={'uidb64': uid, 'token': token})
            self.response = self.client.get(url)

        def test_status_code(self):
            self.assertEquals(self.response.status_code, 200)

        def test_html(self):
            password_reset_url = reverse('password_reset')
            self.assertContains(self.response, 'invalid password reset link')
            self.assertContains(self.response, 'href="{0}"'.format(password_reset_url))

</figure>

##### Password Reset Complete View

**templates/password_reset_complete.html**

<figure class="highlight">

    {% extends 'base_accounts.html' %}

    {% block title %}Password changed!{% endblock %}

    {% block content %}
      <div class="row justify-content-center">
        <div class="col-lg-6 col-md-8 col-sm-10">
          <div class="card">
            <div class="card-body">
              <h3 class="card-title">Password changed!</h3>
              <div class="alert alert-success" role="alert">
                You have successfully changed your password! You may now proceed to log in.
              </div>
              <a href="{% url 'login' %}" class="btn btn-secondary btn-block">Return to log in</a>
            </div>
          </div>
        </div>
      </div>
    {% endblock %}

</figure>

![Password Reset Complete](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_reset_complete.jpg)

**accounts/tests/test_view_password_reset.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/c9657d39d28c2a0cfb0933e715bfc9cf#file-test_view_password_reset-py-L149)</small>

<figure class="highlight">

    from django.contrib.auth import views as auth_views
    from django.core.urlresolvers import reverse
    from django.urls import resolve
    from django.test import TestCase

    class PasswordResetCompleteTests(TestCase):
        def setUp(self):
            url = reverse('password_reset_complete')
            self.response = self.client.get(url)

        def test_status_code(self):
            self.assertEquals(self.response.status_code, 200)

        def test_view_function(self):
            view = resolve('/reset/complete/')
            self.assertEquals(view.func.view_class, auth_views.PasswordResetCompleteView)

</figure>

* * *