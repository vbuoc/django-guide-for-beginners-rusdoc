# Интерфейс администратора Django

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html

Когда мы начинаем новый проект, Django уже обладает **Интерфейсом администратора Django**, который можно увидеть в `INSTALLED_APPS` в `settings.py`.

![Django Admin Comic](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Django_Admin.png)

Его можно использовать, например, в блоге: через него авторы могут писать и публиковать статьи. Или в интернет-магазине персонал может создавать, изменять или удалять товары.

Сейчас мы собираемся настроить **Интерфейс администратора Django** для управления досками в приложении.

Давайте начнем с того, что создадим аккаунт администратора:

```bash
python manage.py createsuperuser
```

Далее следовать инструкциям:

```bash
Username (leave blank to use 'vitorfs'): admin
Email address: admin@example.com
Password:
Password (again):
Superuser created successfully.
```

Теперь можно открыть браузер по адресу `http://127.0.0.1:8000/admin/`

![Django Admin Login](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-login.png)

Введите имя **пользователя** и **пароль** для того, чтобы войти в **Интерфейс администратора Django**:

![Django Admin](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin.png)

Он уже обладает некоторыми возможностиями. Вы уже можете добавлять **пользователей(Users)** и **группы(Groups)** для управления разрешениями. Мы рассмотрим данную возможность позже.

Для того чтобы добавить **Доску(Board)** необходимо открыть файл `admin.py` в приложении `boards` и изменить его следующим образом:

**boards/admin.py**
```python
from django.contrib import admin
from .models import Board

admin.site.register(Board)
```

Сохраните `admin.py` и обновите страницу браузера:

![Django Admin Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards.png)

И все! Ее(модель) уже можно использовать. Нажмите на ссылку `Boards`, чтобы увидеть список уже существующих досок:

![Django Admin Boards List](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-list.png)

Мы можем добавить доску нажав на `Add Board`:

![Django Admin Boards Add](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-add.png)

Нажмите `save`:

![Django Admin Boards List](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/django-admin-boards-list-2.png)

Мы можем проверить, что все нормально, открыв `http://127.0.0.1:8000`:

![Boards Homepage](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/boards-homepage-bootstrap-3.png)
