# Авторизация

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/25/a-complete-beginners-guide-to-django-part-4.html

<h4 id="introduction">Introduction</h4>

<p>This tutorial is going to be all about Django’s authentication system. We are going to implement the whole thing:
registration, login, logout, password reset, and password change.</p>

<p>You are also going to get a brief introduction on how to protect some views from non-authorized users and how to
access the information of the logged in user.</p>

<p>In the next section, you will find a few wireframes of authentication-related pages that we are going to implement
in this tutorial. After that, you will find an initial setup of a new Django app. So far we have been working on
an app named <strong>boards</strong>. But all the authentication related stuff can live in a different app, so to achieve a better
organization of the code.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-4/Pixton_Comic_Accounts_App.png" alt="Accounts App" /></p>

<hr />

<h4 id="conclusions">Conclusions</h4>

<p>Authentication is a very common use case for most Django applications. In this tutorial, we implemented all the
important views: sign up, log in, log out, password reset, and change password. Now that we have a way to create users
and authenticate them, we will be able to proceed with the development of the other views of our application.</p>

<p>We still have to improve lots of things regarding the code design: the templates folder is starting to get messy with
too many files. The <strong>boards</strong> app tests are still disorganized. Also, we have to start refactoring the <strong>new topic</strong>
view, because now we can retrieve the logged in user. We will get to that part soon.</p>

<p>I hope you enjoyed the forth part of this tutorial series! The fifth part is coming out next week, on Oct 2, 2017.
If you would like to get notified when the fifth part is out, you can <a href="http://eepurl.com/b0gR51" target="_blank">subscribe to our mailing list</a>.</p>

<p>The source code of the project is available on GitHub. The current state of the project can be found under the release
tag <strong>v0.4-lw</strong>. The link below will take you to the right place:</p>

<p><a href="https://github.com/sibtc/django-beginners-guide/tree/v0.4-lw" target="_blank">https://github.com/sibtc/django-beginners-guide/tree/v0.4-lw</a></p>

<hr />
