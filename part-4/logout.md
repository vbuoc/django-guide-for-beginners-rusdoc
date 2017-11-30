#### Logout

To keep a natural flow in the implementation, let’s add the log out view. First, edit the **urls.py** to add a new route:

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
        url(r'^logout//figure>, auth_views.LogoutView.as_view(), name='logout'),
        url(r'^boards/(?P<pk>\d+)//figure>, views.board_topics, name='board_topics'),
        url(r'^boards/(?P<pk>\d+)/new//figure>, views.new_topic, name='new_topic'),
        url(r'^admin/', admin.site.urls),
    ]

```

We imported the **views** from the Django’s contrib module. We renamed it to **auth_views** to avoid clashing with the **boards.views**. Notice that this view is a little bit different: `LogoutView.as_view()`. It’s a Django’s class-based view. So far we have only implemented classes as Python functions. The class-based views provide a more flexible way to extend and reuse views. We will discuss more that subject later on.

Open the **settings.py** file and add the `LOGOUT_REDIRECT_URL` variable to the bottom of the file:

**myproject/settings.py**

```

    LOGOUT_REDIRECT_URL = 'home'

```

Here we are passing the name of the URL pattern we want to redirect the user after the log out.

After that, it’s already done. Just access the URL **127.0.0.1:8000/logout/** and you will be logged out. But hold on a second. Before you log out, let’s create the dropdown menu for logged in users.

* * *