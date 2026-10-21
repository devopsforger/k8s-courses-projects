# K8S-310: Kubernetes for Beginners

**Course:** [Kubernetes for Beginners on DevOpsForger](https://devopsforger.com/courses/k8s-310)

This directory contains the practical projects for K8S-310, an introductory course designed to teach students how to deploy, manage, configure, secure, and troubleshoot containerized applications using Kubernetes.

The projects follow a progressive approach. Students begin by creating and exploring a Kubernetes cluster, then gradually advance toward deploying, operating, securing, and troubleshooting multi-service applications.

## Learning Objectives

By completing these projects, you should be able to:

- Create and explore a local Kubernetes cluster.
- Use `kubectl` to inspect and manage Kubernetes resources.
- Deploy applications using Pods, Deployments, Services, and declarative YAML manifests.
- Scale applications, perform rolling updates, and roll back failed deployments.
- Configure applications using ConfigMaps and Secrets.
- Connect and expose workloads using Kubernetes networking.
- Deploy stateful applications using StatefulSets and persistent storage.
- Implement basic security controls using ServiceAccounts, RBAC, and security contexts.
- Run one-time and scheduled workloads using Jobs and CronJobs.
- Package and manage Kubernetes applications using Helm.
- Diagnose and resolve common application and cluster configuration problems.

## Prerequisites

Before starting, ensure you have:

- Docker Engine or Docker Desktop installed and running.
- Kind or Minikube for running a local Kubernetes cluster.
- `kubectl` installed and configured.
- Helm installed for the Helm project.
- Git and a code editor.
- A terminal or command-line interface.
- Basic understanding of containers, images, and Docker.

Some projects require additional components, such as an Ingress controller, a compatible networking plugin, or a storage provisioner. The relevant project instructions should identify these requirements.

We recommend completing [DOC-301: Docker & Containerization](https://devopsforger.com/courses/doc-301) before beginning this course if you are new to containers.

## Projects

| # | Project | Difficulty |
|---|---|---|
| 01 | [Set Up and Explore a Kubernetes Cluster](k8s-310-proj-1-set-up-and-explore-a-kubernetes-cluster.md) | Beginner |
| 02 | [Deploy and Manage Your First Application](k8s-310-proj-2-deploy-and-manage-your-first-application.md) | Beginner |
| 03 | [Scale, Update, and Recover an Application](k8s-310-proj-3-scale-update-and-recover-an-application.md) | Intermediate |
| 04 | [Configure Applications with ConfigMaps and Secrets](k8s-310-proj-4-configure-applications-with-configmaps-and-secrets.md) | Intermediate |
| 05 | [Connect and Expose Applications](k8s-310-proj-5-connect-and-expose-applications.md) | Intermediate |
| 06 | [Deploy a Stateful Database with Persistent Storage](k8s-310-proj-6-deploy-a-stateful-database-with-persistent-storage.md) | Intermediate |
| 07 | [Secure Applications and Cluster Access](k8s-310-proj-7-secure-applications-and-cluster-access.md) | Intermediate |
| 08 | [Run Jobs and Schedule Workloads](k8s-310-proj-8-run-jobs-and-schedule-workloads.md) | Intermediate |
| 09 | [Package and Deploy Applications with Helm](k8s-310-proj-9-package-and-deploy-applications-with-helm.md) | Intermediate |
| 10 | [Troubleshoot and Recover a Broken Kubernetes Application](k8s-310-proj-10-troubleshoot-and-recover-a-broken-kubernetes-application.md) | Intermediate |

## How to Complete the Projects

We recommend completing the projects in order, as later projects build upon concepts introduced in earlier exercises.

For each project:

1. **Read the project description.** Understand the scenario, objectives, and expected outcome.
2. **Review the requirements.** Identify which Kubernetes resources you must create, configure, or troubleshoot.
3. **Prepare your environment.** Confirm that your cluster is running, `kubectl` is configured, and any required starter files are available.
4. **Implement your solution.** Apply the concepts learned in the course using Kubernetes commands and declarative manifests.
5. **Verify your implementation.** Test your work against the project's acceptance criteria.
6. **Document your findings.** Explain your approach, technical decisions, challenges, and solutions.

The goal is not merely to create Kubernetes resources, but to understand how those resources work together and how to operate them correctly.

## Starter Application

Some projects use a Task Management application to provide a consistent environment for learning Kubernetes.

The application is provided separately and may include:

- A frontend for interacting with the application.
- A backend API for managing tasks.
- A PostgreSQL database for persistent storage.
- Docker images or Dockerfiles for deploying the application.

Students should use the starter application referenced in each project rather than developing the application from scratch.

## Assessment and Verification

Each project contains acceptance criteria describing the expected results.

Your implementation should satisfy those criteria before you consider the project complete.

Where applicable, verify that:

- Kubernetes resources are created successfully.
- Pods start and reach their expected readiness state.
- Applications are accessible through the configured Services or Ingress routes.
- Workloads communicate correctly using Kubernetes networking.
- Configuration changes are applied as expected.
- Persistent data survives Pod replacement.
- Security policies and access permissions behave as intended.
- Application updates and rollbacks complete successfully.
- Failures can be diagnosed and resolved.

## Reference Solutions

Reference solutions are intended to help students understand correct implementations and review their work.

Attempt each project independently before consulting its solution.

There may be multiple valid approaches to solving a problem. What matters is that your implementation meets the requirements and that you understand the decisions you made.

## Environment and Safety

These projects are intended primarily for local learning environments.

Use a dedicated Kubernetes cluster or namespace for exercises. Avoid running destructive commands against production or shared clusters.

Use dummy credentials for demonstrations, avoid committing real secrets, and remove unused resources when you finish an exercise.

Some projects intentionally introduce failures. Ensure you understand the scope of these changes before applying them.

## Next Steps

After completing these projects, you should have a practical understanding of how Kubernetes manages containerized applications and how its resources work together.

You can continue developing your skills by exploring advanced Kubernetes networking, cluster administration, observability, GitOps, and production security practices.

Return to the [K8S-310 course](https://devopsforger.com/courses/k8s-310) to review lessons or revisit concepts you need to strengthen.