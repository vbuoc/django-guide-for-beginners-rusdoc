# Hello, World!

Давайте напишем наше первое представление. Мы разберемся с ними в деталях дальше в руководстве, но пока давайте просто поэксперементируем с тем, каково это создавать новые страницы в Django.

Откройте `views.py`, находящийся в приложении `boards` и добавьте следующий код:

**views.py**
```python
from django.http import HttpResponse

def home(request):
    return HttpResponse('Hello, World!')
```

Представлений - функции Python, которые получают объект `HttpRequest` и возвращают объект `HttpResponse`. Получают запрос, как параметр и возвращают ответ, как результат. Этот порядок вам необходимо держать в голове.

Итак, мы создали простое представление, названное `home`, которое просто возвращает сообщение, говорящее "Hello, World!".

Теперь мы должны сказать Django, когда использовать данное представление. Это указывается в файле `urls.py`:
Now we have to tell Django when to serve this view. It’s done inside the urls.py file:

**urls.py**
```python
from django.conf.urls import url
from django.contrib import admin

from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^admin/', admin.site.urls),
]
```

Если вы сравните пример кода выше и содержимое вашего файла `urls.py`, вы заметите, что добавлена строка `url(r'^$', views.home, name='home')` и импортирован модуль представлений нашего приложения `boards`, используя `from boards import views`.

Как было сказано выше, мы изучим все это чуть позже.

Но для начала, Django работает с регулярными выражениями, чтобы находить запрашиваемый URL. Для нашего представления `home` мы используем выражение `^$`, которое соответствует пустому пути. Это означает, что путь соответствует `http://127.0.0.1:8000`. Если мы хотим, чтобы наше представление работало по адресу, на пример, `http://127.0.0.1:8000/homepage/`, URL должен был бы быть `url(r'^homepage/$', views.home, name='home')`.

Посмотрите, что получилось:

```bash
python manage.py runserver
```

В браузере откройте `http://127.0.0.1:8000`:

![Hello, World!](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-1/hello-world.png)

Вы только что создали свое первое представление.

> [Начиная работу](/part-1/getting-started.md)
