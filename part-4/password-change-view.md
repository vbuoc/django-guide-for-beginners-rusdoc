#### Password Change View

This view is meant to be used by logged in users that want to change their password. Usually, those forms are composed of three fields: old password, new password, and new password confirmation.

**myproject/urls.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/0927898f37831cad0d6a4ec538b8a002#file-urls-py-L31)</small>

<figure class="highlight">

    url(r'^settings/password//figure>, auth_views.PasswordChangeView.as_view(template_name='password_change.html'),
        name='password_change'),
    url(r'^settings/password/done//figure>, auth_views.PasswordChangeDoneView.as_view(template_name='password_change_done.html'),
        name='password_change_done'),

</figure>

Those views only works for logged in users. They make use of a view decorator named `@login_required`. This decorator prevents non-authorized users to access this page. If the user is not logged in, Django will redirect them to the login page.

Now we have to define what is the login URL of our application in the **settings.py**:

**myproject/settings.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/2d3a2c45df7deb025b8206c5a9b55e12#file-settings-py-L133)</small>

<figure class="highlight">

    LOGIN_URL = 'login'

</figure>

**templates/password_change.html**

<figure class="highlight">

    {% extends 'base.html' %}

    {% block title %}Change password{% endblock %}

    {% block breadcrumb %}
      <li class="breadcrumb-item active">Change password</li>
    {% endblock %}

    {% block content %}
      <div class="row">
        <div class="col-lg-6 col-md-8 col-sm-10">
          <form method="post" novalidate>
            {% csrf_token %}
            {% include 'includes/form.html' %}
            <button type="submit" class="btn btn-success">Change password</button>
          </form>
        </div>
      </div>
    {% endblock %}

</figure>

![Change Password](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_change.png)

**templates/password_change_done.html**

<figure class="highlight">

    {% extends 'base.html' %}

    {% block title %}Change password successful{% endblock %}

    {% block breadcrumb %}
      <li class="breadcrumb-item"><a href="{% url 'password_change' %}">Change password</a></li>
      <li class="breadcrumb-item active">Success</li>
    {% endblock %}

    {% block content %}
      <div class="alert alert-success" role="alert">
        <strong>Success!</strong> Your password has been changed!
      </div>
      <a href="{% url 'home' %}" class="btn btn-secondary">Return to home page</a>
    {% endblock %}

</figure>

![Change Password Successful](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/password_change_done.png)

Regarding the password change view, we can implement similar test cases as we have already been doing so far. Create a new test file named **test_view_password_change.py**.

I will list below new types of tests. You can check all the tests I wrote for the password change view clicking in the _view complete file contents_ link next to the code snippet. Most of the tests are similar to what we have been doing so far. I moved to an external file to avoid being too repetitive.

**accounts/tests/test_view_password_change.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/03e2f20a4c1e693c9b22b343503fb461#file-test_view_password_change-py-L40)</small>

<figure class="highlight">

    class LoginRequiredPasswordChangeTests(TestCase):
        def test_redirection(self):
            url = reverse('password_change')
            login_url = reverse('login')
            response = self.client.get(url)
            self.assertRedirects(response, f'{login_url}?next={url}')

</figure>

The test above tries to access the **password_change** view without being logged in. The expected behavior is to redirect the user to the login page.

**accounts/tests/test_view_password_change.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/03e2f20a4c1e693c9b22b343503fb461#file-test_view_password_change-py-L48)</small>

<figure class="highlight">

    class PasswordChangeTestCase(TestCase):
        def setUp(self, data={}):
            self.user = User.objects.create_user(username='john', email='john@doe.com', password='old_password')
            self.url = reverse('password_change')
            self.client.login(username='john', password='old_password')
            self.response = self.client.post(self.url, data)

</figure>

Here we defined a new class named **PasswordChangeTestCase**. It does a basic setup, creating a user and making a **POST** request to the **password_change** view. In the next set of test cases, we are going to use this class instead of the **TestCase** class and test a successful request and an invalid request:

**accounts/tests/test_view_password_change.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/03e2f20a4c1e693c9b22b343503fb461#file-test_view_password_change-py-L60)</small>

<figure class="highlight">

    class SuccessfulPasswordChangeTests(PasswordChangeTestCase):
        def setUp(self):
            super().setUp({
                'old_password': 'old_password',
                'new_password1': 'new_password',
                'new_password2': 'new_password',
            })

        def test_redirection(self):
            '''
            A valid form submission should redirect the user
            '''
            self.assertRedirects(self.response, reverse('password_change_done'))

        def test_password_changed(self):
            '''
            refresh the user instance from database to get the new password
            hash updated by the change password view.
            '''
            self.user.refresh_from_db()
            self.assertTrue(self.user.check_password('new_password'))

        def test_user_authentication(self):
            '''
            Create a new request to an arbitrary page.
            The resulting response should now have an `user` to its context, after a successful sign up.
            '''
            response = self.client.get(reverse('home'))
            user = response.context.get('user')
            self.assertTrue(user.is_authenticated)

    class InvalidPasswordChangeTests(PasswordChangeTestCase):
        def test_status_code(self):
            '''
            An invalid form submission should return to the same page
            '''
            self.assertEquals(self.response.status_code, 200)

        def test_form_errors(self):
            form = self.response.context.get('form')
            self.assertTrue(form.errors)

        def test_didnt_change_password(self):
            '''
            refresh the user instance from the database to make
            sure we have the latest data.
            '''
            self.user.refresh_from_db()
            self.assertTrue(self.user.check_password('old_password'))

</figure>

The `refresh_from_db()` method make sure we have the latest state of the data. It forces Django to query the database again to update the data. We have to do it because the **change_password** view update the password in the database. So to test if the password _really_ changed, we have to grab the latest data from the database.

* * *