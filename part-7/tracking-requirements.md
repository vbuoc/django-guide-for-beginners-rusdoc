#### Tracking Requirements

It’s a good practice to keep track of the project’s dependencies, so to be easier to install it on another machine.

We can check the currently installed Python libraries by running the command:

```
pip freeze

dj-database-url==0.4.2
Django==1.11.6
django-widget-tweaks==1.4.1
Markdown==2.6.9
python-decouple==3.1
pytz==2017.2
```

Create a file named **requirements.txt** in the project root, and add the dependencies there:

**requirements.txt**

```
dj-database-url==0.4.2
Django==1.11.6
django-widget-tweaks==1.4.1
Markdown==2.6.9
python-decouple==3.1
```

I kept the **pytz==2017.2** out, because it is automatically installed by Django.

You can update your source code repository:

```
git add .
git commit -m "Add requirements.txt file"
git push origin master
```

* * *