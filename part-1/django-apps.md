# Приложения Django

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/04/a-complete-beginners-guide-to-django-part-1.html

В философии Django мы имеет две важные концепции:

* `app` - приложение, которое что-то делает. Приложение обычно состоит из моделей(таблиц БД), представлений, шаблонов, тестов.
* `project` - набор настрое и приложений. Один проект может состоять из нескольких приложений или даже из одного.

Важно помнить, что вы не можете запустить Django приложение без проекта. Простые приложения могут быть написаны внутри одного приложение, которое может быть названо **blog** или **weblog**, например.

![Django Apps](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-1/Pixton_Comic_Django_Apps.png)

Это способ организовать исходный код. В начале, определить, что является приложением, а что нет - не самая простая задача. Как организовать код и т.д. Но не вонуйтесь об этом. Давайте лучше освоимся в API Django и основами.

Давайте создадим простой Web-форум или "Доски обсуждения". Для создания нашего первого приложения пройдите в папку, содержащую `manage.py` и выполните следующую команду:

```bash
django-admin startapp boards
```

Обратите внимание, что мы использовали команду `startapp`.

Эта команда стоздаст нам следующую структуру:

```
myproject/
 |-- myproject/
 |    |-- boards/                <-- our new django app!
 |    |    |-- migrations/
 |    |    |    +-- __init__.py
 |    |    |-- __init__.py
 |    |    |-- admin.py
 |    |    |-- apps.py
 |    |    |-- models.py
 |    |    |-- tests.py
 |    |    +-- views.py
 |    |-- myproject/
 |    |    |-- __init__.py
 |    |    |-- settings.py
 |    |    |-- urls.py
 |    |    |-- wsgi.py
 |    +-- manage.py
 +-- venv/
```

Давайте разберемся, что делает каждый файл:

* `migrations /`: здесь Django хранит некоторые файлы, чтобы отслеживать изменения, которые вы создаете в файле `models.py`, чтобы поддерживать синхронизацию базы данных и `models.py`.
* `admin.py`: это файл конфигурации для встроенного приложения Django под названием **Django Admin**.
* `apps.py`: это файл конфигурации самого приложения.
* `models.py`: здесь мы определяем сущности нашего веб-приложения. Модели автоматически переводятся Django в таблицы базы данных.
* `tests.py`: этот файл используется для написания модульных тестов приложения.
* `views.py`: это файл, в котором мы обрабатываем цикл запросов / ответов нашего веб-приложения.

Теперь, когда мы создали наше приложение, давайте настроим проект, чтобы иметь возможность его использовать.

Для этого откройте `settings.py` и найдите переменную `INSTALLED_APPS`: 

**settings.py**
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Как вы можете видеть, Django уже идет с 6 установленными приложениями. Они реализуют общий функционал, требуемый большинством Web-приложений, такой как авториазция, сессии, статика и т.д.

Мы будем изучать эти приложения по мере продвижения в этом руководстве. Но пока, просто добавьте наше приложение к списку INSTALLED_APPS:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'boards',
]
```

По аналогии с квадратами и кругами из предыдущей иллюстрации, желтый круг - наше приложение `boards`, а `django.contrib.admin` или `django.contrib.auth` - красные круги.

> ["Hello World"](/part-1/hello-world.md)
