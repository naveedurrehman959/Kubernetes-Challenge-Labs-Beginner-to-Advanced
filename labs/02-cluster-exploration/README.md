# Lab 02: Kubernetes Cluster Exploration and Troubleshooting

**Level:** Beginner
**Track:** DevOps / Site Reliability Engineering (SRE)
**Environment:** Ubuntu + Kind + kubectl
**Estimated time:** 3–4 hours
**Prerequisites:** Lab 01 completed

---

## 1. Lab Overview

Welcome to Lab 02!

In this lab, you will learn how to explore a Kubernetes cluster and troubleshoot a broken application.

Imagine that you are a junior DevOps engineer. Your team reports that an application is not starting. Your responsibility is to investigate the problem, identify its root cause, fix it, and verify that the application is working.

You will deliberately create a broken Kubernetes Deployment and learn how to diagnose it using Kubernetes commands.

**Important:** Do not open `solution.md` until you have attempted the tasks yourself.

## 2. Learning Objectives

By the end of this lab, you should be able to:

- Understand the basic structure of a Kubernetes cluster.
- Check which Kubernetes cluster you are connected to.
- View nodes, namespaces, Pods, and Deployments.
- Understand namespaced and cluster-scoped resources.
- Read Kubernetes events and error messages.
- Troubleshoot a broken Deployment.
- Fix an incorrect container image.
- Verify that an application is running correctly.
- Write a simple incident report, as an SRE would.

## 3. What You Will Build

You will create a namespace named `lab-debug` and deploy an NGINX application with an intentionally incorrect image tag.

\<escape>
Kubernetes cluster
&#x20;   |
&#x20;   └── Namespace: lab-debug
&#x20;         |
&#x20;         └── Deployment: debug-demo
&#x20;                |
&#x20;                └── Pod: NGINX container
&#x20;                       |
&#x20;                       └── Incorrect image tag
\</escape>

Your task is to discover why the application cannot start, fix the problem, and confirm that NGINX responds to HTTP requests.

## 4. Prerequisites

Before starting, make sure you have:

- Ubuntu Linux running.
- Docker installed and running.
- Kind installed.
- kubectl installed.
- A Kind cluster named `k8s-lab`.

If you completed Lab 01 with a different cluster name, use your actual cluster name consistently.

### Step 1: Check kubectl

Open your terminal and run:

```bash
kubectl version --client
```

Expected result: kubectl displays its client version.

### Step 2: Check Kind

```bash
kind get clusters
```

Expected result should include:

```text
k8s-lab
```

### Step 3: Check your active context

```bash
kubectl config current-context
```

For a Kind cluster named `k8s-lab`, the context is commonly:

```text
kind-k8s-lab
```

### Step 4: Check cluster connectivity

```bash
kubectl cluster-info
```

This command checks whether kubectl can communicate with the Kubernetes API server.

**If any prerequisite fails:** Stop and fix your environment before continuing. Do not create resources in a cluster you have not identified.

## 5. Understand the Basic Commands

Before the challenge, learn these common commands.

| Command                      | Purpose                                       |
| ---------------------------- | --------------------------------------------- |
| `kubectl get nodes`          | Lists cluster nodes                           |
| `kubectl get namespaces`     | Lists namespaces                              |
| `kubectl get pods -A`        | Lists Pods across namespaces                  |
| `kubectl describe pod NAME`  | Shows detailed Pod information                |
| `kubectl get events -A`      | Lists events across namespaces                |
| `kubectl get deployments -A` | Lists Deployments across namespaces           |
| `kubectl logs POD_NAME`      | Displays container logs when available        |
| `kubectl apply -f FILE.yaml` | Creates or updates resources from a YAML file |

Replace example names such as `NAME` and `POD_NAME` with the actual resource names.

## 6. Task 1: Explore Your Kubernetes Cluster

Run each command separately.

### 6.1 List available contexts

```bash
kubectl config get-contexts
```

