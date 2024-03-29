# Celery and Redis

Now, we need to add containers for Celery, Celery Beat, and Redis.

We'll begin by adding the dependencies to the *requirements.txt* file:

```
Django==3.2.4
celery==5.1.2
redis==3.5.3
```

Next, add the following to the end of the *docker-compose.yml* file:

```
redis:
  image: redis:alpine
celery:
  build: ./project
  command: celery -A core worker -l info
  volumes:
    - ./project/:/usr/src/app/
  environment:
    - DEBUG=1
    - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
    - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
  depends_on:
    - redis
celery-beat:
  build: ./project
  command: celery -A core beat -l info
  volumes:
    - ./project/:/usr/src/app/
  environment:
    - DEBUG=1
    - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
    - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
  depends_on:
    - redis
```

The full docker-compose.yml file should now look like this:

```
version: '3.8'

services:
  web:
    build: ./project
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./project/:/usr/src/app/
    ports:
      - 1337:8000
    environment:
      - DEBUG=1
      - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
      - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
    depends_on:
      - redis
  redis:
    image: redis:alpine
  celery:
    build: ./project
    command: celery -A core worker -l info
    volumes:
      - ./project/:/usr/src/app/
    environment:
      - DEBUG=1
      - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
      - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
    depends_on:
      - redis
  celery-beat:
    build: ./project
    command: celery -A core beat -l info
    volumes:
      - ./project/:/usr/src/app/
    environment:
      - DEBUG=1
      - SECRET_KEY=dbaa1_i7%*3r9-=z-+_mz4r-!qeed@(-a_r(g@k8jo8y3r27%m
      - DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
    depends_on:
      - redis
```

Before building the new containers we need to configure Celery in our Django app.