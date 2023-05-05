---
title: Getting Started
layout: framework_docs
order: 1
redirect_from: /docs/getting-started/django/
subnav_glob: docs/django/getting-started/*.html.*
objective: Build and deploy a very basic Django app on Fly. This guide is the fastest way to try using Fly, so if you're short on time start here.
related_pages:
  - /docs/django/getting-started/existing
  - /docs/flyctl/
  - /docs/reference/configuration/
  - /docs/postgres/
---

In this guide we build and deploy a [simple Django website](https://github.com/fly-apps/hello-django) to demonstrate how quickly Django apps can be deployed to Fly.io!

## Initial Set Up

Make sure that [Python](https://www.python.org/) is already installed on your computer along with a way to create virtual environments.

> We recommend the latest [supported versions](https://devguide.python.org/versions/#supported-versions) of Python.

Go ahead, create and enter your project's folder, here called `hello-django`:

```cmd
mkdir hello-django
```
```cmd
cd hello-django
```
### Virtual Environment

For this guide, we use [venv](https://docs.python.org/3/library/venv.html#module-venv) but any of the other popular choices such as [Poetry](https://python-poetry.org/), [Pipenv](https://github.com/pypa/pipenv), or [pyenv](https://github.com/pyenv/pyenv) work too.

```shell
# Unix/macOS
$ python3 -m venv .venv
$ source .venv/bin/activate
(.venv) $
```
```shell
# Windows
$ python -m venv .venv
$ .venv\Scripts\activate
(.venv) $
```

> From this point on, the commands won't be displayed with (.venv) $ but we assume you have your Python virtual environment activated.

### Create a Django Project

With your virtual environment **activated**, install the [latest](https://www.djangoproject.com/download/#supported-versions) version of Django using [pip](https://pip.pypa.io/en/stable/).
```cmd
python -m pip install Django
```

Create a new Django project, here called `hello_django`:

```cmd
django-admin startproject hello_django .
```

> Don't forget the `.` in the end. It's crutial because it tells the script to install Django in the current directory, our folder `hello-django`.

Now create a new app called `hello`.

```cmd
python manage.py startapp hello
```

Add the new `hello` app to the `INSTALLED_APPS` configuration in the [`hello_django/settings.py`](https://github.com/fly-apps/hello-django/blob/main/hello_django/settings.py) file.

```python
# hello_django/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'hello',  # updated
]
```

### Create an App

Now let's configure a basic view that returns the text, `Hello, Fly!` by updating the [`hello/views.py`](https://github.com/fly-apps/hello-django/blob/main/hello/views.py) file.

```python
# hello/views.py
from django.http import HttpResponse


def hello(request):
    return HttpResponse('Hello, Fly!')
```

Create a new file called [`hello/urls.py`](https://github.com/fly-apps/hello-django/blob/main/hello/urls.py) for our app-level URL configuration.

```python
# hello/urls.py
from django.urls import path

from .views import hello

urlpatterns = [
    path('', hello, name='hello'),
]
```

And update the existing [`hello_django/urls.py`](https://github.com/fly-apps/hello-django/blob/main/hello_django/urls.py) file as well for project-level URL configuration.

```python
# hello_django/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('hello.urls'))
]
```

That's it! Run the `migrate` command to initialize the local database.

```cmd
python manage.py migrate
```

Now `runserver` to start up Django's local web server.

```cmd
python manage.py runserver
```

If you open `http://127.0.0.1:8000/` in your web browser it now displays the text "Hello, Fly!"

## Django Deployment Checklist

By default, Django is configured for local development. The [How to Deploy Django](https://docs.djangoproject.com/en/stable/howto/deployment/) and [Django Deployment Checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/) guide list the various steps required for a secure deployment. 

However, for demonstration purposes, we can take some shortcuts.

First, in the [`hello_django/settings.py`](https://github.com/fly-apps/hello-django/blob/main/hello_django/settings.py) file update the [`ALLOWED_HOSTS`](https://github.com/fly-apps/hello-django/blob/main/hello_django/settings.py#L29) configuration to accept all hosts.

```python
# hello_django/settings.py
ALLOWED_HOSTS = ["*"]  # new
```

Second, install [Gunicorn](https://gunicorn.org/) as our production server.

```cmd
python -m pip install gunicorn
```

Third, create a [`requirements.txt`](https://github.com/fly-apps/hello-django/blob/main/requirements.txt) file listing all the packages in the current Python virtual environment.

```cmd
pip freeze > requirements.txt
```

That's it! We're ready to deploy on Fly.

## flyctl

Fly.io has its own command-line utility for managing apps, [flyctl](https://fly.io/docs/hands-on/install-flyctl/). If not already installed, follow the instructions on the [installation guide](https://fly.io/docs/hands-on/install-flyctl/) and [log in to Fly](https://fly.io/docs/getting-started/log-in-to-fly/).


## Provision Django

To configure and launch the app, run the `fly launch` command and follow the wizard. You can set a name for the app and choose a default region. You can also choose to launch and attach a Postgresql database and/or a Redis database though we are not using either in this example.

```cmd
fly launch
```
```output
Creating app in ~/hello-django
Scanning source code
Detected a Django app
? Choose an app name (leave blank to generate one): hello-django
automatically selected personal organization: Fly.io
? Choose a region for deployment: Ashburn, Virginia (US) (iad)
Created app hello-django in organization personal
Set secrets on hello-django: SECRET_KEY
Wrote config file fly.toml
? Would you like to set up a Postgresql database now? No
? Would you like to set up an Upstash Redis database now? No
Your app is ready! Deploy with `flyctl deploy`
```

This creates two new files in the project that are automatically configured: a [`Dockerfile`](https://github.com/fly-apps/hello-django/blob/main/Dockerfile) and [`fly.toml`](https://github.com/fly-apps/hello-django/blob/main/fly.toml) file to configure applications for deployment.

## Deploy Your Application

To deploy the application use the following command:

```cmd
fly deploy
```

This will take a few seconds as it uploads your application, verifies the app configuration, builds the image, and then monitors to ensure it starts successfully. Once complete visit your app with the following command:

```cmd
fly open
```

You are up and running! Wasn't that easy?

## Recap

We started with an empty directory and in a matter of minutes had a running Django application deployed to the web. A few things to note:

  * Your application is running on a Virtual Machine that was created based on the `Dockerfile` image.
  * The `fly.toml` file controls your app configuration and can be modified as needed.
  * `fly dashboard` can be used to monitor and adjust your application. Pretty much anything you can do from the browser window you can also do from the command line using `fly` commands. Try `fly help` to see what you can do.

Now that you have seen how to deploy a simple Django application, it is time to move on to [Existing Django Apps](/docs/django/getting-started/existing/) that feature static files and a PostgreSQL database.
