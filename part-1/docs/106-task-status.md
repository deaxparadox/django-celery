# Task Status

Turn back to the event handler on the client side:

```javascript
// project/static/main.js

$('.button').on('click', function() {
  $.ajax({
    url: '/tasks/',
    data: { type: $(this).data('type') },
    method: 'POST',
  })
  .done((res) => {
    getStatus(res.task_id);
  })
  .fail((err) => {
    console.log(err);
  });
});
```

When the response comes back from the original AJAX request, we then continue to call `getStatus()` with the task id every second:

```javascript
function getStatus(taskID) {
  $.ajax({
    url: `/tasks/${taskID}/`,
    method: 'GET'
  })
  .done((res) => {
    const html = `
      <tr>
        <td>${res.task_id}</td>
        <td>${res.task_status}</td>
        <td>${res.task_result}</td>
      </tr>`
    $('#tasks').prepend(html);

    const taskStatus = res.task_status;

    if (taskStatus === 'SUCCESS' || taskStatus === 'FAILURE') return false;
    setTimeout(function() {
      getStatus(res.task_id);
    }, 1000);
  })
  .fail((err) => {
    console.log(err)
  });
}
```

If the response is successful, a new row is added to the table on the DOM.

Update the `get_status` view to return the status:

```py
# project/tasks/views.py

@csrf_exempt
def get_status(request, task_id):
    task_result = AsyncResult(task_id)
    result = {
        "task_id": task_id,
        "task_status": task_result.status,
        "task_result": task_result.result
    }
    return JsonResponse(result, status=200)
```

import `AsyncResult`:

```py
from celery.result import AsyncResult
```

Update the containers:

```bash
$ docker compose up -d --build
```

Trigger a new task:

```bash
$ curl -F type=1 http://localhost:1337/tasks/
```

Then, grab the `task_id` from the response and call the updated endpoint to view the status:

```bash
$ curl http://localhost:1337/tasks/25278457-0957-4b0b-b1da-2600525f812f/

{
    "task_id": "25278457-0957-4b0b-b1da-2600525f812f",
    "task_status": "SUCCESS",
    "task_result": true
}
```

Test it out in the broser as well.