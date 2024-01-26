# Celery Logs

Update the `celery` service, in *docker-compose.yml*, so that Celery logs are dumpued to a log file:

```
celery:
  build: ./project
  command: celery --app=core worker --loglevel=info --logfile=logs/celery.log
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
```

Add a new directory to "project" called "logs". Then, add a new file called *celery.log* to that newly created directory.

Update:

```bash
$ docker-compose up -d --build
```

You should see the log file fill up locally since we set up a volumn:

```bash
[2024-01-26 11:25:04,261: INFO/MainProcess] Connected to redis://redis:6379/0
[2024-01-26 11:25:04,267: INFO/MainProcess] mingle: searching for neighbors
[2024-01-26 11:25:05,278: INFO/MainProcess] mingle: all alone
[2024-01-26 11:25:05,295: WARNING/MainProcess] /usr/local/lib/python3.11/site-packages/celery/fixups/django.py:203: UserWarning: Using settings.DEBUG leads to a memory
            leak, never use this setting in production environments!
  warnings.warn('''Using settings.DEBUG leads to a memory

[2024-01-26 11:25:05,295: INFO/MainProcess] celery@965213819a79 ready.
[2024-01-26 11:25:26,855: INFO/MainProcess] Task tasks.sample_tasks.create_task[ac3a4c6d-77a2-4d6c-b513-c915c62978ca] received
[2024-01-26 11:25:36,888: INFO/ForkPoolWorker-1] Task tasks.sample_tasks.create_task[ac3a4c6d-77a2-4d6c-b513-c915c62978ca] succeeded in 10.030824464993202s: True
```