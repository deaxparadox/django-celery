# Project Setup


[<<< Objectives](101-objectives.md) | [Project Setup >>>](102-project-setup.md)

----------


Clone down the base project:

```bash
$ git clone https://github.com/testdrivenio/django-celery --branch v1 --single-branch
$ cd django-celery
$ git checkout v1 -b master
```
Since we'll need to manage three process in total (Django, Redis, worker), we'll use Docker to simplify our workflow by wiring them up so that they can all be run from one terminal window with a single command.

From the project root, create the images and spin up the Docker containers:

```bash
$ docker-compose up -d --build
```

Ouce the build is complete, navigate to [http://localhost1337](http://localhost1337)

----------

Make sure the tests pass as well:

```bash
$ docker compose exec web python -m pytest
================================= test session starts =================================
platform linux -- Python 3.11.1, pytest-7.2.0, pluggy-1.3.0
django: settings: core.settings (from ini)
rootdir: /usr/src/app, configfile: pytest.ini
plugins: django-4.5.2
collected 1 item                                                                      

tests/test_tasks.py .                                                           [100%]

================================== 1 passed in 0.67s ==================================
```