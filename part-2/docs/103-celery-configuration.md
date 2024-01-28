# Celery Configuration

### Setup

In the "core" director, create a *celery.py* file and add the following code:

```py
import os

from celery import Celery


os.environ.setdefault("DJANGO_SETTINGS_MODULE", "core.settings")

app = Celery("core")
app.config_from_object("django.conf:settings", namespace="CELERY")
app.autodiscover_tasks()
```


----------

####  What's happening here?

1. First, we set a default value for the `DJANGO_SETTINGS_MODULE` environment variable so that the Celery will know how to find the Django project.
2. Next, we created a new Celery instance, with the name `core`, and assigned the value to a variable called `app`.
3. We then loaded the celery configuration values from the settings object from `django.conf`. We used `namespace="CELERY"` to prevent clashes with other Django settings. All config settings for Celery must be prefixed with `CELERY_`, in other words.
4. Finally, `app.autodiscover_tasks()` tells Celery to look for Celery tasks from applications defined in `settings.INSTALLED_APPS`.

Add the following code to *`core/__init__.py`*:

```py
from .celery import app as celery_app

__all__ = ("celery_app",)
```

Lastly, update the *core/settings.py* file with the following Celery settings so that it can connect to Redis:

```py
CELERY_BROKER_URL = "redis://redis:6379"
CELERY_RESULT_BACKEND = "redis://redis:6379"
```

Build the new containers to ensure that everything works:

```bash
Build the new containers to ensure that everything works:
```

Take a look at the logs for each service to see that they are ready, without errors:

```bash
$ docker-compose logs 'web'
$ docker-compose logs 'celery'
$ docker-compose logs 'celery-beat'
$ docker-compose logs 'redis'
```

If all went well, we now have four containers, each with different services.

Now we're ready to create a sample task to see that it works as it should.

### Create a Task

Create a new file called *core/tasks.py* and add the following code for a sample task that just logs to the console:

```py
from celery import shared_task
from celery.utils.log import get_task_logger


logger = get_task_logger(__name__)


@shared_task
def sample_task():
    logger.info("The sample task just ran.")
```

### Schedule the Task

At the end of your *settings.py* file, add the following code to schedule `sample_task` to run once per minute, using Celery Beat:

```py
CELERY_BEAT_SCHEDULE = {
    "sample_task": {
        "task": "core.tasks.sample_task",
        "schedule": crontab(minute="*/1"),
    },
}
```

Here, we defined a periodic task using the [CELERY_BEAT_SCHEDULE](https://docs.celeryq.dev/en/latest/userguide/configuration.html#std:setting-beat_schedule) setting. We gave the task a name, `sample_task`, and then declared two settings:

1. `task` declares which task to run.
1. `schedule` sets the interval on which the task should run. This can be an integer, a timedelta, or a crontab. We used a crontab pattern for our task to tell it to run once every minute. You can find more info on Celery's scheduling [here](https://docs.celeryq.dev/en/stable/reference/celery.schedules.html).

Make sure to add the imports:

```python
from celery.schedules import crontab

import core.tasks
```

Restart the container to pull in the new settings:

```bash
$ docker-compose up -d --build
```

Once done, take a look at the celery logs in the container:

```bash
$ docker-compose logs -f 'celery'
```

You should see something similar to:

```bash
celery_1  |  -------------- [queues]
celery_1  |                 .> celery           exchange=celery(direct) key=celery
celery_1  |
celery_1  |
celery_1  | [tasks]
celery_1  |   . core.tasks.sample_task
```

We can see that Celery picked up our samples task, `core.tasks.sample_task`.

Every minute you see a row in the log ends with "The sample task just ran".

```bash
celery_1  | [2021-07-01 03:06:00,003: INFO/MainProcess]
              Task core.tasks.sample_task[b8041b6c-bf9b-47ce-ab00-c37c1e837bc7] received
celery_1  | [2021-07-01 03:06:00,004: INFO/ForkPoolWorker-8]
              core.tasks.sample_task[b8041b6c-bf9b-47ce-ab00-c37c1e837bc7]:
              The sample task just ran.
```

