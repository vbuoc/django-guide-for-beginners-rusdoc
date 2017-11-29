# Авторизация > Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/25/a-complete-beginners-guide-to-django-part-4.html

#### Introduction

This tutorial is going to be all about Django’s authentication system. We are going to implement the whole thing: registration, login, logout, password reset, and password change.

You are also going to get a brief introduction on how to protect some views from non-authorized users and how to access the information of the logged in user.

In the next section, you will find a few wireframes of authentication-related pages that we are going to implement in this tutorial. After that, you will find an initial setup of a new Django app. So far we have been working on an app named **boards**. But all the authentication related stuff can live in a different app, so to achieve a better organization of the code.

![Accounts App](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Accounts_App.png)

* * *

#### Conclusions

Authentication is a very common use case for most Django applications. In this tutorial, we implemented all the important views: sign up, log in, log out, password reset, and change password. Now that we have a way to create users and authenticate them, we will be able to proceed with the development of the other views of our application.

We still have to improve lots of things regarding the code design: the templates folder is starting to get messy with too many files. The **boards** app tests are still disorganized. Also, we have to start refactoring the **new topic** view, because now we can retrieve the logged in user. We will get to that part soon.

I hope you enjoyed the forth part of this tutorial series! The fifth part is coming out next week, on Oct 2, 2017. If you would like to get notified when the fifth part is out, you can [subscribe to our mailing list](http://eepurl.com/b0gR51).

The source code of the project is available on GitHub. The current state of the project can be found under the release tag **v0.4-lw**. The link below will take you to the right place:

[https://github.com/sibtc/django-beginners-guide/tree/v0.4-lw](https://github.com/sibtc/django-beginners-guide/tree/v0.4-lw)

* * *