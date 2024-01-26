# Asynchronous Tasks with Django and Celery

If a long-running process is part of your application's workflow, rather than blocking the response, you should handle it in the background, outside the normal request/response flow.

Perhaps your web application requires users to submit a thumbnail (which will probably need to be re-sized) and confirm their email when they register. If your application processed the image and sent a confirmation email directly to the request handler, then the end user would have to wait unnecessarily for them both to finish processing before the page loads or updates. Instead, you'll want to pass these processes off to a task queue and let a separate worker process deal with it, so you can immediately send a response back to the client. The end user can then do other things on the client side while the processing takes place. Your application is also free to respond to requests from other users and clients.

To achieve this, we'll walk you through the process of setting up and configuring Celery and Redis for handling long-running processes in a Django app.

We'll also use Docker and Docker Compose to tie everything together. Finally, we'll look at how to test the Celery tasks with unit and integration tests.

Contents:

- [Objectives, Backgrounds Tasks and Workflows](docs/101-objectives.md)
- [Project Setup](docs/102-project-setup.md)
- [Trigger a Task](docs/103-trigger-a-task.md)
- [Celery Setup](docs/104-celery-setup.md)
- [Trigger a Task](docs/105-trigger-a-task.md)
- [Task Status](docs/106-task-status.md)
- [Celery Logs](docs/107-celery-logs.md)
- [Flower Dashboard](docs/108-flow-dashboard.md)
- [Tests](docs/109-tests.md)
- [Conclusion](https://)