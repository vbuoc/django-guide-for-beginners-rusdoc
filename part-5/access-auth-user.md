#### Accessing the Authenticated User

Now we can improve the **new_topic** view and this time set the proper user, instead of just querying the database and picking the first user. That code was temporary because we had no way to authenticate the user. But now we can do better:

**boards/views.py** <small>[(view complete file contents)](https://gist.github.com/vitorfs/483936caca4618dc275545ad2dfbef24#file-views-py-L19)</small>

```

    from django.contrib.auth.decorators import login_required
    from django.shortcuts import get_object_or_404, redirect, render

    from .forms import NewTopicForm
    from .models import Board, Post

    @login_required
    def new_topic(request, pk):
        board = get_object_or_404(Board, pk=pk)
        if request.method == 'POST':
            form = NewTopicForm(request.POST)
            if form.is_valid():
                topic = form.save(commit=False)
                topic.board = board
                topic.starter = request.user  # <- here
                topic.save()
                Post.objects.create(
                    message=form.cleaned_data.get('message'),
                    topic=topic,
                    created_by=request.user  # <- and here
                )
                return redirect('board_topics', pk=board.pk)  # TODO: redirect to the created topic page
        else:
            form = NewTopicForm()
        return render(request, 'new_topic.html', {'board': board, 'form': form})

```

We can do a quick test here by adding a new topic:

![New topic](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-5/topics-2.png)

* * *