# Trigger a Task

An event handler in *project/statis/main.js* is set up that listens for a button click. On click, an AJAX POST request is sent to the receiver with the appropriate task type: `1`, `2`, or `3`.`

```javascript
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

On the server-side, a view is already configured to handle the request in *project/tasks/views.py*.


```py
@csrf_exempt
def run_task(request):
    if request.POST:
        task_type = request.POST.get("type")
        return JsonResponse({"task_type": task_type}, status=202)
```

Now comes the fun part: wiring up Celery!