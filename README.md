# Kubernetes Challenge Labs: Beginner to Advanced

A hands-on Kubernetes learning path designed to build practical skills
for **DevOps, SRE, and platform engineering**---from your first Pod to
GitOps-based delivery, security, observability, and production-style
operations.

The learning path is organized into **60 challenge labs across six
levels**. Each lab should include a goal, practical tasks, verification
steps, a safe failure scenario, cleanup instructions, and notes about
what you learned.

> **Learning principle:** don't just make the YAML work. Learn to
> explain what Kubernetes is doing, prove it with commands and events,
> troubleshoot a controlled failure, and document the recovery.

## Goals

By completing this repository, you will practice how to:

-   Deploy, scale, update, and troubleshoot containerized applications.
-   Work with Kubernetes workloads, networking, storage, configuration,
    and identity.
-   Package applications with Helm and Kustomize.
-   Build CI pipelines and deploy through GitOps with Argo CD.
-   Apply workload security, supply-chain checks, and policy controls.
-   Monitor applications, define basic SLOs, and practice incident
    recovery.
-   Build a portfolio-ready Kubernetes project with reproducible
    instructions.

## Learning roadmap

  ------------------------------------------------------------------------
  Level                                         Labs Focus
  --------------------- ---------------------------- ---------------------
  1\. Kubernetes                              01--10 Cluster architecture,
  fundamentals                                       Pods, Deployments,
                                                     labels, namespaces,
                                                     Services, ConfigMaps,
                                                     Secrets, probes,
                                                     resource
                                                     requests/limits

  2\. Networking,                             11--20 DNS, Ingress, Gateway
  storage, identity                                  API, NetworkPolicy,
                                                     PV/PVC, StatefulSets,
                                                     Jobs, RBAC, sidecars,
                                                     scheduling

  3\. Workloads and                           21--30 Rollouts, rollback,
  packaging                                          blue-green/canary,
                                                     Helm, Kustomize,
                                                     CRDs, operators,
                                                     admission policy,
                                                     quotas

  4\. CI/CD and GitOps                        31--40 Image publishing,
                                                     Jenkins, manifest
                                                     validation, Argo CD,
                                                     environment
                                                     promotion,
                                                     progressive delivery,
                                                     SBOMs, secret
                                                     workflows

  5\. Security and                            41--50 Pod Security
  reliability                                        Standards, container
                                                     hardening, scanning,
                                                     API access,
                                                     troubleshooting,
                                                     availability,
                                                     backup/restore,
                                                     incident reviews

  6\. Production-style                        51--60 Metrics, logs,
  operations                                         tracing, SLOs,
                                                     HPA/VPA, cluster
                                                     autoscaling,
                                                     upgrades, platform
                                                     engineering, capstone
  ------------------------------------------------------------------------

See the individual lab directories for detailed challenges as they are
added.

## Suggested lab environment

Start locally to reduce cost and make experiments repeatable.

-   **Docker** --- build and run containers.
-   **kind** *or* **minikube** --- run a local Kubernetes cluster. Pick
    one initially.
-   **kubectl** --- inspect and manage Kubernetes resources.
-   **Git** --- track manifests, scripts, and notes.
-   **Helm and Kustomize** --- add these when you reach the packaging
    level.
-   **Jenkins and Argo CD** --- introduce these in the CI/CD and GitOps
    level.
-   **Prometheus, Grafana, and OpenTelemetry** --- introduce these in
    the observability level.
-   **Trivy or comparable maintained tooling** --- image and dependency
    scanning.

Use the official installation instructions for your operating system and
check tool compatibility with your Kubernetes version.

### Check your tools

``` bash
docker --version
kubectl version --client
kind version       # if you choose kind
minikube version   # if you choose minikube
git --version
```

Only run the command for the local cluster tool you chose. Helm can be
checked later:

``` bash
helm version
```

### Create a kind cluster

After installing Docker, kind, and kubectl:

``` bash
kind create cluster --name k8s-challenge
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods -A
```

Confirm that `kubectl` is using the intended context before making
changes:

``` bash
kubectl config current-context
kubectl config get-contexts
```

To delete this lab cluster when you are finished:

``` bash
kind delete cluster --name k8s-challenge
```

This removes the cluster and its local workloads. Save any evidence or
data you need before deleting it.

## Repository structure

A suggested structure for this learning repository:

