# ci-cd-test

This repository serves as a **minimal testbed** for the custom CI/CD runner **Argon**. It is designed to simulate real-world build steps and help validate the full pipeline from webhook trigger to job execution and notification.

---

## Purpose

This repo exists purely to test and iterate on the CI/CD flow, ensuring that:
- **Webhooks** from GitHub/GitLab are received correctly.
- The **orchestrator** queues and dispatches jobs.
- The **sandbox** runs build/test steps inside Docker containers.
- The **notification** service receives success/failure messages.

---

## Getting Started

### 1. Setup Your Webhook

To connect this repo to the CI/CD pipeline, add a **Webhook** to your GitHub/GitLab project:

- **URL**: `http://<your-public-orchestrator-endpoint>/webhook`
- **Content type**: `application/json`
- **Triggers**: Push events (or customize as needed)
- (Optional) Add a secret for validation

---

## Project Structure

This repo must contain a .runnerci.yml :

### Sample runnerci.yml

```yml
version: 1.0

jobs:
  build:
    image: python:3.10
    steps:
      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run main script
        run: python main.py
```

They are used by Argon to simulate real CI/CD steps inside a Docker sandbox.

---

## Example Flow

1. You push code to this repo.
2. GitHub sends a webhook to your **webhook listener**.
3. The listener passes it to the **orchestrator**.
4. The orchestrator schedules and dispatches the job to **Argon's sandbox**.
5. Each step is run in isolation (in `/app`) inside a Docker container.
6. After execution, a message is sent to the **notification queue** indicating success or failure.

---

## Notifications

You should see JSON payloads like:

```json
{
  "message": "Job failed at step 'Run tests'. Check container logs for details."
}
````

or

```json
{
  "message": "Job completed successfully."
}
```

Sent to: `notification.queue`

---

## Add Test Steps

You can add simple shell scripts like:

```bash
#!/bin/bash
echo "Running build"
exit 0
```

And push it to trigger the pipeline.

---

## Notes

* Make sure your **Argon services** are running: listener, orchestrator, sandbox, notification.
* All temporary containers, repo clones, and logs are handled by Argon internally.
* Keep this repo intentionally minimal.



