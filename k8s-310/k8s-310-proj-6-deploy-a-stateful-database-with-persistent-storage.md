# K8S-310 — Project 06 Reference Solution

## Deploy a Stateful Database with Persistent Storage

**Estimated time:** 5-7 hours  
**Level:** Intermediate

> Instructor reference. Follow the student brief first. Commands assume a disposable local Kind cluster named `k8s310`, a `taskforge` namespace, and the separately supplied TaskForge starter project. Replace placeholders with real names. Verify commands against your installed Kubernetes and tooling versions.

## Shared application and starter assets

Use the **TaskForge starter application**, distributed separately from these Markdown documents. The starter project contains a small web frontend, a FastAPI backend, PostgreSQL, a Docker Compose setup, a `.env.example`, and instructions for building and publishing the application images. The frontend calls the API; the API stores tasks in PostgreSQL. **No application source code is included in this project brief or reference solution.**

Download/extract the separately distributed `taskforge-starter.zip` (or clone the course's TaskForge starter repository when published). Before starting, follow its `README.md`, verify the application works locally using `docker compose up --build`, and publish or load the API image into Kind. Replace placeholders such as `<YOUR_REGISTRY>/taskforge-api:<TAG>` with the actual image you built. The image and repository names in this guide are examples, not live links.

> A Kubernetes cluster does not automatically see images built on your host. Use `kind load docker-image taskforge-api:dev --name k8s310` for a locally built image, or publish the image to a registry. Use `imagePullPolicy: IfNotPresent` for local Kind development. The frontend is optional for API-only exercises, but is required when demonstrating the full application.


## Why this solution works

Deploy PostgreSQL with persistent storage. The exercise emphasizes observable Kubernetes behavior, not merely successful resource creation. A correct implementation must demonstrate the requirements in the student brief, preserve data where relevant, and avoid leaking credentials.

## Preparation

1. Run `kubectl config current-context` and ensure you are using `kind-k8s310`, not a production context.
2. Run `kubectl get namespace taskforge || kubectl create namespace taskforge`.
3. Check `kubectl get nodes` and `kubectl get pods -n kube-system`. Do not proceed until nodes are Ready.
4. Build or pull the application images using the separate starter project's README. If a project needs the database, deploy the PostgreSQL baseline first.

## Detailed implementation

1. Check storage support: `kubectl get storageclass`. Kind often supplies a local-path provisioner, but verify rather than assume.
2. Create a Secret containing dummy PostgreSQL credentials (as in Project 04).
3. Write `postgres.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: taskforge
spec:
  clusterIP: None
  selector: {app: postgres}
  ports: [{port: 5432, targetPort: 5432}]
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: taskforge
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels: {app: postgres}
  template:
    metadata:
      labels: {app: postgres}
    spec:
      containers:
        - name: postgres
          image: postgres:16
          ports: [{containerPort: 5432}]
          env:
            - name: POSTGRES_DB
              value: taskforge
            - name: POSTGRES_USER
              value: taskforge
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: taskforge-db
                  key: POSTGRES_PASSWORD
          volumeMounts:
            - name: pgdata
              mountPath: /var/lib/postgresql/data
              subPath: postgres
  volumeClaimTemplates:
    - metadata:
        name: pgdata
      spec:
        accessModes: [ReadWriteOnce]
        resources:
          requests:
            storage: 1Gi
```

4. Apply `kubectl apply -f postgres.yaml`, then `kubectl rollout status statefulset/postgres -n taskforge` and `kubectl get pvc -n taskforge`.
5. Configure the API's database connection with a Secret. PostgreSQL hostname inside the namespace is `postgres`; use port 5432. Create a task using the frontend or documented API endpoint and record its ID.
6. Delete only the Pod: `kubectl delete pod postgres-0 -n taskforge`. Wait for the StatefulSet to recreate it.
7. Fetch the task again; it must still exist. Record the PVC name before and after.
8. A single Pod plus PVC is not a backup or a high-availability architecture. Never run `kubectl delete pvc` as part of the persistence test.

## Verification checklist

- [ ] Create a headless Service for PostgreSQL.
- [ ] Deploy PostgreSQL as a single-replica StatefulSet.
- [ ] Use a volumeClaimTemplate and verify Bound PVC.
- [ ] Create a test task through the API or a SQL row.
- [ ] Delete the PostgreSQL Pod and prove the data remains.
- [ ] Explain why a single-replica database is not highly available.
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