A context tells kubectl which cluster, user, and namespace configuration to use.

### 6.2 Display the active context

```bash
kubectl config current-context
```

**Challenge:** Write down the active context name.

### 6.3 List cluster nodes

```bash
kubectl get nodes -o wide
```

Look at the `STATUS` column.

A healthy local Kind node will normally show:

```text
Ready
```

### 6.4 List namespaces

```bash
kubectl get namespaces
```

You may see namespaces such as:

- `default`
- `kube-system`
- `kube-public`
- `kube-node-lease`

Kubernetes uses namespaces to organize resources within a cluster.

### Your task

Answer these questions in your notes:

1. What is the name of your active context?
2. How many nodes are listed?
3. Are your nodes Ready?
4. What is a Kubernetes namespace?
5. Why should you check your active context before making changes?

## 7. Task 2: Understand Resource Scope

Some Kubernetes resources belong to a namespace, while others exist at cluster level.

### 7.1 List namespaced resources

```bash
kubectl api-resources --namespaced=true
```

Examples include:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets

### 7.2 List cluster-scoped resources

```bash
kubectl api-resources --namespaced=false
```

Examples include:

- Nodes
- Namespaces
- PersistentVolumes
- ClusterRoles

The exact list depends on the Kubernetes APIs installed in your cluster.

### Your task

Write down three namespaced resources and three cluster-scoped resources.

Explain in one or two sentences why resource scope matters when troubleshooting an application.

## 8. Task 3: Create a Troubleshooting Namespace

Create a separate namespace for this lab.

```bash
kubectl create namespace lab-debug
```

Verify it:

```bash
kubectl get namespace lab-debug
```

Expected result: the namespace appears in the output.

Why use a separate namespace? It keeps your practice resources organized and makes cleanup safer.

## 9. Task 4: Create a Broken Deployment

Inside your repository, create this directory structure:

```text
labs/
└── 02-cluster-exploration/
    ├── README.md
    ├── hints.md
    ├── solution.md
    ├── troubleshooting.md
    └── manifests/
        └── broken-deployment.yaml
```

Create the manifest file:

```bash
mkdir -p labs/02-cluster-exploration/manifests
```

Open the file:

```bash
nano labs/02-cluster-exploration/manifests/broken-deployment.yaml
```

Add the following YAML:

```yaml
kube
```

Save the file in nano using `Ctrl+O`, Enter, then `Ctrl+X`.

**Beginner explanation:**

- `apiVersion`: identifies the Kubernetes API used.
- `kind`: specifies the resource type.
- `metadata`: contains the resource name and namespace.
- `replicas: 1`: asks Kubernetes to maintain one Pod.
- `selector`: tells the Deployment which Pods it manages.
- `template`: defines the Pod Kubernetes should create.
- `image`: specifies the container image.
- `containerPort: 80`: documents the port the container is expected to serve.
- `resources`: declares CPU and memory requests and limits.

The image tag is intentionally incorrect. Your job is to find the problem rather than guessing immediately.

### Apply the Deployment

```bash
kubectl apply -f labs/02-cluster-exploration/manifests/broken-deployment.yaml
```

Expected result:

```text
deployment.apps/debug-demo created
```

The command may say `configured` if the resource already exists.

### Inspect the resources

```bash
kubectl -n lab-debug get deployments
```

Then:

```bash
kubectl -n lab-debug get pods
```

The Pod may show an image-pull-related status. That is expected for this exercise.

## 10. Task 5: Troubleshoot the Broken Application

You are now acting as the on-call DevOps engineer.

**Rule:** Gather evidence before changing the configuration.

### Step 1: Inspect the Deployment

```bash
kubectl -n lab-debug get deployment debug-demo
```

Check the desired and available replica counts.

### Step 2: Inspect the Pods

```bash
kubectl -n lab-debug get pods -o wide
```

Look at the `STATUS` column.

Possible image-related errors include:

- `ErrImagePull`
- `ImagePullBackOff`

