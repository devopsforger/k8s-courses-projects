# Kubernetes Course Projects | DevOpsForger

Hands-on Kubernetes projects for learners following the [DevOpsForger](https://devopsforger.com) Kubernetes courses.

This repository groups exercises by **course code**, with separate student briefs and reference solutions. Projects progress from creating a local cluster to deploying, configuring, networking, securing, packaging, and troubleshooting workloads.

## Courses

| Course | Level | Projects | Course page |
| --- | --- | ---: | --- |
| K8S-310: Kubernetes for Beginners | Beginner, progressing to intermediate exercises | 10 | [View course](https://devopsforger.com/courses/k8s-310) |

Additional Kubernetes courses can be added under `courses/` as they become available.

## K8S-310 project catalogue

| # | Project | Difficulty |
| --- | --- | --- |
| 01 | Set Up and Explore a Kubernetes Cluster | Beginner |
| 02 | Deploy and Manage Your First Application | Beginner |
| 03 | Scale, Update, and Recover an Application | Intermediate |
| 04 | Configure Applications with ConfigMaps and Secrets | Intermediate |
| 05 | Connect and Expose Applications | Intermediate |
| 06 | Deploy a Stateful Database with Persistent Storage | Intermediate |
| 07 | Secure Applications and Cluster Access | Intermediate |
| 08 | Run Jobs and Scheduled Workloads | Intermediate |
| 09 | Package and Deploy Applications with Helm | Intermediate |
| 10 | Troubleshoot and Recover a Broken Kubernetes Application | Intermediate · Final capstone |

Several projects reuse a small frontend, API, and PostgreSQL application. The application lives in a **separate starter repository**, linked from the relevant project briefs; the Markdown files document the Kubernetes work rather than embedding the application's source code.

## How to use this repository

1. Study the relevant modules in [K8S-310](https://devopsforger.com/courses/k8s-310).
2. Read the project brief and its acceptance criteria without opening the solution.
3. Create the required local cluster and obtain the versioned starter application or supplied assets referenced in the brief.
4. Implement your own Kubernetes manifests and run the verification commands.
5. Record the observed state and explain failures or design choices.
6. Compare your implementation with the reference solution only after attempting the task.
7. Remove project resources when finished. Check your active cluster context before any destructive operation.

## Prerequisites

- Docker or another supported local container runtime.
- [kubectl](https://kubernetes.io/docs/reference/kubectl/) and [Kind](https://kind.sigs.k8s.io/) for the default local-cluster workflow; Minikube may be used when the project supports it.
- [Helm](https://helm.sh/) for Project 09.
- A terminal, a text editor, and basic knowledge of containers and YAML.

Check the installed tools and selected context:

```bash
kubectl version --client
kind version
kubectl config current-context
kubectl get nodes
helm version
```

A cluster must exist and be reachable for the last three commands to succeed. Follow Project 01 if you have not created one yet.

### Cluster-dependent features

A local Kubernetes cluster does **not** automatically provide every feature used by these projects:

- **Ingress** requires a compatible Ingress controller and suitable local traffic routing.
- **NetworkPolicy enforcement** requires a CNI plugin that implements NetworkPolicy.
- **PersistentVolumeClaims** require suitable persistent volumes or a working StorageClass/provisioner.
- **LoadBalancer Services** may need additional local integration and are not guaranteed to receive an external address.

Each project should document the required add-ons and tested versions. If a feature is unsupported by your cluster, install the required component or use the project-approved alternative rather than assuming the YAML is broken.

## Safety and good practices

Use an isolated learning cluster, not a production cluster. Verify the active context and namespace before applying or deleting resources. Use fake credentials only: Kubernetes Secrets are not automatically encrypted at rest merely because they use the Secret resource type. Prefer least-privilege RBAC, explicit resource requests and limits, and repeatable manifests.

## Submissions and solutions

Follow the deliverables in each brief. Typical evidence includes manifest files, relevant `kubectl` output, successful endpoint checks, and a short explanation of troubleshooting steps. Reference solutions demonstrate a valid approach; different manifests may also meet the acceptance criteria.

Publish personal solutions in your own repository or fork, with instructions that let another learner reproduce your results.

## Issues and contributions

Report inaccuracies with the course code, project number, Kubernetes version, cluster type, installed add-ons, commands run, expected result, and observed result. Contributions should preserve the distinction between student-facing challenges and instructor/reference solutions.

## License

See the repository's `LICENSE` file, when provided, for reuse permissions. No license is implied by this README alone.
