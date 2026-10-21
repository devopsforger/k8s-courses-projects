# K8S-310 — Project 08 Reference Solution

## Run Jobs and Scheduled Workloads

**Estimated time:** 3-5 hours  
**Level:** Intermediate

> Instructor reference. Follow the student brief first. Commands assume a disposable local Kind cluster named `k8s310`, a `taskforge` namespace, and the separately supplied TaskForge starter project. Replace placeholders with real names. Verify commands against your installed Kubernetes and tooling versions.

## Shared application and starter assets

Use the **TaskForge starter application**, distributed separately from these Markdown documents. The starter project contains a small web frontend, a FastAPI backend, PostgreSQL, a Docker Compose setup, a `.env.example`, and instructions for building and publishing the application images. The frontend calls the API; the API stores tasks in PostgreSQL. **No application source code is included in this project brief or reference solution.**

Download/extract the separately distributed `taskforge-starter.zip` (or clone the course's TaskForge starter repository when published). Before starting, follow its `README.md`, verify the application works locally using `docker compose up --build`, and publish or load the API image into Kind. Replace placeholders such as `<YOUR_REGISTRY>/taskforge-api:<TAG>` with the actual image you built. The image and repository names in this guide are examples, not live links.

> A Kubernetes cluster does not automatically see images built on your host. Use `kind load docker-image taskforge-api:dev --name k8s310` for a locally built image, or publish the image to a registry. Use `imagePullPolicy: IfNotPresent` for local Kind development. The frontend is optional for API-only exercises, but is required when demonstrating the full application.


## Why this solution works

Run a finite batch task. The exercise emphasizes observable Kubernetes behavior, not merely successful resource creation. A correct implementation must demonstrate the requirements in the student brief, preserve data where relevant, and avoid leaking credentials.

## Preparation

1. Run `kubectl config current-context` and ensure you are using `kind-k8s310`, not a production context.
2. Run `kubectl get namespace taskforge || kubectl create namespace taskforge`.
3. Check `kubectl get nodes` and `kubectl get pods -n kube-system`. Do not proceed until nodes are Ready.
4. Build or pull the application images using the separate starter project's README. If a project needs the database, deploy the PostgreSQL baseline first.

## Detailed implementation

1. Choose a standard CLI image for a self-contained smoke test. Write `job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: taskforge-smoke
  namespace: taskforge
spec:
  backoffLimit: 2
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: check
          image: curlimages/curl:8.11.1
          command: ["sh", "-c", "curl --fail --silent --show-error http://taskforge-api:8000/health"]
```

2. Apply: `kubectl apply -f job.yaml`. Verify `kubectl wait --for=condition=complete job/taskforge-smoke -n taskforge --timeout=120s` and `kubectl logs -n taskforge job/taskforge-smoke`.
3. Create a CronJob from the same Pod template with `schedule: "*/5 * * * *"`, `concurrencyPolicy: Forbid`, `successfulJobsHistoryLimit: 2`, and `failedJobsHistoryLimit: 1`.
4. Apply and inspect `kubectl get cronjobs,jobs -n taskforge`.
5. Trigger manually: `kubectl create job --from=cronjob/taskforge-smoke-scheduled taskforge-manual -n taskforge`; inspect status and logs.
6. Test failure by changing the endpoint to a nonexistent path in a separate Job; record nonzero exit and failure events. Restore the working configuration.
7. Explain why a Job should be finite, why a CronJob is not a precise real-time scheduler, and why init containers are used to perform prerequisite work before application containers start.

## Verification checklist

- [ ] Create a Job that executes a read-only database check or API smoke test.
- [ ] Verify completion and collect logs.
- [ ] Create a CronJob scheduled every five minutes for demonstration.
- [ ] Set concurrencyPolicy Forbid and history limits.
- [ ] Trigger a one-off Job from the CronJob and inspect it.
- [ ] Explain when init containers are appropriate.
- [ ] Capture `kubectl get pods -n taskforge -o wide` and relevant `kubectl describe` output.
- [ ] Verify behavior using the API health endpoint and, when a database is present, a create/read task test.
- [ ] Check `kubectl get events -n taskforge --sort-by=.lastTimestamp` for unexpected warnings.
- [ ] Record the exact image tag and cluster context used for reproducibility.

## Expected observations and interpretation

Healthy workloads show desired and available replica counts in agreement; ready endpoints appear behind Services; successful Jobs have a Complete condition; PVCs are Bound when storage provisioning works. These observations depend on the exercise. A resource can exist while still being misconfigured, so always test actual application behavior.

## Troubleshooting decision path

1. **Resource missing:** Check namespace and `kubectl apply` errors.
2. **Pod Pending:** Inspect scheduling events, quotas, PVC binding, and available resources.
3. **ImagePullBackOff:** Inspect image name, tag, registry authentication, and whether Kind has the image.
4. **CrashLoopBackOff:** Inspect current and previous container logs, configuration, and startup dependencies.
5. **Not Ready:** Verify probe endpoint, application port, database connectivity, and container health.
6. **Service unreachable:** Check selectors, EndpointSlices, DNS, and NetworkPolicy enforcement.
7. **Data missing:** Check the actual PVC mount, PostgreSQL data directory, and whether a new empty claim was accidentally created.

## Cleanup and safety

Delete only the resources created specifically for this exercise, unless continuing to the next project. **Do not delete persistent volume claims or the cluster until data has been backed up or is confirmed disposable.** Do not include credentials, kubeconfigs, or tokens in screenshots or logs. For instructor fault injection, always use a disposable cluster.

## Suggested submission evidence

Include the final YAML or chart, command transcript with sensitive values removed, screenshots only where they add value, a short explanation of tradeoffs, and the successful acceptance tests. State any environment-dependent differences from these instructions.
