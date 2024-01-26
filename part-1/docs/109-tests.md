# Tests

Let's start with the most basic test:

```py
def test_task():
    assert sample_tasks.create_task.run(1)
    assert sample_tasks.create_task.run(2)
    assert sample_tasks.create_task.run(3)
```

Add the above test case to *project/tasks/test_task.py*, and then add to the following output:

```py
from tasks import sample_tasks  
```

Run that test individually:

```bash
$ docker-compose exec web python -m pytest -k "test_task and not test_home"
```

It should take about one minute to run:

```bash
$ docker compose exec web python -m pytest -k "test_task and not test_home"
================================== test session starts ===================================
platform linux -- Python 3.11.1, pytest-7.2.0, pluggy-1.4.0
django: settings: core.settings (from ini)
rootdir: /usr/src/app, configfile: pytest.ini
plugins: django-4.5.2
collected 2 items / 1 deselected / 1 selected                                            

tests/test_tasks.py .                                                              [100%]

======================= 1 passed, 1 deselected in 60.25s (0:01:00) =======================
```

It's worth nothing that in the above asserts, we used the `.run` method (rather than `.delay`) to run the task directlly without a Celery worker.

Want to mock the `.run` method to speed things up?

```python
@patch("tasks.sample_tasks.create_task.run")
def test_mock_task(mock_run):
    assert sample_tasks.create_task.run(1)
    sample_tasks.create_task.run.assert_called_once_with(1)

    assert sample_tasks.create_task.run(2)
    assert sample_tasks.create_task.run.call_count == 2

    assert sample_tasks.create_task.run(3)
    assert sample_tasks.create_task.run.call_count == 3
```

Import:

```py
from unittest.lock import patch
```

Test:

```bash
$ docker c
ompose exec web python -m pytest -k "test_mock_task"
================================== test session starts ===================================
platform linux -- Python 3.11.1, pytest-7.2.0, pluggy-1.4.0
django: settings: core.settings (from ini)
rootdir: /usr/src/app, configfile: pytest.ini
plugins: django-4.5.2
collected 3 items / 2 deselected / 1 selected                                            

tests/test_tasks.py .                                                              [100%]

============================ 1 passed, 2 deselected in 0.23s =============================
```

Much quicker!

How about a full integration test?

```py
def test_task_status(client):
    response = client.post(reverse("run_task"), {"type": 0})
    content = json.loads(response.content)
    task_id = content["task_id"]
    assert response.status_code == 202
    assert task_id

    response = client.get(reverse("get_status", args=[task_id]))
    content = json.loads(response.content)
    assert content == {"task_id": task_id, "task_status": "PENDING", "task_result": None}
    assert response.status_code == 200

    while content["task_status"] == "PENDING":
        response = client.get(reverse("get_status", args=[task_id]))
        content = json.loads(response.content)
    assert content == {"task_id": task_id, "task_status": "SUCCESS", "task_result": True}
```

Keep in mind that this test uses the same broker and backend used in development. You may want to instantiate a new Celery app for testing:

```python
app = celery.Celery('tests', broker=CELERY_TEST_BROKER, backend=CELERY_TEST_BACKEND)
```

Add the import:

```python
import json
```

