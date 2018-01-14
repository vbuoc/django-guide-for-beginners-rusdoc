# Представления, шаблоны и статические файлы

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html

На данный момент у нас уже есть представление, называемое `home`, отображающее “Hello, World!” в корне нашего приложения

**myproject/urls.py**

```python
from django.conf.urls import url
from django.contrib import admin

from boards import views

urlpatterns = [
    url(r'^$', views.home, name='home'),
    url(r'^admin/', admin.site.urls),
]

```

**boards/views.py**

```python
from django.http import HttpResponse

def home(request):
    return HttpResponse('Hello, World!')

```

Мы можем импользовать это, как нашу отправную точку. Если вы вернетесь и посмотрите, на наши макеты, вы увидите, как главная страница должна выглядеть. Все, что мы хотим, это отобразить список досок в таблице с небольшим количеством дополнительной информации.

Первое, что нам нужно сделать - испортировать модель **Board** и получить список всех существующих досок:

**boards/views.py**

```python
from django.http import HttpResponse
from .models import Board

def home(request):
    boards = Board.objects.all()
    boards_names = list()

    for board in boards:
        boards_names.append(board.name)

    response_html = '<br>'.join(boards_names)

    return HttpResponse(response_html)

```

Результатом будет простая страница:

![Boards Homepage HttpResponse](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-httpresponse.png)

Но давайте остановимся здесь. Мы не собираемся слишком глубоко погружаться в рендеринг HTML. Для этого простого представления все, что нам нужно - список досок. Остальное - работа для **Django Template Engine**.

## Django Template Engine Setup

Создайте новую дирректорию **templates**, рядом с **boards** и **mysite**:

```bash
myproject/
    |-- myproject/
    |    |-- boards/
    |    |-- myproject/
    |    |-- templates/   <-- here!
    |    +-- manage.py
    +-- venv/
```

Теперь создайте внутри данной папки HTML файл **home.html**:

**templates/home.html**

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Boards</title>
</head>

<body>
    <h1>Boards</h1>

    {% for board in boards %} {{ board.name }}
    <br> {% endfor %}

</body>

</html>
```

В примере выше мы смешали чистый HTML и специальные тэги `{% for ... in ... %}` и `{{ variable }}`. Они - часть языка шаблонов Django. Пример выше показывает, как проитерироваться через список объектов, используя `for`. `{{ board.name }}` рендерит имя доски в шаблоне HTML, генерируя динамический HTML документ.

Прежде чем мы используем эту HTML страницу, мы должны указать Django, где ей искать шаблоны приложений.

Откройте **settings.py** внутри **myproject**, найдите переменную `TEMPLATES` и установите `DIRS` в `os.path.join(BASE_DIR, 'templates')`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(BASE_DIR, 'templates')
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

```

По сути, эта строка указывает полный путь к проекту и добавляет в конец "/templates".

Мы можем проверить это через Python shell:

```bash
$ python manage.py shell

```

```python

>>> from django.conf import settings

>>> settings.BASE_DIR
'/Users/vitorfs/Development/myproject'

>>> import os

>>> os.path.join(settings.BASE_DIR, 'templates')
'/Users/vitorfs/Development/myproject/templates'

```

Видите? Она указывает на папку **templates**, которую мы указали на предыдущих шагах.

Теперь мы можем обновить наше представление **home**:

**boards/views.py**

```python
from django.shortcuts import render
from .models import Board

def home(request):
    boards = Board.objects.all()
    return render(request, 'home.html', {'boards': boards})

```

Итоговый HTML:

![Boards Homepage render](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-render.png)

Мы можем улучшить HTML, используя таблицу:

**templates/home.html**

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Boards</title>
</head>

<body>
    <h1>Boards</h1>

    <table border="1">
        <thead>
            <tr>
                <th>Board</th>
                <th>Posts</th>
                <th>Topics</th>
                <th>Last Post</th>
            </tr>
        </thead>
        <tbody>
            {% for board in boards %}
            <tr>
                <td>
                    {{ board.name }}
                    <br>
                    <small style="color: #888">{{ board.description }}</small>
                </td>
                <td>0</td>
                <td>0</td>
                <td></td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>

