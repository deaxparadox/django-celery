# Trigger a Task

Update the view to kick off the task and respond with the id:

```py
# project/tasks/views.py

@csrf_exempt
def run_task(request):
    if request.POST:
        task_type = request.POST.get("type")
        task = create_task.delay(int(task_type))
        return JsonResponse({"task_id": task.id}, status=202)
```

Don't forget to import that task:

```py
from tasks.sample_tasks import create_task
```

Build the images and spin up the new containers:

```bash
$ docker compose up -d --build
```

To trigger a new task, run:

```bash
$ curl -F type=0 http://localhost:1337/tasks/
```

You should see something like:

```
{
  "task_id": "6f025ed9-09be-4cbb-be10-1dce919797de"
}
```