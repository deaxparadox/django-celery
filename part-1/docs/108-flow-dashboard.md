# Flower Dashboard

`Flower` is a lightweight, real-time, web-based monitoring tool for Celery. You can monitor currently running  tasks, increase or decrease the worker pool, view graphs and a number of statistics, to name a few.

Add it to *requirements.txt*

```
Django==4.1.4
pytest==7.2.0
pytest-django==4.5.2

celery==5.2.7
redis==4.4.0
flower==1.2.0
```

Then, add a new service to *docker-compose.yml*:

```
dashboard:
  build: ./project
  command: celery flower -A core --port=5555 --broker=redis://redis:6379/0
  ports:
    - 5555:5555
  environment:
    - DEBUG=1
    - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
    - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
    - CELERY_BROKER=redis://redis:6379/0
    - CELERY_BACKEND=redis://redis:6379/0
  depends_on:
    - web
    - redis
    - celery
```

Test it out:

```bash
$ docker compose up --build
```

Navigate to [http://localhost:5555](http://localhost:5555) to view the dashboard. You should see one worker ready to go:


Try adding a few more workers to see how that affects things:


```bash
$ docker-compose up -d --build --scale celery=3
```