</html>
```

![Boards Homepage render](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-render-2.png)

## Тестирование главной страницы

![Testing Comic](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Testing.png)

Это будет повторяющейся темой, и мы собираемся совместно изучить различные концепции и стратегии на протяжении всей серии обучающих программ.

Давайте напишем наш первый тест. Сейчас мы будем работать в **tests.py** внутри приложения **boards**:

**boards/tests.py**

```python
from django.core.urlresolvers import reverse
from django.test import TestCase

class HomeTests(TestCase):
    def test_home_view_status_code(self):
        url = reverse('home')
        response = self.client.get(url)
        self.assertEquals(response.status_code, 200)
```

Это крайне простой тест, но очень полезный. Мы тестируем _код статуса_ ответа на знапрос. Статус 200 означает **OK**.

Мы можем проверить код статуса ответа в консоли:

![Response 200](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/test-homepage-view-status-code-200.png)

Если будет поймано исключение, ошибка синтаксиса или что-либо еще, Django вернет **500**, что означает **Внутренняя ошибка сервера**. Сейчас, представьте, что приложение имеет 100 представлений. Если мы напишем один простой тест для всех этих представлений, с помощью всего лишь одной команды мы сможем протестировать их. Таким образом, пользователь не увидит ошибок. Без автоматизированных тестов, нам нужно будет тестировать каждую страницу отдельно.

Чтобы запустить тесты:

```bash
$ python manage.py test
```

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.041s

OK
Destroying test database for alias 'default'...
```

Теперь мы можем проверить, что Django вернул правильное представление на запрошенный URL-адрес. Это так же полезный тест, поскольку, продолжая разработку вы увидите, что **urls.py** может быть очень большим и сложным. 

Вот как мы это сделаем:

**boards/tests.py**

```python
from django.core.urlresolvers import reverse
from django.urls import resolve
from django.test import TestCase
from .views import home

class HomeTests(TestCase):
    def test_home_view_status_code(self):
        url = reverse('home')
        response = self.client.get(url)
        self.assertEquals(response.status_code, 200)

    def test_home_url_resolves_home_view(self):
        view = resolve('/')
        self.assertEquals(view.func, home)
```

Во втором тесте мы используем функцию `resolve`. Django использует это для того, чтобы сопоставить URL и представление, представленное в модуле **urls.py**. Этот тест будет проверять, что URL-адрес `/` вызывает представление `home`.

Давайте запустим тесты:

```bash
$ python manage.py test
```

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.027s

OK
Destroying test database for alias 'default'...
```

Чтобы увидеть больше информации, укажите флаг **verbosity** и задайте уровень:

```bash 
$ python manage.py test --verbosity=2
```

```
Creating test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
Operations to perform:
    Synchronize unmigrated apps: messages, staticfiles
    Apply all migrations: admin, auth, boards, contenttypes, sessions
Synchronizing apps without migrations:
    Creating tables...
    Running deferred SQL...
Running migrations:
    Applying contenttypes.0001_initial... OK
    Applying auth.0001_initial... OK
    Applying admin.0001_initial... OK
    Applying admin.0002_logentry_remove_auto_add... OK
    Applying contenttypes.0002_remove_content_type_name... OK
    Applying auth.0002_alter_permission_name_max_length... OK
    Applying auth.0003_alter_user_email_max_length... OK
    Applying auth.0004_alter_user_username_opts... OK
    Applying auth.0005_alter_user_last_login_null... OK
    Applying auth.0006_require_contenttypes_0002... OK
    Applying auth.0007_alter_validators_add_error_messages... OK
    Applying auth.0008_alter_user_username_max_length... OK
    Applying boards.0001_initial... OK
    Applying sessions.0001_initial... OK
System check identified no issues (0 silenced).
test_home_url_resolves_home_view (boards.tests.HomeTests) ... ok
test_home_view_status_code (boards.tests.HomeTests) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.017s

