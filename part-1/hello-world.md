Hello, World!

Let’s write our first view. We will explore it in great detail in the next tutorial. But for now, let’s just experiment how it looks like to create a new page with Django.

Open the views.py file inside the boards app, and add the following code:

views.py

from django.http import HttpResponse

def home(request):
    return HttpResponse('Hello, World!')
Views are Python functions that receive an HttpRequest object and returns an HttpResponse object. Receive a request as a parameter and returns a response as a result. That’s the flow you have to keep in mind!

So, here we defined a simple view called home which simply returns a message saying Hello, World!.

Now we have to tell Django when to serve this view. It’s done inside the urls.py file:

urls.py

from django.conf.urls import url
from django.contrib import admin

from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^admin/', admin.site.urls),
]
If you compare the snippet above with your urls.py file, you will notice I added the following new line: url(r'^$', views.home, name='home') and imported the views module from our app boards using from boards import views.

As I mentioned before, we will explore those concepts in great detail later on.

But for now, Django works with regex to match the requested URL. For our home view, I’m using the ^$ regex, which will match an empty path, which is the homepage (this url: http://127.0.0.1:8000). If I wanted to match the URL http://127.0.0.1:8000/homepage/, my url would be: url(r'^homepage/$', views.home, name='home').

Let’s see what happen:

python manage.py runserver
In a Web browser, open the http://127.0.0.1:8000 URL:

Hello, World!

That’s it! You just created your very first view.
