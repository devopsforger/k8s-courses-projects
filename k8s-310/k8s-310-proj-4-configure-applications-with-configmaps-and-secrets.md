# K8S-310 — Project 04 Reference Solution

## Configure Applications with ConfigMaps and Secrets

**Estimated time:** 3-5 hours  
**Level:** Intermediate

> Instructor reference. Follow the student brief first. Commands assume a disposable local Kind cluster named `k8s310`, a `taskforge` namespace, and the separately supplied TaskForge starter project. Replace placeholders with real names. Verify commands against your installed Kubernetes and tooling versions.

## Shared application and starter assets

Use the **TaskForge starter application**, distributed separately from these Markdown documents. The starter project contains a small web frontend, a FastAPI backend, PostgreSQL, a Docker Compose setup, a `.env.example`, and instructions for building and publishing the application images. The frontend calls the API; the API stores tasks in PostgreSQL. **No application source code is included in this project brief or reference solution.**

Download/extract the separately distributed `taskforge-starter.zip` (or clone the course's TaskForge starter repository when published). Before starting, follow its `README.md`, verify the application works locally using `docker compose up --build`, and publish or load the API image into Kind. Replace placeholders such as `<YOUR_REGISTRY>/taskforge-api:<TAG>` with the actual image you built. The image and repository names in this guide are examples, not live links.

> A Kubernetes cluster does not automatically see images built on your host. Use `kind load docker-image taskforge-api:dev --name k8s310` for a locally built image, or publish the image to a registry. Use `imagePullPolicy: IfNotPresent` for local Kind development. The frontend is optional for API-only exercises, but is required when demonstrating the full application.


## Why this solution works

Separate configuration from container images. The exercise emphasizes observable Kubernetes behavior, not merely successful resource creation. A correct implementation must demonstrate the requirements in the student brief, preserve data where relevant, and avoid leaking credentials.

## Preparation

1. Run `kubectl config current-context` and ensure you are using `kind-k8s310`, not a production context.
2. Run `kubectl get namespace taskforge || kubectl create namespace taskforge`.
3. Check `kubectl get nodes` and `kubectl get pods -n kube-system`. Do not proceed until nodes are Ready.
4. Build or pull the application images using the separate starter project's README. If a project needs the database, deploy the PostgreSQL baseline first.

## Detailed implementation

1. Create a dummy secret without putting its value in Git: `kubectl create secret generic taskforge-db -n taskforge --from-literal=POSTGRES_PASSWORD='local-demo-only-change-me' --dry-run=client -o yaml | kubectl apply -f -`. This command can appear in shell history; for real systems use a secure secret delivery mechanism.
2. Write `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: taskforge-settings
  namespace: taskforge
data:
  APP_ENV: "training"
  LOG_LEVEL: "info"
```

3. Apply with `kubectl apply -f configmap.yaml`.
4. In the API Deployment container add `envFrom: [{configMapRef: {name: taskforge-settings}}]`. For database settings use `env.valueFrom.secretKeyRef` and the keys supported by the starter application. If using `DATABASE_URL`, store the full URL in a Secret, not in the ConfigMap.
5. Run `kubectl apply -f api.yaml` and `kubectl rollout status deployment/taskforge-api -n taskforge`.
6. Verify nonsecret config: `kubectl exec -n taskforge deploy/taskforge-api -- printenv APP_ENV`. Do not print credentials in your submission.
7. Update `LOG_LEVEL` in the ConfigMap and run `kubectl rollout restart deployment/taskforge-api -n taskforge`. Environment variables from ConfigMaps are not automatically updated inside existing containers.
8. Explain that base64 encoding of Secret data is not encryption, and RBAC plus encryption-at-rest configuration are separate protections.

## Verification checklist

- [ ] Store non-sensitive API settings in a ConfigMap.
- [ ] Store a dummy database password in a Secret.
- [ ] Inject configuration into the API without embedding credentials in the image.
- [ ] Verify settings inside a Pod without printing secret values.
- [ ] Explain how to restart workloads after environment-variable changes.
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
