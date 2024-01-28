# Part 2: Handling Periodic Tasks in Django with Celery and Docker

As you build and scale a Django app you'll inevitably need to run certain tasks periodically and automatically in the background.

Some examples:

- Generating periodic reports
- Clearing cache
- Sending batch e-mail notifications
- Running nightly maintenance jobs

This is one of the few pieces of functionality required for building and scaling a web app that isn't part of the Django core. Fortunately, Celery provides a powerful solution, which is fairly easy to implement called `Celery Beat`.


- [Objectives and Project Setup](docs/101-objective-and-project-setup.md)
- [Celery and Redis](docs/102-celery-and-redis.md)
- [Celery Configuration](docs/103-celery-configuration.md)
- [Customer Django Admin Command](docs/104-custom-django-admin-command.md)
- [text](https://)
- [text](https://)