#### Version Control

Version control is an extremely important topic in software development. Especially when working with teams and maintaining production code at the same time, several features are being developed in parallel. No matter if it’s a one developer project or a multiple developers project, every project should use version control.

There are several options of version control systems out there. Perhaps because of the popularity of GitHub, **Git** become the _de facto_ standard in version control. So if you are not familiar version control, Git is a good place to start. There are many tutorials, courses, and resources in general so that it’s easy to find help.

GitHub and Code School have a great [interactive tutorial about Git](https://try.github.io), which I used years ago when I started moving from SVN to Git. It’s a very good introduction.

This is such an important topic that I probably should have brought it up since the first tutorial. But the truth is I wanted the focus of this tutorial series to be on Django. If all this is new for you, don’t worry. It’s important to take one step at a time. Your first project won’t be perfect. It’s important to keep learning and evolving your skills slowly but with constancy.

A very good thing about Git is that it’s much more than just a version control system. There’s a rich ecosystem of tools and services built around it. Some good examples are continuous integration, deployment, code review, code quality, and project management.

Using Git to support the deployment process of Django projects works very well. It’s a convenient way to pull the latest version from the source code repository or to rollback to a specific version in case of a problem. There are many services that integrate with Git so to automate test execution and deployment for example.

If you don’t have Git installed on your local machine, grab the installed from [https://git-scm.com/downloads](https://git-scm.com/downloads).

##### Basic Setup

First thing, set your identity:

```
git config --global user.name "Vitor Freitas"
git config --global user.email vitor@simpleisbetterthancomplex.com
```

In the project root (the same directory as **manage.py** is), initialize a git repository:

```
git init
```

```
Initialized empty Git repository in /Users/vitorfs/Development/myproject/.git/
```

Check the status of the repository:

```
git status
```

```
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

  accounts/
  boards/
  manage.py
  myproject/
  requirements.txt
  static/
  templates/

nothing added to commit but untracked files present (use "git add" to track)
```

Before we proceed in adding the source files, create a new file named **.gitignore** in the project root. This special file will help us keep the repository clean, without unnecessary files like cache files or logs for example.

You can grab a [generic .gitignore file for Python projects](https://github.com/github/gitignore/blob/master/Python.gitignore) from GitHub.

Make sure to rename it from **Python.gitignore** to just **.gitignore** (the dot is important!).

You can complement the **.gitignore** file telling it to ignore SQLite database files for example:

**.gitignore**

```
__pycache__/
*.py[cod]
.env
venv/

# SQLite database files

*.sqlite3
```

Now add the files to the repository:

```
git add .
```

Notice the dot here. The command above is telling Git to add _all_ untracked files within the current directory.

Now make the first commit:

```
git commit -m "Initial commit"
```

Always write a comment telling what this commit is about, briefly describing what have you changed.

##### Remote Repository

Now let’s setup [GitHub](https://github.com) as a remote repository. First, create a free account on GitHub, then confirm your email address. After that, you will be able to create public repositories.

For now, just pick a name for the repository, don’t initialize it with a README, or add a .gitignore or add a license so far. Make sure you start the repository empty:

![GitHub](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/github1.png)

After you create the repository you should see something like this:

![GitHub](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/github2.png)

Now let’s configure it as our remote repository:

```
git remote add origin git@github.com:sibtc/django-boards.git
```

Now push the code to the remote server, that is, to the GitHub repository:

```
git push origin master

Counting objects: 84, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (81/81), done.
Writing objects: 100% (84/84), 319.70 KiB | 0 bytes/s, done.
Total 84 (delta 10), reused 0 (delta 0)
remote: Resolving deltas: 100% (10/10), done.
To git@github.com:sibtc/django-boards.git
 * [new branch]      master -> master
```

![GitHub](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/github3.png)

I create this repository just to demonstrate the process to create a remote repository with an existing code base. The source code of the project is officially hosted in this repository: [https://github.com/sibtc/django-beginners-guide](https://github.com/sibtc/django-beginners-guide).

* * *