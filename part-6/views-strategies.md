#### Views Strategies

At the end of the day, all Django views are _functions_. Even class-based views (CBV). Behind the scenes, it does all the magic and ends up returning a view function.

Class-based views were introduced to make it easier for developers to reuse and extend views. There are many benefits of using them, such as the extendability, the ability to use O.O. techniques such as multiple inheritances, the handling of HTTP methods are done in separate methods, rather than using conditional branching, and there are also the Generic Class-Based Views (GCBV).

Before we move forward, let’s clarify what those three terms mean:

*   Function-Based Views (FBV)
*   Class-Based Views (CBV)
*   Generic Class-Based Views (GCBV)

A FBV is the simplest representation of a Django view: it’s just a function that receives an **HttpRequest** object and returns an **HttpResponse**.

A CBV is every Django view defined as a Python class that extends the `django.views.generic.View` abstract class. A CBV essentially is a class that wraps a FBV. CBVs are great to extend and reuse code.

GCBVs are built-in CBVs that solve specific problems such as listing views, create, update, and delete views.

Below we are going to explore some examples of the different implementation strategies.

##### Function-Based View

**views.py**

```
def new_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('post_list')
    else:
        form = PostForm()
    return render(request, 'new_post.html', {'form': form})
```

**urls.py**

```
urlpatterns = [
    url(r'^new_post//figure>, views.new_post, name='new_post'),
]
```

##### Class-Based View

A CBV is a view that extends the **View** class. The main difference here is that the requests are handled inside class methods named after the HTTP methods, such as **get**, **post**, **put**, **head**, etc.

So, here we don’t need to do a conditional to check if the request is a **POST** or if it’s a **GET**. The code goes straight to the right method. This logic is handled internally in the **View** class.

**views.py**

```
from django.views.generic import View

class NewPostView(View):
    def post(self, request):
        form = PostForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('post_list')
        return render(request, 'new_post.html', {'form': form})

    def get(self, request):
        form = PostForm()
        return render(request, 'new_post.html', {'form': form})
```

The way we refer to the CBVs in the **urls.py** module is a little bit different:

**urls.py**

```
urlpatterns = [
    url(r'^new_post//figure>, views.NewPostView.as_view(), name='new_post'),
]
```

Here we need to use the `as_view()` class method, which returns a view function to the url patterns. In some cases, we can also feed the `as_view()` with some keyword arguments, so to customize the behavior of the CBV, just like we did with some of the authentication views to customize the templates.

Anyway, the good thing about CBV is that we can add more methods, and perhaps do something like this:

```
from django.views.generic import View

class NewPostView(View):
    def render(self, request):
        return render(request, 'new_post.html', {'form': self.form})

    def post(self, request):
        self.form = PostForm(request.POST)
        if self.form.is_valid():
            self.form.save()
            return redirect('post_list')
        return self.render(request)

    def get(self, request):
        self.form = PostForm()
        return self.render(request)
```

It’s also possible to create some generic views that accomplish some tasks so that we can reuse it across the project.

But that’s pretty much all you need to know about CBVs. Simple as that.

##### Generic Class-Based View

Now about the GCBV. That’s a different story. As I mentioned earlier, those views are built-in CBVs for common use cases. Their implementation makes heavy usage of multiple inheritances (mixins) and other O.O. strategies.

They are very flexible and can save many hours of work. But in the beginning, it can be difficult to work with them.

When I first started working with Django, I found GCBV hard to work with. At first, it’s hard to tell what is going on, because the code flow is not obvious, as there is good chunk of code hidden in the parent classes. The documentation is a little bit challenging to follow too, mostly because the attributes and methods are sometimes spread across eight parent classes. When working with GCBV, it’s always good to have the [ccbv.co.uk](https://ccbv.co.uk/) opened for quick reference. No worries, we are going to explore it together.

Now let’s see a GCBV example.

**views.py**

```
from django.views.generic import CreateView

class NewPostView(CreateView):
    model = Post
    form_class = PostForm
    success_url = reverse_lazy('post_list')
    template_name = 'new_post.html'
```

Here we are using a generic view used to create model objects. It does all the form processing and save the object if the form is valid.

Since it’s a CBV, we refer to it in the **urls.py** the same way as any other CBV:

**urls.py**

```
urlpatterns = [
    url(r'^new_post//figure>, views.NewPostView.as_view(), name='new_post'),
]
```

Other examples of GCBVs are **DetailView**, **DeleteView**, **FormView**, **UpdateView**, **ListView**.

* * *