OK
Destroying test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
```

Детализация (Verbosity) определяет количество уведомлений и отладочной информации, которая будет выведена в консоль; 0 - нет вывода, 1 - обычный вывод, 2 - детальный вывод.

## Настройка статических файлов

Статические файлы - CSS, JavaScripts, Шрифты, Изображения и любые другие ресурсы, которые мы можем использовать для построения нашего интерфейса.

По умолчанию, Django не работает с этими файлами. За исключением времени, пока идет разработка. Но Django предоставляет некоторые возможности для управления статическими файлами. Эти возможности сосредоточены в приложении `django.contrib.staticfiles`, которое уже есть в конфигурации `INSTALLED_APPS`.

С таким большим количеством библиотек, нет причин рендерить честый HTML. Мы можем просто добавить BootStrap 4 в наш проект. Это Инструмент с открытым исходным кодом для разработки на HTML, CSS, JavaScript.

В папке приложения, рядом с папками **boards**, **templates** и **myproject**, создайте новую папку с названием **static**, а внутри создайте папку **css**:


```
myproject/
    |-- myproject/
    |    |-- boards/
    |    |-- myproject/
    |    |-- templates/
    |    |-- static/       <-- здесь
    |    |    +-- css/     <-- и здесь
    |    +-- manage.py
    +-- venv/

```

пройдите на [getbootstrap.com](https://getbootstrap.com/docs/4.0/getting-started/download/#compiled-css-and-js) и загрузите последнюю версию:

![Bootstrap Download](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/bootstrap-download.png)

Загрузите версию **Compiled CSS and JS**.

Распакуйте содержимое загруженного архива и скопируйте **css/bootstrap.min.css** в вашу папку **css**:

```
myproject/
    |-- myproject/
    |    |-- boards/
    |    |-- myproject/
    |    |-- templates/
    |    |-- static/
    |    |    +-- css/
    |    |         +-- bootstrap.min.css    <-- сюда
    |    +-- manage.py
    +-- venv/

```

Теперь укажите Django, где искать статические файлы. Откройте **settings.py**, опуститесь в конец файла и после `STATIC_URL` добавьте:

```python
STATIC_URL = '/static/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
```

Так же, как для папки `TEMPLATES`.

Теперь у нас есть возможность подключать статические файлы в наш шаблон:

**templates/home.html**

```html
{% load static %}
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Boards</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
</head>

<body>
    <!-- body suppressed for brevity ... -->
</body>

</html>
```

Сначала мы загружаем тэг статических файлов, используя `{% load static %}` в начале шаблона. 

Тэг `{% static %}` используется для указания, где находится ресурс. В этом случае `{% static 'css/bootstrap.min.css' %}` вернет **/static/css/bootstrap.min.css**, что является относительным путем к нашему ресурсу.

Тэг `{% static %}` использует `STATIC_URL` параметр из **settings.py**, чтобы собрать конечный URL. Например, если вы храните статику на поддомене, например **https://static.example.com/**, вы должны указать `STATIC_URL=https://static.example.com/`, а затем `{% static css/bootstrap.min.css %}`, что вернет **https://static.example.com/css/bootstrap.min.css**.

Если что-то не понятно пока - не волнуйтесь. Просто помните, что необходимо использовать `{% static %}`, когда вы хотите сослаться на CSS, JS или картинку. Позже, когда мы будем разбираться с развертыванием, мы обсудим это подробнее. На данный момент мы закончили с настройкой.

Обновив страницу **127.0.0.1:8000** мы можем увидеть, что это сработало.

![Boards Homepage Bootstrap](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-bootstrap.png)

Теперь мы можем переделать шаблон, чтобы использовать преимущества Bootstrap CSS:

```html
{% load static %}
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Boards</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
</head>

<body>
    <div class="container">
        <ol class="breadcrumb my-4">
            <li class="breadcrumb-item active">Boards</li>
        </ol>
        <table class="table">
            <thead class="thead-inverse">
                <tr>
                    <th>Board</th>
                    <th>Posts</th>
                    <th>Topics</th>
                    <th>Last Post</th>
                </tr>
            </thead>
            <tbody>
                {% for board in boards %}
                <tr>
                    <td>
                        {{ board.name }}
                        <small class="text-muted d-block">{{ board.description }}</small>
                    </td>
                    <td class="align-middle">0</td>
                    <td class="align-middle">0</td>
                    <td></td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    </div>
</body>

</html>

```

Вот что получилось:

![Boards Homepage Bootstrap](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-bootstrap-2.png)

Далее мы реализуем интерфейс администратора, чтобы управлять приложением.
