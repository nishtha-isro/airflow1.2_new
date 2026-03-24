---
description: How to run and test the Multi-Node Celery Execution locally
---

# Running the Multi-Node Airflow Setup

This document provides instructions on how to set up the environment and trigger your Celery multi-node test DAGs (`test_multi_node_celery`).

## 1. Start the Environment

Make sure you are in the `Tutorial2` directory. You will need Docker Desktop running. 

Start the cluster (which includes webserver, Postgres, Redis broker, Flower monitor, Scheduler, and Worker):
```bash
docker-compose up --build -d
```
Wait a couple of minutes for all the services to start and initialize.

## 2. Verify Services

Check that your UIs are up:
- **Airflow Webserver**: [http://localhost:8085](http://localhost:8085)
- **Flower (Celery Monitor)**: [http://localhost:5557](http://localhost:5557)

*In the Airflow UI, find `test_multi_node_celery` and toggle it ON.*

## 3. Triggering the DAG Multiple Times

Because this is a Dockerized environment, the `airflow` CLI needs to run *inside* the container, not on your local machine.

Run the appropriate command below in your local terminal to trigger the DAG 10 times consecutively:

**Windows (Command Prompt):**
```cmd
for /l %x in (1, 1, 10) do docker-compose exec webserver airflow dags trigger test_multi_node_celery
```

**Windows (PowerShell):**
```powershell
1..10 | ForEach-Object { docker-compose exec webserver airflow dags trigger test_multi_node_celery }
```

**Linux / Git Bash / macOS:**
```bash
for i in {1..10}; do docker-compose exec webserver airflow dags trigger test_multi_node_celery; done
```

## 4. What to Observe

During and after the execution, monitor these two areas:

1. **Airflow Task Logs and UI**: 
   - Check the `test_multi_node_celery` Gantt Chart. You should clearly see parallel execution paths natively starting side-by-side (`task_2` and `task_3`).
   - Read the individual python task logs. Look for: `Task executing on worker node / hostname: <ID>`. You will notice the task hostnames naturally rotate across your cluster since Celery distributes the load.

2. **Flower Dashboard (http://localhost:5557/tasks)**:
   - Go to the Tasks page. Because you enqueued 50 tasks (10 DAGs x 5 Tasks), you will see Celery workers dynamically pulling these messages from the queue. You can monitor the live throughput and concurrency behavior of your architecture in real time.
