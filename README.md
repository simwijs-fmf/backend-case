# backend-case
A backend case for a software engineering role at Find my factory (FMF) in **Kubernetes, Redis, Celery, and Python**. This case assesses your ability to solve a defined task and set up a **local Kubernetes stack** and implement **task orchestration with Celery and FastAPI**, while also handling **auto-scaling based on CPU usage**.

This test **simulates real-world backend engineering work** at FMF, requiring:
- Kubernetes infrastructure knowledge.
- Async task processing with Celery.
- API design with FastAPI.
- Autoscaling strategies.

It's designed to assess your general knowledge of modern systems and writing good code. If it's not apparent, I expect that you use AI alongside your own technical knowledge to solve the case. 

## **Objective**
Set up a **local Kubernetes stack** that includes:
- **FastAPI** (API layer)
- **Celery** (task queue)
- **Redis** (broker for Celery)
- **Horizontal Pod Autoscaling (HPA)** to scale Celery workers based on CPU usage.

### **Deliverables**
1. **Kubernetes manifests** (YAML files) for deploying:
   - Redis
   - Celery workers
   - Celery beat (optional, if they want to schedule periodic tasks)
   - FastAPI application
   - HPA configuration to scale Celery workers
2. **FastAPI app** that:
   - Accepts a `POST` request to trigger a Celery task.
   - Returns a task ID.
   - Provides a `GET` endpoint to check the task status.
3. **A Celery task** that consumes CPU resources and runs long enough to trigger auto-scaling.
4. **Instructions** on how to deploy and test the setup locally.
5. **(Optional)** A simple visualisation dashboard, written in Vue or other framework of choice that uses the endpoints from FastAPI. 

## **Requirements**
- **Language**: Python
- **Task Queue**: Celery
- **Message Broker**: Redis
- **API Framework**: FastAPI
- **Container Orchestration**: Kubernetes
- **Scaling**: Horizontal Pod Autoscaler (HPA)

## **Task Definition**
1. **Set up a Celery task** that simulates a CPU-intensive operation. Options:
   - **Option 1**: Perform a large matrix multiplication using NumPy.
   - **Option 2**: Compute a large Fibonacci sequence recursively (intentionally inefficient for CPU load).
   - **Option 3**: Simulate a long-running data processing job (e.g., hashing a large file multiple times).

   Example of a **CPU-intensive task**:
   ```python
   import numpy as np
   from celery import Celery

   # Broker and backend should be taken from environment variables in your real case
   celery_app = Celery("tasks", broker="redis://redis:6379/0", backend="redis://redis:6379/0")

   @celery_app.task
   def cpu_intensive_task(n: int):
       """Simulate a CPU-intensive task here"""
       return f"Completed CPU intensive task"
   ```

2. **FastAPI API Implementation**:
   ```python
   from fastapi import FastAPI, BackgroundTasks
   from tasks import cpu_intensive_task  # Import Celery task

   app = FastAPI()

   @app.post("/start-task/")
   def start_task(n: int = 1000):
       """Start a CPU-intensive Celery task and return the task ID."""
       task = cpu_intensive_task.delay(n)
       return {"task_id": task.id}

   @app.get("/task-status/{task_id}")
   def get_task_status(task_id: str):
       """Check the status of a Celery task."""
       ...
   ```

3. **Kubernetes Deployment (Manifests)**
   - **FastAPI Deployment & Service**
   - **Redis Deployment & Service**
   - **Celery Worker Deployment (with CPU requests/limits)**
   - **HPA (Horizontal Pod Autoscaler) based on CPU usage**

   Example **HPA configuration**:
   ```yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: celery-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: celery-worker
     minReplicas: 1
     maxReplicas: 5
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 50
   ```


## **Evaluation Criteria**
- **Containerization & Kubernetes**:
  - Correct **Kubernetes manifests** (FastAPI, Redis, Celery, HPA).
  - Use of **Helm or Kustomize** (bonus points).
- **Celery Implementation**:
  - Proper **Celery worker setup** and integration with Redis.
  - Celery workers should process tasks and report completion.
- **FastAPI Implementation**:
  - API should start tasks and fetch statuses correctly.
  - Clean API design and error handling.
- **Scaling Mechanism**:
  - The Kubernetes **HPA should correctly scale Celery workers** when CPU usage spikes.
- **Frontend**:
  - Bonus points if you decide to include a simple dashboard (this will require extending FastAPI) 

## **Instructions**
1. **Deploy the stack locally using Kubernetes**.
2. **Test the endpoints**:
   - `POST /start-task/` with a number to trigger a CPU-intensive Celery task.
   - `GET /task-status/{task_id}` to track the task’s progress.
3. **Observe autoscaling**:
   - Use `kubectl get hpa` and `kubectl get pods` to monitor pod scaling.
4. **Test case**:
   - Make sure that you test your code. I will be intentionally trying to break the system. Good systems are reliabe.
