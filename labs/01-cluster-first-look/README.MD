# Lab 01: First Look at a Kubernetes Cluster

**Level:** Beginner
**Track:** DevOps / SRE
**Environment:** Ubuntu, Docker, Kind, kubectl
**Estimated time:** 60–90 minutes
**Status:** In progress

## 1. Scenario

You have joined a DevOps team as a junior engineer. Your first task is to prepare a local Kubernetes environment, inspect its components, deploy a web application, and confirm that the application is healthy.

You must document the work so another engineer can reproduce it.

## 2. Learning objectives

By completing this lab, you will learn to:

- Explain the purpose of a Kubernetes cluster and its nodes.
- Understand the roles of Pods and Deployments.
- Create and inspect a local Kind cluster.
- Use kubectl to query cluster resources.
- Deploy an NGINX application.
- Access an application using port forwarding.
- Troubleshoot basic deployment and connectivity problems.
- Document technical work in GitHub.

## 3. Prerequisites

- Ubuntu Linux.
- Docker Engine installed and running.
- `kubectl` installed.
- Kind installed.
- Internet access to download the required container image.
- Basic Linux terminal knowledge.

Verify your environment:

```bash
docker --version
docker info
kubectl version --client
kind version
```

**Expected result:** Each command completes successfully. `docker info` should show information about the Docker server, not just the client.

If a tool is missing, consult the official installation guides:

- Docker: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
- kubectl: [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)
- Kind: [https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/)

## 4. Repository structure

```text
01-cluster-first-look/
├── README.md
├── manifests/
│   └── demo-deployment.yaml
├── hints.md
├── troubleshooting.md
└── solution.md
```

Create the directories:

```bash
mkdir -p manifests
touch hints.md troubleshooting.md solution.md
```

## 5. Tasks

### Task 1: Inspect the local environment

Run:

```bash
docker ps
kubectl config get-contexts
kind get clusters
```

Record the output and answer:

1. Is Docker running?
2. Is a Kubernetes context already configured?
3. Are any Kind clusters already present?

Do not delete existing clusters just to start this lab.

### Task 2: Create a Kubernetes cluster

Create a cluster named `k8s-lab`:

```bash
kind create cluster --name <cluster-name> --wait 120s
```

Verify:

```bash
kind get clusters
kubectl config current-context
kubectl cluster-info
kubectl get nodes -o wide
```

**Acceptance criteria:**

- The cluster is listed by Kind.
- The current context refers to `kind-k8s-lab`.
- The node reaches `Ready`.

### Task 3: Explore the cluster

Run:

```bash
kubectl get nodes
kubectl describe nodes
kubectl get namespaces
kubectl get pods -A
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

Answer:

1. What is a node?
2. Why does Kubernetes use namespaces?
3. What is the difference between a Pod and a container?
4. What are the `kube-system` namespace and its components used for?
5. Why are events useful when troubleshooting?

Save useful output in your lab notes. Avoid dumping large amounts of unfiltered output into the repository.

### Task 4: Deploy an NGINX application

Create `manifests/demo-deployment.yaml` with a Kubernetes Deployment that has:

- Name: `demo-web`
- One replica
- Container name: `nginx`
- Image: `nginx:stable`
- Container port: `80`

Apply the manifest:

```bash
kubectl apply -f manifests/demo-deployment.yaml
```

Verify:

```bash
kubectl get deployments
kubectl get replicasets
kubectl get pods -o wide
kubectl rollout status deployment/demo-web
```

**Acceptance criteria:**

- The Deployment has one desired and one available replica.
- The Pod reaches `Running` and reports ready.
- The rollout completes successfully.

### Task 5: Access the application

Start port forwarding:

```bash
kubectl port-forward deployment/demo-web 8080:80
```

Keep this command running. Open another terminal and execute:

```bash
curl -I http://localhost:8080   #if aws replace localhost with pubip
curl http://localhost:8080
```

Expected result:

- The HTTP response is successful.
- The response body contains the default NGINX welcome page.

Port forwarding is intended for local testing; it is not a production ingress solution.

### Task 6: Inspect application health

Run:

```bash
kubectl describe deployment demo-web
kubectl describe pods -l app=demo-web
kubectl logs deployment/demo-web
kubectl get events --sort-by=.metadata.creationTimestamp
```

Explain which commands help you inspect resource configuration, container logs, and recent cluster events.

### Task 7: Perform a controlled recovery exercise

Delete the application Pod:

```bash
kubectl get pods
kubectl delete pod -l app=demo-web
```

Observe the result:

```bash
kubectl get pods -w
```

Stop watching with `Ctrl+C` once a replacement Pod is ready.

**Question:** Why does a replacement Pod appear even though you deleted the original Pod?

Explain the role of the Deployment and its ReplicaSet in your notes.

### Task 8: Clean up

Stop the port-forward process with `Ctrl+C`.

Remove the application:

```bash
kubectl delete -f manifests/demo-deployment.yaml
```

Verify that its Deployment is gone:

```bash
kubectl get deployments
```

When you no longer need the cluster, remove only the cluster created for this lab:

```bash
kind delete cluster --name k8s-lab
```

Verify:

```bash
kind get clusters
```

## 6. Verification checklist

- [x] Docker and required tools work.
- [x] Kind cluster was created successfully.
- [x] Kubernetes node reached `Ready`.
- [x] Cluster namespaces and system Pods were inspected.
- [x] Deployment manifest was created and applied.
- [x] NGINX Pod reached `Running` and became ready.
- [x] HTTP response was verified using curl.
- [x] Pod recovery behavior was observed.
- [x] Application and cluster cleanup were completed.
- [x] Notes and evidence were recorded.

## 7. Troubleshooting

See `troubleshooting.md` for diagnostic steps covering:

- Docker daemon unavailable.
- Kind cluster creation failure.
- Node not reaching `Ready`.
- Pod stuck in `Pending`.
- `ImagePullBackOff` or `ErrImagePull`.
- Deployment rollout timeout.
- Port 8080 already in use.
- HTTP connection failure.

Use `kubectl describe`, `kubectl logs`, and `kubectl get events` before attempting random fixes.

## 8. Hints and solutions

Attempt the tasks independently first.

- `hints.md` provides progressively stronger hints.
- `solution.md` contains the reference commands and manifest.

Do not copy the solution until you have attempted the relevant task.

## 9. Evidence and reflection

Record:

- Ubuntu version and CPU architecture.
- Docker, Kind, and kubectl versions.
- Kubernetes version reported by the cluster.
- The commands used to verify success.
- One issue encountered and how you diagnosed it.
- What you learned about Pod replacement and declarative management.

Never commit kubeconfig files, access tokens, credentials, or other secrets.

## 10. Completion criteria

The lab is complete when all checklist items pass and you can explain the difference between a container, Pod, Deployment, ReplicaSet, and node without relying on notes.

**Next lab:** Lab 02 — Cluster Exploration, Namespaces, Events, and Kubernetes Architecture.

