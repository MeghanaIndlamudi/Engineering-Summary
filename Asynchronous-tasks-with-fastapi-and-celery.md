# Asynchronous Tasks with FastAPI and Celery

This document provides detailed notes on how to integrate Celery into a FastAPI application to handle long-running processes in the background. It covers setting up and configuring Celery and Redis, containerizing with Docker, testing Celery tasks, and monitoring jobs with Flower.

---

## **Why Use Asynchronous Tasks?**

Long-running tasks, like resizing images or sending emails, should not block the request/response cycle of a web application. Instead, they should run in the background to:
- Improve user experience.
- Allow the application to respond to other requests.
- Reduce unnecessary wait times for users.

### **Examples of Background Tasks**
- Running machine learning models
- Sending confirmation emails
- Web scraping
- Analyzing data
- Processing images
- Generating reports

---

## **FastAPI's BackgroundTasks vs. Celery**

### **FastAPI's BackgroundTasks**
Use for lightweight background tasks within the same event loop.

Example:
```python
from fastapi import BackgroundTasks

def send_email(email, message):
    pass

@app.get("/")
async def ping(background_tasks: BackgroundTasks):
    background_tasks.add_task(send_email, "email@address.com", "Hi!")
    return {"message": "pong!"}
```

### **Celery**
Use Celery for:
- **CPU-intensive tasks**: Heavy computations.
- **Task queue management**: Retrieving task status, retrying failed tasks, etc.

---

## **Workflow Overview**

1. User initiates a task via a POST request.
2. Task is added to the queue and its ID is sent back to the client.
3. The client polls the server to check the task status.
4. The task runs in the background, processed by a Celery worker.

---

## **Project Setup**

### **Clone the Repository**
```bash
git clone https://github.com/testdrivenio/fastapi-celery --branch v1 --single-branch
cd fastapi-celery
git checkout v1 -b master
```

### **Run the Application**
Build and start Docker containers:
```bash
docker-compose up -d --build
```

### **Verify Tests**
```bash
docker-compose exec web python -m pytest
```

Expected output:
```
=============================== test session starts ================================
tests/test_tasks.py .                                                        [100%]
================================ 1 passed in 0.20s =================================
```

---

## **Project Structure**

```
├── .gitignore
├── LICENSE
├── README.md
├── docker-compose.yml
└── project
    ├── Dockerfile
    ├── main.py
    ├── requirements.txt
    ├── static
    │   ├── main.css
    │   └── main.js
    ├── templates
    │   ├── _base.html
    │   ├── footer.html
    │   └── home.html
    └── tests
        ├── __init__.py
        ├── conftest.py
        └── test_tasks.py
```

---

## **Celery Setup**

### **Add Dependencies**
Update `requirements.txt`:
```
aiofiles==23.1.0
celery==5.2.7
fastapi==0.95.0
redis==4.5.4
```

### **Update `docker-compose.yml`**
Add Celery worker and Redis services:
```yaml
version: '3.8'

services:

  web:
    build: ./project
    ports:
      - 8004:8000
    command: uvicorn main:app --host 0.0.0.0 --reload
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
    depends_on:
      - redis

  worker:
    build: ./project
    command: celery -A worker.celery worker --loglevel=info
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
    depends_on:
      - web
      - redis

  redis:
    image: redis:7
```

---

### **Create a Celery Worker**
Create `worker.py`:
```python
import os
import time

from celery import Celery

celery = Celery(__name__)
celery.conf.broker_url = os.environ.get("CELERY_BROKER_URL", "redis://localhost:6379")
celery.conf.result_backend = os.environ.get("CELERY_RESULT_BACKEND", "redis://localhost:6379")

@celery.task(name="create_task")
def create_task(task_type):
    time.sleep(int(task_type) * 10)
    return True
```

---

## **Trigger a Task**

### **Update `main.py`**
Add a route to initiate a Celery task:
```python
from worker import create_task

@app.post("/tasks", status_code=201)
def run_task(payload = Body(...)):
    task_type = payload["type"]
    task = create_task.delay(int(task_type))
    return JSONResponse({"task_id": task.id})
```

### **Trigger Task via cURL**
```bash
curl http://localhost:8004/tasks -H "Content-Type: application/json" --data '{"type": 1}'
```

Expected response:
```json
{
  "task_id": "<unique-task-id>"
}
```

---

## **Monitor Task Status**

### **Update Client-side Code**
Modify `main.js`:
```javascript
function handleClick(type) {
  fetch('/tasks', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ type: type }),
  })
  .then(response => response.json())
  .then(res => getStatus(res.task_id));
}

function getStatus(taskId) {
  fetch(`/tasks/${taskId}`)
    .then(response => response.json())
    .then(data => console.log(data.status));
}
```

---

## **Monitoring with Flower**

### **Install Flower**
Add to `requirements.txt`:
```
flower==1.0.0
```

### **Update `docker-compose.yml`**
```yaml
  flower:
    image: mher/flower
    ports:
      - 5555:5555
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
    depends_on:
      - redis
```

### **Access Flower Dashboard**
Navigate to [http://localhost:5555](http://localhost:5555).

---

## **Testing Celery Tasks**

### **Write Tests**
Example test in `test_tasks.py`:
```python
def test_create_task():
    task = create_task.delay(1)
    assert task.get(timeout=10) == True
```

Run tests:
```bash
docker-compose exec web pytest