``` text
kubernetes-challenge-labs/
├── README.md
├── labs/
│   ├── 01-cluster-first-look/
│   ├── 02-pods/
│   ├── 03-deployments/
│   └── .../
├── app/
│   ├── app.py
│   ├── Dockerfile
│   └── tests/
├── environments/
│   ├── dev/
│   └── staging/
├── charts/
├── gitops/
├── policies/
├── dashboards/
└── evidence/
```

Keep each lab self-contained where practical. The root README explains
the overall path; each lab's README should explain how to perform that
challenge.

## Standard workflow for every lab

1.  **Read:** understand the problem, prerequisites, and expected
    outcome.
2.  **Plan:** identify the resources and commands you expect to need.
3.  **Build:** create declarative YAML, scripts, or chart templates.
4.  **Verify:** collect evidence using `kubectl get`, `describe`,
    `logs`, events, and application tests.
5.  **Break safely:** introduce only the failure described by the lab,
    in a disposable environment.
6.  **Recover:** diagnose the cause from evidence and restore the
    desired state.
7.  **Document:** record what happened, what fixed it, and why.
8.  **Clean up:** remove temporary resources and cloud resources when
    applicable.

### Suggested lab deliverables

Each lab should contain:

-   `README.md` --- objective, prerequisites, tasks, verification,
    troubleshooting, and cleanup.
-   `manifests/` --- Kubernetes YAML, separated by resource where
    helpful.
-   `evidence/` --- screenshots or command output with secrets removed.
-   `notes.md` --- explanation, mistakes, root cause, and lessons
    learned.

## Main project

The labs will progressively improve a small Flask service with `/` and
`/health` endpoints. The final capstone will turn it into a
production-style delivery workflow with:

-   A tested, versioned container image.
-   Kubernetes Deployment, Service, configuration, probes, and resource
    settings.
-   Ingress or Gateway API routing, where supported.
-   Helm or Kustomize for repeatable deployment configuration.
-   Jenkins CI for tests, validation, scanning, and image publishing.
-   Argo CD GitOps deployment from version-controlled desired state.
-   Least-privilege access, container hardening, and supply-chain
    checks.
-   Metrics, dashboards, logs, alerting, and a basic SLO.
-   Rollback, troubleshooting, backup/restore notes, and a clean-cluster
    reproduction test.

The capstone is a learning project, not a claim of production
experience. Document what you tested and what remains out of scope.

## How progress is measured

Score each category from 0 to 2:

-   **0 --- Not yet:** incomplete or unable to explain the concept.
-   **1 --- Working:** successful with help, or without failure/recovery
    evidence.
-   **2 --- Demonstrated:** reproducible, verified, documented, and
    safely tested.

Categories: fundamentals, workloads, networking, storage, security,
packaging, CI/CD, GitOps, observability, reliability, troubleshooting,
documentation. Maximum: **24 points**.

A score is a self-assessment of lab evidence, not a certification or
substitute for production experience.

## Safety and good practices

-   Use a disposable local cluster for destructive exercises.
-   Never commit passwords, access tokens, private keys, kubeconfig
    files, or real credentials.
-   Kubernetes Secret values are commonly base64-encoded; base64 is
    **not encryption**.
-   Check the active context and namespace before changing or deleting
    resources.
-   Avoid broad delete commands until you have verified their scope.
-   Pin versions for reproducible projects and consult compatibility
    documentation.
-   Newer APIs, controllers, and features can have version-specific
    requirements. Check maturity and support before using them.
-   Cloud-based labs can incur charges. Set budgets, clean up resources,
    and verify that resources were removed.

## Official learning resources

-   [Kubernetes documentation](https://kubernetes.io/docs/)
-   [Kubernetes concepts](https://kubernetes.io/docs/concepts/)
-   [Kubernetes Basics
    tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
-   [kind quick start](https://kind.sigs.k8s.io/docs/user/quick-start/)
-   [Helm documentation](https://helm.sh/docs/)
-   [Argo CD documentation](https://argo-cd.readthedocs.io/)
-   [Prometheus documentation](https://prometheus.io/docs/)
-   [OpenTelemetry documentation](https://opentelemetry.io/docs/)

## Start here

**Begin with Lab 01 --- Inspect your cluster.** Create a local cluster,
inspect nodes and namespaces, review system Pods, and draw a basic
control-plane/worker-node diagram. Do not move on until you can explain
the role of the API server, scheduler, controller manager, kubelet, and
container runtime.

Good luck, and keep your evidence and notes in Git as you progress.
