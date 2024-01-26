# Celery Setup

Start by adding both Celery and Redis to the *project/requirements.txt* file:

```
Django==4.1.4
pytest==7.2.0
pytest-django==4.5.2

celery==5.2.7
redis==4.4.0
```

Celery uses a message `broker` -- `RabbitMQ`, `Redis`, or `AWS Simple Queue Sservice (SQS)` -- to faciliate communication between the Celery worker and the web application. Messages are added to the broker, which are then processes by the worker(s). Once done, the results are added to the backend.

Redis will be used as both the broker and backend. Add both Redis and a Celery `worker` to the *docker-compose.yml* file like os:

```
version: '3.8'

services:
  web:
    build: ./project
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./project:/usr/src/app/
    ports:
      - 1337:8000
    environment:
      - DEBUG=1
      - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
      - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
      - CELERY_BROKER=redis://redis:6379/0
      - CELERY_BACKEND=redis://redis:6379/0
    depends_on:
      - redis

  celery:
    build: ./project
    command: celery --app=core worker --loglevel=info
    volumes:
      - ./project:/usr/src/app
    environment:
      - DEBUG=1
      - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
      - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
      - CELERY_BROKER=redis://redis:6379/0
      - CELERY_BACKEND=redis://redis:6379/0
    depends_on:
      - web
      - redis

  redis:
    image: redis:7-alpine
```

Take not of `celery --app=core worker --loglevel=info`:

1. `celery worker` is used to start a Celery `worker`
2. `--app=core` runs the `core` Celery `Application` (which we'll define shortly)
3. `--loglevel=info` sets the `logging level` to info.`

Within the project's settings module, add the following at the bottom to tell Celery to use Redis as the broker and backend:

```py
CELERY_BROKER_URL = os.environ.get("CELERY_BROKER", "redis://redis:6379/0")
CELERY_RESULT_BACKEND = os.environ.get("CELERY_BROKER", "redis://redis:6379/0")
```

Next, create a new file called *sample_tasks.py* in "project/tasks":

```py
# project/tasks/sample_tasks.py

import time

from celery import shared_task


@shared_task
def create_task(task_type):
    time.sleep(int(task_type) * 10)
    return True
```

Here, using the `shared_task` decorator, we defined a new Celery task function called `create_task`.

Now, add a *celery.py* file to "project/core".

```py
import os

from celery import Celery


os.environ.setdefault("DJANGO_SETTINGS_MODULE", "core.settings")
app = Celery("core")
app.config_from_object("django.conf:settings", namespace="CELERY")
app.autodiscover_tasks()
```

What's happening here?

1. First, we set a default value for the `DJANGO_SETTINGS_MODULE` environment variable so that Celery will known how to find the Django project.
2. Next, we created a new Celery instance, with the name `core`, and assigned the value to a variable called `app`.
3. We  then loaded the celery configuration values from the settings objects from `django.conf`. We used `namespace="CELERY"` to prevent clashes with other Django settings. All config settings for Celery must be prefixed with `CELERY_`, in other words.
4. Finally, `app.autodiscover_tasks()` tell Celery to look for Celery tasks from applications defined in `settings.INSTALLED_APPS`.

Update *`project/core/__init__.py`* so that the Celery app is automatically import from Django starts:

```python
from .celery import app as celery_app

__all__ = ("celery_app", )
```