### Step 3: Describe the Deployment

```bash
kubectl -n lab-debug describe deployment debug-demo
```

Read the output carefully, especially the conditions and events.

### Step 4: Describe the Pod

First, find the actual Pod name:

```bash
kubectl -n lab-debug get pods
```

Then replace `POD_NAME` in the command below:

```bash
kubectl -n lab-debug describe pod POD_NAME
```

Look at the bottom of the output for the `Events` section.

### Step 5: Check event

If your kubectl version supports it, you can also run:

```bash
kubectl events -n lab-debug
```

Events may reveal why Kubernetes cannot retrieve the image.

**Important:** Events are useful diagnostic evidence, but they have limited retention and are not a permanent audit log.

### Your troubleshooting questions

Before moving forward, answer:

1. What status is the Pod showing?
2. What error message appears in the events?
3. Is the problem related to scheduling, image retrieval, or application execution?
4. Which part of the YAML should you investigate?
5. What is the smallest change that could fix the issue?

If you are stuck, open `hints.md`. Keep `solution.md` closed until you have made a genuine attempt.

## 11. Task 6: Fix the Deployment

Once you have identified the root cause, edit the manifest:

```bash
nano labs/02-cluster-exploration/manifests/broken-deployment.yaml
```

Replace the incorrect image:

```yaml
image: nginx:not-a-real-lab-tag
```

with:

```yaml
image: nginx:stable
```

Save the file.

Apply the corrected configuration:

```bash
kubectl apply -f labs/02-cluster-exploration/manifests/broken-deployment.yaml
```

Wait for the Deployment to become available:

```bash
kubectl -n lab-debug rollout status deployment/debug-demo --timeout=120s
```

Expected result:

```text
deployment "debug-demo" successfully rolled out
```

The exact wording may vary slightly.

### Verify the fix

```bash
kubectl -n lab-debug get deployment,pods
```

Check that:

- The Deployment has one available replica.
- The Pod is `Running`.
- The Pod is Ready.

A successful `kubectl apply` alone does not prove that an application is healthy. Always verify the rollout and workload status.

## 12. Task 7: Test the Application

Now verify that NGINX responds to HTTP requests.

### Step 1: Start port-forwarding

```bash
kubectl -n lab-debug port-forward deployment/debug-demo 8081:80
```

Keep this command running in your terminal.

Port-forwarding sends traffic from local port `8081` to port `80` on the selected workload.

### Step 2: Open another terminal

Run:

```bash
curl -I http://127.0.0.1:8081
```

Expected result includes an HTTP success status, usually:

```text
HTTP/1.1 200 OK
```

If you receive a successful HTTP response, your NGINX application is responding.

Stop port-forwarding with `Ctrl+C` when you finish testing.

## 13. Task 8: Write an Incident Report

Create a file named `incident-notes.md` inside:

```text
labs/02-cluster-exploration/
```

Use this template:

```markdown
# Incident Report — Lab 02

## Symptom
What was not working?

## Evidence
Which commands and event messages helped identify the problem?

## Root Cause
What was incorrect in the configuration?

## Resolution
What change fixed the issue?

## Verification
How did you prove the application was working?

## Prevention
What review, automated check, or monitoring could help catch this problem earlier?
```

Write the report in your own words. This is an important SRE habit: record the evidence, cause, resolution, and prevention rather than simply saying “fixed.”

## 14. Task 9: Clean Up Safely

First, confirm your active context:

```bash
kubectl config current-context
```

Confirm that the namespace belongs to this lab:

```bash
kubectl get namespace lab-debug
```

Delete only the lab namespace:

```bash
kubectl delete namespace lab-debug
```

Verify the cleanup:

```bash
kubectl get namespace lab-debug
```

Kubernetes should report that the namespace was not found.

**Do not delete your entire Kind cluster as a shortcut for cleaning up this lab.**

## 15. Common Problems and Troubleshooting

### Problem A: `kubectl` cannot connect to the cluster

