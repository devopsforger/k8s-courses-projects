# K8S-310 — Project 02 Reference Solution

## Deploy and Manage Your First Application

**Estimated time:** 3-5 hours  
**Level:** Beginner

> Instructor reference. Follow the student brief first. Commands assume a disposable local Kind cluster named `k8s310`, a `taskforge` namespace, and the separately supplied TaskForge starter project. Replace placeholders with real names. Verify commands against your installed Kubernetes and tooling versions.

## Shared application and starter assets

Use the **TaskForge starter application**, distributed separately from these Markdown documents. The starter project contains a small web frontend, a FastAPI backend, PostgreSQL, a Docker Compose setup, a `.env.example`, and instructions for building and publishing the application images. The frontend calls the API; the API stores tasks in PostgreSQL. **No application source code is included in this project brief or reference solution.**

Download/extract the separately distributed `taskforge-starter.zip` (or clone the course's TaskForge starter repository when published). Before starting, follow its `README.md`, verify the application works locally using `docker compose up --build`, and publish or load the API image into Kind. Replace placeholders such as `<YOUR_REGISTRY>/taskforge-api:<TAG>` with the actual image you built. The image and repository names in this guide are examples, not live links.

> A Kubernetes cluster does not automatically see images built on your host. Use `kind load docker-image taskforge-api:dev --name k8s310` for a locally built image, or publish the image to a registry. Use `imagePullPolicy: IfNotPresent` for local Kind development. The frontend is optional for API-only exercises, but is required when demonstrating the full application.


## Why this solution works

Deploy the supplied API image declaratively. The exercise emphasizes observable Kubernetes behavior, not merely successful resource creation. A correct implementation must demonstrate the requirements in the student brief, preserve data where relevant, and avoid leaking credentials.

## Preparation

1. Run `kubectl config current-context` and ensure you are using `kind-k8s310`, not a production context.
2. Run `kubectl get namespace taskforge || kubectl create namespace taskforge`.
3. Check `kubectl get nodes` and `kubectl get pods -n kube-system`. Do not proceed until nodes are Ready.
4. Build or pull the application images using the separate starter project's README. If a project needs the database, deploy the PostgreSQL baseline first.

## Detailed implementation

1. Build the starter API image in its backend directory: `docker build -t taskforge-api:dev ./backend` from the starter root. Load it: `kind load docker-image taskforge-api:dev --name k8s310`.
2. Create `api.yaml` from the following example. Confirm the starter API's health endpoint is `/health`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: taskforge-api
  namespace: taskforge
spec:
  replicas: 2
  selector:
    matchLabels: {app: taskforge-api}
  template:
    metadata:
      labels: {app: taskforge-api}
    spec:
      containers:
        - name: api
          image: taskforge-api:dev
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: taskforge-api-db
                  key: DATABASE_URL
---
apiVersion: v1
kind: Service
metadata:
  name: taskforge-api
  namespace: taskforge
spec:
  type: ClusterIP
  selector: {app: taskforge-api}
  ports:
    - port: 8000
      targetPort: 8000
```

3. The starter API requires PostgreSQL. First deploy the working PostgreSQL baseline described in Project 06 (or use the instructor-supplied database manifest). Create a Secret named `taskforge-api-db` with key `DATABASE_URL` containing the PostgreSQL connection URL. For a disposable local lab, use `kubectl create secret generic taskforge-api-db -n taskforge --from-literal=DATABASE_URL='postgresql+psycopg2://taskforge:<URL_ENCODED_PASSWORD>@postgres:5432/taskforge'`. Replace the placeholder and avoid exposing credentials in terminal history or submissions. Verify the database is Ready before applying the API.
4. Apply `kubectl apply -f api.yaml` after replacing the placeholder configuration. Check `kubectl rollout status deployment/taskforge-api -n taskforge`.
5. Run `kubectl port-forward -n taskforge svc/taskforge-api 8000:8000`; in another terminal run `curl -f http://localhost:8000/health`.
6. Inspect endpoints: `kubectl get endpointslices -n taskforge -l kubernetes.io/service-name=taskforge-api`. Two ready endpoints should appear.
7. Verify the Service selector exactly matches Pod labels. Stop port-forward with Ctrl+C.

## Verification checklist

- [ ] Deploy the TaskForge API as a Deployment with two replicas.
- [ ] Use a ClusterIP Service with correct selectors.
- [ ] Set clear labels and namespace.
- [ ] Verify readiness and reach the API health endpoint.
- [ ] Do not expose PostgreSQL publicly.
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
