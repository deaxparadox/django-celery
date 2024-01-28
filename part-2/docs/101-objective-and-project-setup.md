# Ojbectives

By the end of this tutorial, you should be able to:

1. Containerize Django, Celery, and Redis with Docker
2. Integrate Celery into a Django app and create tasks
3. Write a custom Django Admin command
4. Schedule a custom Django Admin command to run periodically via Celery Beat

## Project Setup

Clone down the base project from the django-celery-beat repo, and then check out the base branch:

```bash
$ git clone https://github.com/testdrivenio/django-celery-beat --branch base --single-branch
$ cd django-celery-beat
```

Since we'll need to manage four processes in total (Django, Redis, worker, and scheduler), we'll use Docker to simplify our workflow by wiring them up so that they can all be run from one terminal window with a single command.

Fom the project root, create the images and spin up the Docker containers:

```bash
$ docker-compose up -d --build
```

Next, apply the migrations

```bash
$ docker-compose exec web python manage.py migrate
```

Once the build is complete, navigate to [http://localhost:1337](http://localhost:1337) to ensure the app works as expected. You should see the following text:

```
Orders

No orders found!
```