Run:

```bash
kind get clusters
kubectl config get-contexts
kubectl config current-context
kubectl cluster-info
```

Check that Docker is running, your Kind cluster exists, and kubectl is using the intended context.

### Problem B: No Pods appear

List Pods across all namespaces:

```bash
kubectl get pods -A
```

Then inspect the lab namespace:

```bash
kubectl -n lab-debug get pods
```

Remember that resources such as Pods are namespaced.

### Problem C: `ImagePullBackOff`

Inspect the Pod:

```bash
kubectl -n lab-debug describe pods
```

Check image spelling, the tag, registry connectivity, and whether authentication is required.

For this lab, investigate the intentionally incorrect image tag.

### Problem D: The Deployment does not become available

```bash
kubectl -n lab-debug get deployment debug-demo
kubectl -n lab-debug get pods
kubectl -n lab-debug describe deployment debug-demo
kubectl -n lab-debug get events
```

Do not assume that the manifest was correct merely because Kubernetes accepted it.

### Problem E: Port-forwarding or curl fails

Check that:

- The Pod is Ready.
- Port-forwarding is still running.
- The local port is `8081`.
- The target container port is `80`.

If local port `8081` is occupied, try:

```bash
kubectl -n lab-debug port-forward deployment/debug-demo 8082:80
```

Then test:

```bash
curl -I http://127.0.0.1:8082
```

## 16. Completion Checklist

Mark each item when you have completed it yourself.

- [ ] Confirmed the Kubernetes context.
- [ ] Checked cluster nodes and namespaces.
- [ ] Compared namespaced and cluster-scoped resources.
- [ ] Created the `lab-debug` namespace.
- [ ] Deployed the intentionally broken application.
- [ ] Investigated the Pod, Deployment, and events.
- [ ] Identified the root cause before reading the solution.
- [ ] Corrected the image tag.
- [ ] Verified the rollout and Pod readiness.
- [ ] Tested NGINX using `curl`.
- [ ] Wrote `incident-notes.md`.
- [ ] Cleaned up the lab namespace.
- [ ] Practised answering the interview questions below.

## 17. DevOps/SRE Interview Practice

Answer these questions without copying from the README:

1. What is the difference between a Pod and a Deployment?
2. What is a Kubernetes namespace?
3. What is the difference between `ErrImagePull` and `CrashLoopBackOff`?
4. What information does `kubectl describe pod` provide?
5. Why should you inspect events during troubleshooting?
6. Why is `kubectl apply` not enough to prove that an application is healthy?
7. Why should you check your active context before running destructive commands?
8. What is the purpose of CPU and memory requests and limits?
9. What does a successful HTTP response from NGINX prove?
10. How would you prevent incorrect image tags from reaching production?

## 18. Suggested 3–4 Hour Practice Schedule

| Time       | Activity                                           |
| ---------- | -------------------------------------------------- |
| 30 minutes | Review cluster architecture and kubectl basics     |
| 60 minutes | Complete cluster and namespace exploration         |
| 60 minutes | Investigate and fix the broken Deployment          |
| 30 minutes | Test the application and troubleshoot any errors   |
| 30 minutes | Write the incident report and clean up             |
| 30 minutes | Practise interview questions and review your notes |

If you finish early, repeat the diagnosis without looking at your previous commands and explain each step aloud.

## 19. GitHub Submission

From the root of your repository, inspect your changes:

```bash
git status
```

After completing the lab and writing your incident report, commit the work:

```bash
git add labs/02-cluster-exploration/
git commit -m "Add Kubernetes lab 02 cluster troubleshooting"
git push
```

Before committing, confirm that the files contain your completed work and that you have not accidentally included secrets or unrelated files.

## Lab Complete!

You have practised an essential DevOps/SRE workflow:

**Observe → Investigate → Identify the root cause → Fix → Verify → Document → Clean up**

Move to the next lab only when you can explain both the commands and the reasoning behind your fix.

