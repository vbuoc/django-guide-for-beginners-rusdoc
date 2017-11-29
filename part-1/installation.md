# Установка

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/04/a-complete-beginners-guide-to-django-part-1.html

The first thing we need to do is install some programs on our machine so to be able to start playing with Django. The basic setup consists of installing Python, Virtualenv, and Django.

Basic Setup

Using virtual environments is not mandatory, but it’s highly recommended. If you are just getting started, it’s better to start with the right foot.

When developing Web sites or Web projects with Django, it’s very common to have to install external libraries to support the development. Using virtual environments, each project you develop will have its isolated environment. So the dependencies won’t clash. It also allows you to maintain in your local machine projects that run on different Django versions.

It’s very straightforward to use it, you will see!

Installing Python 3.6.2

The first thing we want to do is install the latest Python distribution, which is Python 3.6.2. At least it was, by the time I was writing this tutorial. If there’s a newer version out there, go with it. The next steps should remain more or less the same.

We are going to use Python 3 because the most important Python libraries have already been ported to Python 3 and also the next major Django version (2.x) won’t support Python 2 anymore. So Python 3 is the way to go.

For this tutorial, I will be using Ubuntu 16.04 as an example. Ubuntu 16.04 already comes with both Python 2 (available as python), and Python 3 (available as python3) installed. We can test the installation by opening the Terminal and checking the versions:

python --version
Python 2.7.12

python3 --version
Python 3.5.2
Test Python Version

So all we have to do is install a newer Python 3 version. But we don’t want to mess with the current Python 3.5.2, as the OS makes use of it. We’re simply going to install Python 3.6.2 under the name python3.6 and let the older version be.

If you are using Ubuntu 16.04 or an older version, first add the following repository:

sudo add-apt-repository ppa:deadsnakes/ppa
If you are using Ubuntu 16.10, 17.04 or 17.10 you don’t need to perform the step above.

Now everyone executes the following commands to install the latest Python 3 distribution:

sudo apt-get update
sudo apt-get install python3.6
The new installation will be available under python3.6, which is fine:

Test Python Version 3.6

Great, Python is up and running. Next step: Virtual Environments!

Installing Virtualenv

For the next step, we are going to use pip, a tool to manage and install Python packages, to install virtualenv.

First let’s install pip for our Python 3.6.2 version:

wget https://bootstrap.pypa.io/get-pip.py
sudo python3.6 get-pip.py
Now we can install virtualenv:

sudo pip3.6 install virtualenv
pip3.6 install virtualenv

So far the installations that we performed was system-wide. From now on, everything we install, including Django itself, will be installed inside a Virtual Environment.

Think of it like this: for each Django project you start, you will first create a Virtual Environment for it. It’s like having a sandbox for each Django project. So you can play around, install packages, uninstall packages without breaking anything.

I like to create a folder named Development on my personal computer. Then, I use it to organize all my projects and websites. But you can follow the next steps creating the directories wherever it feels right for you.

Usually, I start by creating a new folder with the project name inside my Development folder. Since this is going to be our very first project, we don’t need to pick a fancy name or anything. For now, we can call it myproject.

mkdir myproject
cd myproject
Create myproject folder

This folder is the higher level directory that will store all the files and things related to our Django project, including its virtual environment.

So let’s start by creating our very first virtual environment and installing Django.

Inside the myproject folder:

virtualenv venv -p python3.6
Virtualenv

Our virtual environment is created. Now before we start using it, we need to activate:

source venv/bin/activate
You will know it worked if you see (venv) in front of the command line, like this:

Virtualenv activate

Let’s try to understand what happened here. We created a special folder named venv. It contains a copy of Python inside this folder. After we activated the venv environment, when we run the python command, it will use our local copy, stored inside venv, instead of the other one we installed earlier.

Another important thing is that the pip program is already installed as well, and when we use it to install a Python package, like Django, it will be installed inside the venv environment.

Note that when we have the venv activated, we will use the command python (instead of python3.6) to refer to Python 3.6.2, and just pip (instead of pip3.6) to install packages.

By the way, to deactivate the venv run the command below:

deactivate
But let’s keep it activated for the next steps.

Installing Django 1.11.4

It’s very straightforward. Now that we have the venv activated, run the following command to install Django:

pip install django
pip install django

We are all set up now!

End Installation

[Новый проект](/part-1/new-project.md)
