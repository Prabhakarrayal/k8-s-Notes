# Kubernetes Mentorship — Day 1 Revision Notes

> **Purpose:** Detailed, concept-first revision notes for everything actually studied in today's Kubernetes session.
>
> **Lab environment:** Existing Ubuntu VM created in Virtual Machine Manager (VMM/KVM), with Docker already installed.
>
> **Important note:** These notes deliberately distinguish **studied**, **introduced**, and **future** concepts. They do not treat commands or components that we have not yet studied as fully understood.

---

# Table of Contents

1. Learning Goal and Today's Strategy
2. The Big Picture: Linux → Docker → Kubernetes → Cloud
3. Why Kubernetes Exists
4. Docker vs Kubernetes — The Mental Model
5. Containers, Scaling, and Orchestration
6. Desired State and Reconciliation
7. Kubernetes Cluster
8. Control Plane
9. API Server
10. Scheduler
11. Controllers and Reconciliation
12. etcd
13. Worker Node
14. kubelet
15. Container Runtime
16. Pod — First Introduction
17. Deployment — First Introduction
18. ReplicaSet — First Introduction
19. Service — First Introduction
20. Kubernetes and Docker Runtime Relationship
21. Where OpenStack Fits
22. Where Ceph Fits
23. CSI — First Introduction
24. Minikube
25. Why Minikube
26. Minikube vs Production Kubernetes
27. Minikube Driver
28. Our Actual Lab Architecture
29. Lab VM Baseline — Verified
30. Existing Docker Workload — Nextcloud
31. Installing kubectl
32. Installing Minikube
33. Starting the Minikube Cluster
34. kubectl Context
35. First Cluster Verification
36. Understanding `kubectl cluster-info`
37. Understanding `kubectl get nodes`
38. Why the Minikube Node Has the Control-Plane Role
39. Understanding Namespaces — Initial Introduction
40. `kubectl get pods`
41. `kubectl get pods -A`
42. Real System Pods in Our Cluster
43. Mapping System Pods to Architecture
44. CoreDNS
45. kube-proxy
46. kindnet / CNI — Future Topic
47. storage-provisioner — Future Topic
48. Reading Pod Status Columns
49. Command Safety Model
50. Command Breakdown Standard
51. Current Architecture Map
52. Current New Terms Dictionary
53. Current Learning Progress Tracker
54. Troubleshooting Lessons Learned
55. Important Confusing Points
56. Mental Model — End of Day 1
57. Recall Questions
58. Interview-Oriented Questions
59. Practical Next Step
60. Future Topic Parking Lot
61. Final Recall Sheet

---

# 1. Learning Goal and Today's Strategy

The goal of this Kubernetes mentorship is **not** to memorize Kubernetes YAML or copy commands.

The target is to eventually be able to reason about questions such as:

- How is a Kubernetes application deployed?
- What is a Pod?
- Why is a Deployment used?
- What happens when a Pod fails?
- How does Kubernetes know what should be running?
- How does Kubernetes choose a Node?
- How does an application become reachable?
- Why can a Service find Pods?
- What happens when the application is updated?
- Where does persistent storage fit?
- How does Kubernetes connect to storage such as Ceph?
- How would we investigate a Pod that is not running?

Today's session intentionally followed a **fast practical path** while preserving the conceptual foundation needed for later production-oriented work.

## Today's practical path

```text
Existing Ubuntu VM
        │
        ▼
Verify Docker environment
        │
        ▼
Install kubectl
        │
        ▼
Install Minikube
        │
        ▼
Start Minikube using Docker driver
        │
        ▼
Verify Kubernetes cluster
        │
        ▼
Inspect Node
        │
        ▼
Inspect Pods
        │
        ▼
Understand what Minikube created
```

## What we have NOT completed yet

We have not yet properly learned:

```text
Pod lifecycle
Deployment creation
ReplicaSet behavior
Service creation
Application access
YAML
Scaling
Rolling updates
Rollback
Configuration
Secrets
Persistent storage
Kubernetes networking
Troubleshooting failure scenarios
```

Those remain future lessons.

---

# 2. The Big Picture: Linux → Docker → Kubernetes → Cloud

The career-oriented infrastructure path we are using is:

```text
Linux
  │
  ▼
Docker / Container Runtime
  │
  ▼
Kubernetes
  │
  ▼
Cloud / OpenStack
  │
  ├── Compute
  ├── Networking
  └── Storage
        │
        ▼
       Ceph
```

This is not saying that every real production environment must use exactly this stack.

It is the learning path that connects the technologies already being studied.

## Existing environment

The practical environment already includes concepts/tools around:

```text
Linux
OpenStack
Ceph
Docker
Chef
Linux networking
S3-compatible object storage
```

Kubernetes is being added as the orchestration layer.

---

# 3. Why Kubernetes Exists

Docker solves an important problem:

> Run a containerized application.

For example:

```text
Docker
  │
  └── Nginx container
```

That works well for one or a few containers.

But imagine an application becomes larger:

```text
Application
│
├── Web container × 5
├── API container × 5
├── Worker container × 3
├── Database
├── Cache
└── Other services
```

Now many operational questions appear.

### Failure

What if one container stops?

```text
Web container
     X
```

Who notices?

Who starts another one?

### Scaling

What if traffic increases?

```text
1 instance
   ↓
5 instances
```

Who creates the additional instances?

### Scheduling

If multiple machines exist:

```text
Node 1
Node 2
Node 3
```

where should a workload run?

### Updates

How can a new application version be introduced without unnecessarily taking down the whole application?

### Desired state

What if we say:

```text
I want 3 application replicas.
```

but only 2 are actually running?

Something must continuously compare the desired state with reality.

---

# 4. Docker vs Kubernetes — The Mental Model

A useful learning model is:

```text
Docker
  │
  └── Primarily provides container execution/building
```

while:

```text
Kubernetes
  │
  └── Orchestrates and manages containerized workloads
```

Kubernetes deals with concerns such as:

- scheduling
- desired state
- reconciliation
- workload management
- self-healing behavior
- service discovery/networking
- scaling
- updates
- configuration
- storage integration

Kubernetes is therefore **not simply "Docker but bigger."**

---

# 5. Containers, Scaling, and Orchestration

Suppose one container handles an application:

```text
                 Users
                   │
                   ▼
             ┌──────────┐
             │ Container│
             └──────────┘
```

As demand grows:

```text
                 Users
                   │
              ┌────┴────┐
              ▼         ▼
         Container   Container
              │         │
              └────┬────┘
                   ▼
              Application
```

Now something must manage:

- how many copies exist
- where they run
- whether they are alive
- how they are updated
- how users reach them

This is the orchestration problem.

## Orchestration

**Orchestration** means automatically coordinating and managing multiple workloads/components according to defined rules and desired state.

---

# 6. Desired State and Reconciliation

This is one of the most important Kubernetes mental models.

Suppose we tell Kubernetes:

```text
Desired state:
3 application Pods
```

But the actual state is:

```text
Actual state:
2 application Pods
```

Kubernetes controllers continuously observe the state and work toward correcting the difference.

```text
          Desired State
               │
               │
               ▼
        "3 Pods should exist"
               │
               ▼
        Kubernetes observes
               │
               ▼
          Actual State
               │
               │
               ▼
        "Only 2 exist"
               │
               ▼
          Reconciliation
               │
               ▼
       Create/restore another
               │
               ▼
          Actual = 3
```

## Desired State

What we declare we want.

## Actual State

What is currently happening in the cluster.

## Reconciliation

The process of comparing the current/actual state with the desired state and taking action to move the system toward the desired state.

This concept will become much more important when we study Deployments and controllers.

---

# 7. Kubernetes Cluster

A **Kubernetes cluster** is the complete Kubernetes environment.

At a simplified architectural level:

```text
Kubernetes Cluster
│
├── Control Plane
│
└── Worker Node(s)
```

A cluster can contain:

- management/control components
- nodes that provide compute capacity
- workloads
- networking components
- storage integrations
- other supporting services

Our Minikube cluster is a local learning cluster.

---

# 8. Control Plane

The **Control Plane** is the management/coordination side of Kubernetes.

The major components we studied are:

```text
Control Plane
│
├── API Server
├── Scheduler
├── Controller Manager
└── etcd
```

### High-level responsibility

```text
Control Plane
     │
     ├── receives/serves cluster API requests
     ├── stores cluster state
     ├── schedules Pods
     └── reconciles desired vs actual state
```

The Control Plane should not be thought of as simply "the machine that runs applications."

Its central purpose is managing and coordinating the cluster.

---

# 9. API Server

The **Kubernetes API Server** is the primary communication interface to the Kubernetes cluster.

A simplified flow:

```text
kubectl
   │
   │ HTTPS/API request
   ▼
API Server
   │
   ├── cluster state / resources
   ├── authentication/authorization mechanisms
   └── communication with other components
```

We observed this directly using:

```bash
kubectl cluster-info
```

Our actual output showed:

```text
Kubernetes control plane is running at https://192.168.49.2:8443
```

This verified that `kubectl` could communicate with the Kubernetes control plane.

## Important mental model

Do not think:

```text
kubectl = Kubernetes
```

Instead:

```text
kubectl
   │
   ▼
Kubernetes API
   │
   ▼
Kubernetes cluster
```

`kubectl` is a client.

---

# 10. Scheduler

The **Scheduler** decides which Node should run an unscheduled Pod.

Simplified:

```text
New Pod
   │
   │ not assigned to a Node
   ▼
Scheduler
   │
   ├── evaluates available nodes
   └── selects a suitable node
             │
             ▼
           Node
```

## Critical distinction

The Scheduler is **not** the component responsible for general desired-vs-actual reconciliation.

That is the responsibility of controllers.

### Common confusion

Wrong mental model:

```text
Scheduler = checks desired vs actual
```

Correct basic model:

```text
Scheduler = selects a Node for an unscheduled Pod
```

Controllers handle reconciliation.

---

# 11. Controllers and Reconciliation

A **Controller** watches relevant state and works to move the cluster toward the desired state.

Basic model:

```text
Desired State
      │
      ▼
Controller
      │
      ▼
Observe Actual State
      │
      ├── matches ───────► continue observing
      │
      └── differs
             │
             ▼
         Take action
             │
             ▼
        Reconcile
```

For example, if a workload should have three replicas but only two exist, the appropriate controller can work toward restoring the desired number.

Controllers become especially important when we study:

- Deployments
- ReplicaSets
- self-healing
- rolling updates

---

# 12. etcd

**etcd** is a distributed key-value store used by Kubernetes to persist cluster state.

Simplified model:

```text
Kubernetes
     │
     ▼
 API Server
     │
     ▼
   etcd
     │
     └── persistent cluster state
```

Think of it as an important persistence layer for Kubernetes' cluster configuration/state.

It is not the application's normal database.

For example:

```text
etcd
  → Kubernetes cluster state

MySQL/PostgreSQL
  → application data
```

Those are different responsibilities.

---

# 13. Worker Node

A **Node** is a machine participating in the Kubernetes cluster and providing resources for workloads.

A Node can be:

- physical
- virtual

Simplified worker-side architecture:

```text
Worker Node
│
├── kubelet
├── Container Runtime
├── Networking components
└── Pods
```

The Node provides resources such as:

- CPU
- memory
- storage
- networking capacity

---

# 14. kubelet

The **kubelet** is the Kubernetes node agent.

High-level responsibility:

> Manage Pods assigned to the Node and make sure the node's workload state is carried out.

Simplified:

```text
Control Plane
     │
     │ Pod assignment / desired workload
     ▼
   kubelet
     │
     ▼
Container Runtime
     │
     ▼
Containers
```

We have only **introduced** kubelet today.

Its detailed behavior, Pod lifecycle interactions, probes, and runtime communication remain future topics.

---

# 15. Container Runtime

A **container runtime** is the software responsible for actually running containers.

Our Docker installation showed:

```text
Runtimes:
io.containerd.runc.v2
runc

Default Runtime:
runc
```

Docker itself now uses modern container runtime components internally.

Kubernetes also uses a container runtime through its container runtime interface.

Important distinction:

```text
Kubernetes
     │
     ▼
Container Runtime Interface / runtime integration
     │
     ▼
Container runtime
     │
     ▼
Container
```

### Modern Kubernetes point

Kubernetes does not conceptually require Docker Engine as its runtime.

Common runtimes include implementations based around:

- containerd
- CRI-O

The important learning point is:

> Kubernetes orchestrates workloads; the runtime actually executes containers.

The detailed CRI topic is future.

---

# 16. Pod — First Introduction

A **Pod** is Kubernetes' basic deployable execution unit.

A Pod contains one or more containers.

Simplified:

```text
Pod
│
├── Container
└── optionally more containers
```

For a simple application:

```text
Pod
└── Nginx container
```

The Pod is an important abstraction because Kubernetes schedules and manages Pods rather than treating every container as an independent scheduling object.

## Important

We have **not completed the Pod lesson yet**.

Future Pod topics include:

- Pod lifecycle
- Pod phases
- restart behavior
- IP addressing
- ephemeral nature
- multiple containers in one Pod
- namespaces/isolation
- storage
- resource requests/limits
- probes

---

# 17. Deployment — First Introduction

A **Deployment** is a higher-level Kubernetes workload object used to manage application Pods.

Initial relationship:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ▼
Pods
     │
     ▼
Containers
```

The Deployment concept will later explain:

- desired replica count
- rolling updates
- rollback
- workload replacement
- self-healing through the controller chain

We have only **introduced** Deployment today.

---

# 18. ReplicaSet — First Introduction

A **ReplicaSet** helps maintain a specified number of matching Pods.

Simplified:

```text
ReplicaSet
     │
     ├── Pod
     ├── Pod
     └── Pod
```

If the desired replica count is:

```text
3
```

the ReplicaSet works toward having three matching Pods.

In normal application workflows, we usually interact with a Deployment rather than manually managing ReplicaSets.

Relationship:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ▼
Pods
```

Detailed ReplicaSet behavior is future.

---

# 19. Service — First Introduction

A **Service** provides a stable way to reach a group of Pods.

Pods are replaceable and their IP addresses can change.

Therefore, directly depending on a specific Pod IP is not a good application-access model.

Conceptually:

```text
Client
  │
  ▼
Service
  │
  ├── Pod
  ├── Pod
  └── Pod
```

The Service concept will later cover:

- ClusterIP
- NodePort
- LoadBalancer
- selectors
- service discovery
- Minikube application access

We have only introduced this today.

---

# 20. Kubernetes and Docker Runtime Relationship

A common beginner misconception is:

> "Kubernetes is Docker."

That is incorrect.

A better conceptual model:

```text
Kubernetes
   │
   │ orchestrates
   ▼
Pods / workloads
   │
   ▼
Container Runtime
   │
   ▼
Containers
```

Our VM itself currently has Docker:

```text
Ubuntu VM
   │
   └── Docker Engine
```

Then Minikube uses Docker as its driver to run the local Kubernetes environment.

This does not mean Kubernetes itself is simply a Docker container manager.

---

# 21. Where OpenStack Fits

Our infrastructure mental model is:

```text
Physical Hardware
       │
       ▼
Cloud / OpenStack
       │
       ▼
VM
       │
       ▼
Linux
       │
       ▼
Container Runtime
       │
       ▼
Containers
       │
       ▼
Kubernetes
```

More precisely, Kubernetes may itself run on VMs provided by OpenStack:

```text
OpenStack
│
├── VM / Node 1
├── VM / Node 2
└── VM / Node 3
       │
       ▼
  Kubernetes Cluster
```

This connects directly with the OpenStack work already being studied.

---

# 22. Where Ceph Fits

Ceph is primarily a storage platform.

Kubernetes itself has storage abstractions and interfaces.

A simplified future relationship:

```text
Kubernetes
    │
    ▼
PersistentVolume / PVC / StorageClass
    │
    ▼
CSI
    │
    ▼
Ceph CSI Driver
    │
    ▼
Ceph
```

Ceph can provide different storage models, such as:

```text
Ceph
├── RBD   → block storage
├── CephFS → file storage
└── RGW   → object storage
```

We are deliberately **not mixing Ceph into the basic Pod/Deployment lesson**.

Storage will be introduced later when the Kubernetes core becomes clear.

---

# 23. CSI — First Introduction

**CSI** stands for:

> Container Storage Interface

CSI provides a standard mechanism/interface through which container orchestration systems such as Kubernetes can interact with storage systems.

A **CSI Driver** is the implementation for a particular storage platform.

Conceptually:

```text
Kubernetes
     │
     ▼
   CSI interface
     │
     ▼
Ceph CSI Driver
     │
     ▼
    Ceph
```

Important distinction:

```text
CSI
= standard interface/mechanism

CSI Driver
= implementation connecting Kubernetes to a specific storage system
```

Detailed CSI workflows are future.

---

# 24. Minikube

**Minikube** is a tool for running a local Kubernetes cluster, primarily for learning, development, and testing.

It is not Kubernetes itself.

```text
Minikube
    │
    └── creates/runs a local Kubernetes environment
```

This makes it useful for our mentorship.

---

# 25. Why Minikube

A production Kubernetes environment can require:

- multiple nodes
- networking
- control-plane infrastructure
- storage
- load balancing
- security configuration
- observability

Building all of that just to learn Kubernetes would be unnecessarily heavy.

Minikube lets us run a local cluster.

Our learning flow is therefore:

```text
Laptop / PC
    │
    ▼
Ubuntu VM
    │
    ▼
Minikube
    │
    ▼
Local Kubernetes Cluster
```

---

# 26. Minikube vs Production Kubernetes

Minikube is a **learning/testing environment**, not a statement that production Kubernetes should look exactly the same.

Our environment:

```text
Minikube
│
└── Single local Node
```

A larger production environment might look like:

```text
Kubernetes Cluster
│
├── Control Plane nodes
│
└── Worker nodes
    ├── Node 1
    ├── Node 2
    ├── Node 3
    └── ...
```

Minikube intentionally simplifies the infrastructure.

---

# 27. Minikube Driver

A **driver** is the mechanism Minikube uses to run its local cluster.

We explicitly selected:

```bash
minikube start --driver=docker
```

This tells Minikube to use Docker as the driver.

Our environment therefore becomes:

```text
Physical PC
   │
   ▼
VMM / KVM
   │
   ▼
Ubuntu VM
   │
   ├── Existing Docker workloads
   │
   └── Minikube
        │
        └── Kubernetes cluster
```

Because we are already inside a VM and already have Docker working, using the Docker driver keeps the lab straightforward.

---

# 28. Our Actual Lab Architecture

This is the most useful architecture diagram for today's session.

```text
┌─────────────────────────────────────────────────────────────┐
│                     Physical PC                            │
│                                                             │
│  Ubuntu host                                                 │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │             VMM / KVM Virtual Machine                 │  │
│  │                                                       │  │
│  │  Ubuntu 24.04.4 LTS VM                               │  │
│  │                                                       │  │
│  │  ┌───────────────────┐                                │  │
│  │  │ Docker Engine     │                                │  │
│  │  │ 29.6.1            │                                │  │
│  │  └─────────┬─────────┘                                │  │
│  │            │                                          │  │
│  │     ┌──────┴────────────────────┐                     │  │
│  │     │                           │                     │  │
│  │     ▼                           ▼                     │  │
│  │ Existing Docker            Minikube                  │  │
│  │ workloads                   │                         │  │
│  │     │                       ▼                         │  │
│  │ Nextcloud AIO          Kubernetes Cluster             │  │
│  │                            │                          │  │
│  │                            ▼                          │  │
│  │                       minikube Node                   │  │
│  │                            │                          │  │
│  │              ┌─────────────┴─────────────┐            │  │
│  │              │                           │            │  │
│  │        Control Plane                Workloads        │  │
│  │              │                           │            │  │
│  │        API Server                   Pods             │  │
│  │        Scheduler                                      │  │
│  │        Controllers                                    │  │
│  │        etcd                                            │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

# 29. Lab VM Baseline — Verified

We did not guess the environment. We inspected it.

Command used:

```bash
cat /etc/os-release && uname -m && nproc && free -h && docker --version && docker info && kubectl version --client && minikube version
```

The command stopped when `kubectl` was not installed, so Minikube's version was not initially checked by that combined command.

## Verified OS

```text
Ubuntu 24.04.4 LTS
```

## Architecture

```text
x86_64
```

## CPU

```text
2 CPUs
```

## Memory

```text
Total: 3.8 GiB
Available: 3.3 GiB
```

## Swap

```text
2.2 GiB
```

## Docker

```text
Docker Engine 29.6.1
```

## Docker runtime information

The actual Docker information showed:

```text
Storage Driver: overlayfs
Cgroup Driver: systemd
Cgroup Version: 2
Runtimes: io.containerd.runc.v2, runc
Default Runtime: runc
```

## Kernel

```text
6.8.0-138-generic
```

## Docker root

```text
/var/lib/docker
```

## Important resource observation

The VM has only:

```text
2 vCPU
3.8 GiB RAM
```

This is adequate for a lightweight learning Minikube environment, but it is limited.

Therefore we should avoid unnecessarily heavy workloads.

---

# 30. Existing Docker Workload — Nextcloud

Before creating Kubernetes resources, we inspected the existing Docker environment.

Actual running container:

```text
nextcloud-aio-mastercontainer
```

Image:

```text
ghcr.io/nextcloud-releases/all-in-one:latest
```

Status:

```text
Up ... (healthy)
```

Existing host port mappings included:

```text
80   → container 80
8080 → container 8080
8443 → container 8443
```

## Important safety lesson

We did not stop or delete this container.

It is an existing Docker workload unrelated to our Kubernetes application lab.

### Port conflict concept

Because host port 80 is already used:

```text
Host
 :80
  │
  ▼
Nextcloud
```

we should not blindly configure a second workload to claim the same host port.

This is a practical networking lesson that will matter later when we expose Kubernetes applications.

---

# 31. Installing kubectl

`kubectl` is the Kubernetes command-line client.

We installed the Linux x86-64 binary.

After installation, actual verification showed:

```text
Client Version: v1.37.0
Kustomize Version: v5.8.1
```

## Mental model

```text
kubectl
   │
   │ API communication
   ▼
Kubernetes API Server
   │
   ▼
Kubernetes Cluster
```

`kubectl` is not the cluster.

It is a client used to interact with the cluster.

---

# 32. Installing Minikube

Minikube was installed from its Linux x86-64 release binary.

Actual version:

```text
minikube version: v1.39.0
commit: 7a9f6a841470a207de8cf4bafcccee0969d8ba10
```

## Why the downloaded file was removed

The installation flow was conceptually:

```text
Download binary
     │
     ▼
minikube-linux-amd64
     │
     │ install
     ▼
/usr/local/bin/minikube
     │
     ▼
Executable available system-wide
```

Then:

```bash
rm minikube-linux-amd64
```

removed only the temporary downloaded copy.

It did **not** uninstall Minikube.

## Why use `/usr/local/bin`?

It is a conventional location for locally installed executables that should be available system-wide.

---

# 33. Starting the Minikube Cluster

We ran:

```bash
minikube start --driver=docker
```

## Command breakdown

### `minikube`

Minikube CLI.

### `start`

Creates/starts the local Kubernetes cluster.

### `--driver=docker`

Explicitly selects Docker as the Minikube driver.

## Why specify the driver?

It makes our intended environment explicit.

Without specifying a driver, Minikube can use its configured/default driver behavior.

For this lab we deliberately want:

```text
Ubuntu VM
   │
   ▼
Docker
   │
   ▼
Minikube
```

## Result

The actual output confirmed:

```text
Configuring CNI ...
Verifying Kubernetes components...
Enabled addons:
- storage-provisioner
- default-storageclass

Done!
kubectl is now configured to use "minikube" cluster
and "default" namespace by default
```

This confirms the cluster was successfully established.

---

# 34. kubectl Context

After Minikube started, the output said:

```text
kubectl is now configured to use "minikube" cluster
and "default" namespace by default
```

This means `kubectl` was configured with the information needed to communicate with the Minikube cluster.

A useful conceptual model:

```text
kubectl
   │
   ▼
Current configuration/context
   │
   ▼
minikube cluster
   │
   ▼
default namespace
```

We have not yet studied Kubernetes contexts deeply.

📌 Contexts → future/deeper topic.

---

# 35. First Cluster Verification

We ran:

```bash
kubectl cluster-info
```

Actual output included:

```text
Kubernetes control plane is running at
https://192.168.49.2:8443
```

and:

```text
CoreDNS is running ...
```

This proved that `kubectl` could communicate with the cluster.

## Safety

Read-only.

No resources were modified.

---

# 36. Understanding `kubectl cluster-info`

Breakdown:

```text
kubectl
```

Kubernetes CLI.

```text
cluster-info
```

Displays information about important cluster endpoints/services.

It answered the practical question:

> "Can my kubectl client communicate with the Kubernetes cluster?"

This is useful as a basic connectivity check.

---

# 37. Understanding `kubectl get nodes`

We ran:

```bash
kubectl get nodes
```

Actual output:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   64m   v1.37.0
```

## Breakdown

### `kubectl`

Kubernetes CLI.

### `get`

Retrieve information about a Kubernetes resource.

### `nodes`

Resource type.

So:

```text
kubectl get nodes
```

means:

> Show the Nodes known to this cluster.

## Actual result

There is one Node:

```text
minikube
```

Its status:

```text
Ready
```

Its role:

```text
control-plane
```

Its Kubernetes version:

```text
v1.37.0
```

---

# 38. Why the Minikube Node Has the Control-Plane Role

Our Minikube environment is a single-node learning cluster.

Therefore, the same Node can perform control-plane responsibilities while also providing a place where workloads can run.

Production-style conceptual architecture:

```text
Cluster
│
├── Control Plane
│
└── Worker Nodes
```

Minikube learning architecture:

```text
Cluster
│
└── Single Node
     ├── Control-plane responsibilities
     └── Workload capacity
```

This is a key distinction.

Do not generalize:

> "A Kubernetes cluster always has one node that is both control plane and worker."

That is simply how our local Minikube lab is structured.

---

# 39. Understanding Namespaces — Initial Introduction

We ran:

```bash
kubectl get pods
```

and got:

```text
No resources found in default namespace.
```

This did **not** mean the cluster had no Pods.

It meant:

> No Pods were found in the current `default` namespace.

A **Namespace** provides a logical scope for Kubernetes resources.

Conceptually:

```text
Cluster
│
├── default
│   └── our application resources
│
└── kube-system
    └── Kubernetes system resources
```

We have only introduced namespaces.

📌 Detailed namespace management → future.

---

# 40. `kubectl get pods`

Command:

```bash
kubectl get pods
```

Breakdown:

```text
kubectl → Kubernetes CLI
get     → retrieve resource information
pods    → resource type
```

Because no namespace was specified, kubectl used the current namespace.

In our environment that was:

```text
default
```

Result:

```text
No resources found in default namespace.
```

This was expected because we had not yet deployed our own application.

---

# 41. `kubectl get pods -A`

To see Pods across all namespaces, we used:

```bash
kubectl get pods -A
```

## Breakdown

```text
-A
```

is shorthand for:

```text
--all-namespaces
```

Therefore:

```text
kubectl get pods
```

means:

```text
Pods in current namespace
```

while:

```text
kubectl get pods -A
```

means:

```text
Pods across all namespaces
```

This is an important command distinction.

---

# 42. Real System Pods in Our Cluster

Our actual output was:

```text
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-559f6c778d-vlmzj           1/1     Running   0          84m
kube-system   etcd-minikube                      1/1     Running   0          84m
kube-system   kindnet-5sszv                      1/1     Running   0          84m
kube-system   kube-apiserver-minikube            1/1     Running   0          84m
kube-system   kube-controller-manager-minikube   1/1     Running   0          84m
kube-system   kube-proxy-c6d9x                   1/1     Running   0          84m
kube-system   kube-scheduler-minikube            1/1     Running   0          84m
kube-system   storage-provisioner                1/1     Running   0          84m
```

This was one of the most useful observations of today's session.

---

# 43. Mapping System Pods to Architecture

## Control Plane

```text
kube-apiserver-minikube
        │
        ▼
     API Server
```

```text
kube-scheduler-minikube
        │
        ▼
      Scheduler
```

```text
kube-controller-manager-minikube
        │
        ▼
 Controller Manager
        │
        ▼
    Controllers
```

```text
etcd-minikube
      │
      ▼
    etcd
```

Therefore the control-plane architecture we studied is visible in the actual cluster:

```text
Control Plane
│
├── API Server
├── Scheduler
├── Controller Manager
└── etcd
```

---

# 44. CoreDNS

Actual Pod:

```text
coredns-559f6c778d-vlmzj
```

Namespace:

```text
kube-system
```

Status:

```text
Running
```

CoreDNS provides DNS functionality for Kubernetes service discovery.

Conceptually:

```text
Pod
 │
 │ DNS query
 ▼
CoreDNS
 │
 ▼
Kubernetes Service name
```

We have only introduced this.

📌 Detailed Kubernetes DNS/networking → future.

---

# 45. kube-proxy

Actual Pod:

```text
kube-proxy-c6d9x
```

`kube-proxy` participates in implementing Kubernetes Service networking on the Node.

It becomes particularly relevant when we study:

```text
Service
   │
   ▼
Pod selection
   │
   ▼
Network traffic
```

We have **not** studied its internal networking behavior yet.

📌 Future topic.

---

# 46. kindnet / CNI — Future Topic

Actual Pod:

```text
kindnet-5sszv
```

This is part of the networking setup visible in this Minikube environment.

The command output during startup also said:

```text
Configuring CNI (Container Networking Interface)
```

## CNI

CNI stands for:

> Container Networking Interface

It is an interface/specification used for configuring container networking.

We have **not** studied CNI architecture yet.

Therefore:

📌 Do not treat `kindnet` or CNI as a completed topic.

Future networking lessons will explain:

- Pod networking
- CNI
- Pod IPs
- Service networking
- kube-proxy
- DNS
- Node networking

---

# 47. storage-provisioner — Future Topic

Actual Pod:

```text
storage-provisioner
```

Minikube also enabled:

```text
storage-provisioner
default-storageclass
```

This supports local storage-related behavior in the Minikube learning environment.

We have not yet studied Kubernetes storage objects in detail.

Future topics include:

```text
StorageClass
PersistentVolume
PersistentVolumeClaim
CSI
Ceph CSI
```

---

# 48. Reading Pod Status Columns

Our output had:

```text
READY   STATUS    RESTARTS   AGE
```

## READY

Example:

```text
1/1
```

This indicates that one container is expected/ready in the displayed Pod status.

Basic model:

```text
Pod
└── 1 container
    └── ready
```

We will later study readiness in more detail.

## STATUS

Example:

```text
Running
```

Indicates the Pod is currently in the Running phase.

Do not assume that `Running` alone proves every application-level health condition is perfect.

## RESTARTS

Our system Pods showed:

```text
0
```

No container restarts had been recorded for those Pods during their displayed lifetime.

This field becomes useful during troubleshooting.

## AGE

Example:

```text
84m
```

Approximate age of that resource.

Do not use AGE alone to infer exact creation order.

---

# 49. Command Safety Model

We are using three broad safety categories.

## 🟢 Read-only

Examples:

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods
kubectl get pods -A
docker ps
docker images
docker info
```

These inspect state.

## 🟡 Modifying

Examples:

```bash
minikube start
kubectl apply
kubectl scale
```

These can change cluster state.

We explain the effect before running them.

## 🔴 Potentially destructive

Examples include:

```bash
kubectl delete
docker rm
docker system prune
```

These require extra care and explicit explanation.

The goal is:

> Observe first → understand → modify → verify.

This is especially important because the Kubernetes lab is running inside a VM that already contains a real Docker workload.

---

# 50. Command Breakdown Standard

From this point forward, every important command should be taught with a compact breakdown.

Example:

```bash
kubectl get pods -A
```

### What?

Retrieve Pods.

### `kubectl`

Kubernetes CLI.

### `get`

Retrieve information.

### `pods`

Resource type.

### `-A`

All namespaces.

### Without `-A`

Only the current namespace is queried.

### Safety

Read-only.

### Why are we using it?

To inspect cluster-wide Pod state.

This format should prevent command memorization without understanding.

---

# 51. Current Architecture Map

## Full relevant architecture

```text
KUBERNETES CLUSTER
│
├── CONTROL PLANE
│   │
│   ├── API SERVER                  🟢 Basic
│   │
│   ├── SCHEDULER                  🟢 Basic
│   │
│   ├── CONTROLLER MANAGER          🟡 Introduced
│   │
│   └── etcd                        🟢 Basic
│
└── NODE
    │
    ├── kubelet                     🟡 Introduced
    │
    ├── Container Runtime           🟡 Introduced
    │
    ├── Networking
    │   ├── kube-proxy              📌 Future
    │   └── CNI / kindnet           📌 Future
    │
    └── Pods                        🟡 Introduced
```

## Workload architecture

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods
    │
    ▼
Containers
```

## Access architecture

```text
Client
   │
   ▼
Service
   │
   ▼
Pods
```

Both Deployment and Service are currently only introduced.

---

# 52. Current New Terms Dictionary

| Term | Current understanding |
|---|---|
| Orchestration | Coordinating/managing workloads according to rules and desired state |
| Kubernetes Cluster | Complete Kubernetes environment |
| Node | Machine participating in the cluster |
| Control Plane | Management/coordination side |
| API Server | Primary Kubernetes API/communication interface |
| Scheduler | Selects a Node for an unscheduled Pod |
| Controller | Watches state and reconciles toward desired state |
| Reconciliation | Moving actual state toward desired state |
| etcd | Distributed key-value store used for Kubernetes cluster state |
| Worker Node | Node providing workload execution capacity |
| kubelet | Node agent managing Pods assigned to the Node |
| Container Runtime | Software that actually executes containers |
| Pod | Basic Kubernetes deployable execution unit |
| Replica | One running copy of a workload |
| Desired State | What we want the cluster to maintain |
| Actual State | What is currently happening |
| Minikube | Local Kubernetes cluster tool for learning/testing |
| Driver | Mechanism Minikube uses to run the local cluster |
| kubectl | Kubernetes command-line client |
| CRI | Container Runtime Interface; future topic |
| Single-node cluster | Local cluster where one Node can perform multiple cluster roles |
| Namespace | Logical scope for Kubernetes resources |
| CSI | Container Storage Interface |
| CSI Driver | Implementation connecting Kubernetes to a specific storage system |
| Ceph | Storage platform |
| CoreDNS | Kubernetes DNS/service discovery component |
| kube-proxy | Node component involved in Service networking |
| CNI | Container Networking Interface |
| kindnet | Networking component visible in this Minikube environment |

---

# 53. Current Learning Progress Tracker

## Kubernetes foundation

```text
Why Kubernetes                  🟢 COMPLETED
Cluster                         🟢 COMPLETED
Node                            🟢 COMPLETED
Control Plane                   🟢 COMPLETED
API Server                      🟢 BASIC
Scheduler                       🟢 BASIC
Controllers                     🟡 INTRODUCED
Reconciliation                  🟡 INTRODUCED
etcd                            🟢 BASIC
Worker Node                     🟢 BASIC
kubelet                         🟡 INTRODUCED
Container Runtime               🟡 INTRODUCED
```

## Minikube

```text
What Minikube is                🟢 COMPLETED
Why Minikube                    🟢 COMPLETED
Driver concept                  🟢 INTRODUCED
Docker driver                   🟢 USED
Minikube installation           🟢 COMPLETED
Cluster creation                🟢 COMPLETED
Cluster verification            🟢 COMPLETED
System Pod inspection           🟢 COMPLETED
```

## Core workload objects

```text
Pod                             🟡 INTRODUCED
Deployment                      🟡 INTRODUCED
ReplicaSet                      🟡 INTRODUCED
Service                         🟡 INTRODUCED
Namespace                       🟡 INTRODUCED
```

## Networking

```text
CoreDNS                         🟡 INTRODUCED
CNI                             📌 FUTURE
kindnet                         📌 FUTURE
kube-proxy                     📌 FUTURE
Service networking              📌 FUTURE
Pod networking                  📌 FUTURE
```

## Storage

```text
StorageClass                    📌 FUTURE
PV                              📌 FUTURE
PVC                             📌 FUTURE
CSI                             🟡 INTRODUCED
Ceph CSI                        📌 FUTURE
```

---

# 54. Troubleshooting Lessons Learned

Even before deploying an application, today's session introduced an important troubleshooting workflow.

## Workflow

```text
SYMPTOM
   │
   ▼
OBSERVE
   │
   ▼
FORM HYPOTHESIS
   │
   ▼
VERIFY
   │
   ▼
IDENTIFY ROOT CAUSE
   │
   ▼
FIX
   │
   ▼
VERIFY AGAIN
```

We practiced the first two stages extensively:

```text
kubectl cluster-info
kubectl get nodes
kubectl get pods
kubectl get pods -A
```

These commands let us observe the actual cluster instead of assuming it is healthy.

---

# 55. Important Confusing Points

## Confusion 1 — `kubectl` vs Kubernetes

```text
kubectl ≠ Kubernetes
```

Correct:

```text
kubectl
   │
   ▼
API Server
   │
   ▼
Kubernetes
```

---

## Confusion 2 — Minikube vs Kubernetes

```text
Minikube ≠ Kubernetes itself
```

Minikube is a tool for running a local Kubernetes environment.

---

## Confusion 3 — Scheduler vs Controller

Wrong:

```text
Scheduler = maintains desired state
```

Correct:

```text
Scheduler
→ selects a Node for an unscheduled Pod

Controller
→ reconciles desired vs actual state
```

---

## Confusion 4 — Node vs Cluster

```text
Cluster
   │
   └── Node(s)
```

Cluster = complete environment.

Node = machine participating in that environment.

---

## Confusion 5 — `kubectl get pods` showing nothing

```text
No resources found in default namespace.
```

does not mean:

```text
Kubernetes has no Pods.
```

It means:

```text
No Pods in the current namespace.
```

Use:

```bash
kubectl get pods -A
```

to inspect all namespaces.

---

## Confusion 6 — `-A`

```text
-A
```

means:

```text
--all-namespaces
```

Without it:

```text
current namespace
```

With it:

```text
all namespaces
```

---

## Confusion 7 — Removing the downloaded Minikube file

```bash
rm minikube-linux-amd64
```

does not uninstall Minikube.

It removes the temporary downloaded binary after it has been installed to:

```text
/usr/local/bin/minikube
```

---

## Confusion 8 — Existing Docker container vs Kubernetes

Our Nextcloud container is:

```text
Docker workload
```

It is not a Kubernetes Pod.

The two environments currently coexist in the same Ubuntu VM.

---

## Confusion 9 — Port 80

Nextcloud already uses:

```text
host :80
```

Therefore we should not blindly expose a Kubernetes workload on the same host port.

---

## Confusion 10 — Single-node Minikube

Our Minikube Node has:

```text
ROLES: control-plane
```

This does not mean production Kubernetes normally uses a single control-plane/worker machine.

It is a local Minikube architecture.

---

# 56. Mental Model — End of Day 1

If you remember only one architecture, remember this:

```text
                    KUBERNETES CLUSTER
                           │
             ┌─────────────┴─────────────┐
             │                           │
        CONTROL PLANE                  NODE
             │                           │
      ┌──────┼─────────┐          ┌─────┼─────┐
      │      │         │          │     │     │
     API   Scheduler Controllers  kubelet Runtime
   Server                       Networking
      │                              │
     etcd                            ▼
                               ┌───────────┐
                               │   POD     │
                               │           │
                               │ Container │
                               └───────────┘
```

And the application-management hierarchy we introduced is:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ▼
Pod
     │
     ▼
Container
```

And application access is conceptually:

```text
Client
  │
  ▼
Service
  │
  ▼
Pods
```

And storage eventually connects through:

```text
Pod / Application
       │
       ▼
PVC / StorageClass
       │
       ▼
CSI
       │
       ▼
Ceph CSI Driver
       │
       ▼
Ceph
```

---

# 57. Recall Questions

Try answering these without looking at the answers immediately.

## Q1

What is the difference between a Kubernetes Cluster and a Node?

## Q2

What is the primary communication interface of Kubernetes?

## Q3

What does the Scheduler do?

## Q4

What is the job of a Controller?

## Q5

What is reconciliation?

## Q6

What does etcd store?

## Q7

What does kubelet do at a high level?

## Q8

What is a Pod?

## Q9

Why does Kubernetes need something like a Deployment?

## Q10

Why is a Service useful if Pods already have IP addresses?

## Q11

What is Minikube?

## Q12

Why are we using the Docker driver?

## Q13

Why did:

```bash
kubectl get pods
```

return:

```text
No resources found in default namespace.
```

while:

```bash
kubectl get pods -A
```

showed many Pods?

## Q14

What does `-A` mean?

## Q15

Why is our Minikube Node showing the `control-plane` role?

## Q16

Where does Ceph fit into the Kubernetes architecture?

## Q17

What is CSI?

---

# 58. Interview-Oriented Questions

## Basic

### What is Kubernetes?

Kubernetes is a container orchestration platform that manages containerized workloads using concepts such as desired state, scheduling, reconciliation, networking, scaling, and workload management.

### What is a Kubernetes cluster?

The complete Kubernetes environment containing its management/control components and Nodes.

### What is a Node?

A physical or virtual machine participating in the Kubernetes cluster and providing resources for workloads.

### What is a Control Plane?

The management and coordination side of Kubernetes.

### What does the Scheduler do?

It selects a suitable Node for an unscheduled Pod.

### What does etcd do?

It persistently stores Kubernetes cluster state in a distributed key-value store.

### What does kubelet do?

It is the node agent responsible for managing Pods assigned to that Node.

### What is Minikube?

A tool for running a local Kubernetes cluster for learning, development, and testing.

---

# 59. Practical Next Step

We have now reached the point where the cluster is ready.

The next practical workflow is:

```text
                 Kubernetes Cluster
                        │
                        ▼
                    Create
                   Deployment
                        │
                        ▼
                    ReplicaSet
                        │
                        ▼
                      Pod
                        │
                        ▼
                   Nginx
                        │
                        ▼
                     Service
                        │
                        ▼
                    Access App
```

The next major lesson should be:

# Pod vs Deployment

We should first understand:

> Why not simply create an Nginx Pod?

Then:

> Why use a Deployment?

Then:

> What does Kubernetes create when we create a Deployment?

Only after that should we move toward Service and application access.

This keeps the practical path fast without turning the lesson into command memorization.

---

# 60. Future Topic Parking Lot

These topics have been intentionally parked rather than skipped permanently.

## Kubernetes Architecture

- Detailed controller behavior
- API Server internals
- etcd internals
- kubelet internals
- control-plane communication

## Pods

- lifecycle
- phases
- restart behavior
- Pod IPs
- ephemeral nature
- multi-container Pods
- init containers
- resource requests/limits
- probes

## Workloads

- Deployment
- ReplicaSet
- StatefulSet
- DaemonSet
- Job
- CronJob

## Networking

- CNI
- kindnet
- Pod networking
- kube-proxy
- Services
- ClusterIP
- NodePort
- LoadBalancer
- DNS
- Ingress
- NetworkPolicy

## Configuration

- ConfigMap
- Secret
- environment variables
- configuration injection

## Storage

- StorageClass
- PersistentVolume
- PersistentVolumeClaim
- CSI
- Ceph CSI
- RBD
- CephFS

## Scheduling

- requests/limits
- affinity
- anti-affinity
- taints
- tolerations
- topology

## Operations

- logs
- describe
- exec
- events
- troubleshooting
- CrashLoopBackOff
- ImagePullBackOff
- Pending
- readiness failures

## Production foundations

- RBAC
- security
- observability
- metrics
- logging
- production cluster architecture
- high availability
- backup/recovery
- upgrade strategy

---

# 61. Final Recall Sheet

## Kubernetes

```text
Kubernetes
= container orchestration platform
```

## Cluster

```text
Cluster
= complete Kubernetes environment
```

## Node

```text
Node
= machine participating in the cluster
```

## Control Plane

```text
Control Plane
= management/coordination side
```

## API Server

```text
API Server
= primary Kubernetes API/communication interface
```

## Scheduler

```text
Scheduler
= selects Node for unscheduled Pod
```

## Controller

```text
Controller
= observes state and reconciles toward desired state
```

## Reconciliation

```text
Desired State
      ↓
Compare
      ↓
Actual State
      ↓
Correct difference
```

## etcd

```text
etcd
= distributed key-value store for Kubernetes cluster state
```

## kubelet

```text
kubelet
= Node agent managing assigned Pods
```

## Container Runtime

```text
Runtime
= actually executes containers
```

## Pod

```text
Pod
= basic Kubernetes deployable execution unit
```

## Deployment

```text
Deployment
      ↓
ReplicaSet
      ↓
Pods
```

## Service

```text
Service
      ↓
stable access to a group of Pods
```

## Minikube

```text
Minikube
= local Kubernetes cluster tool
```

## kubectl

```text
kubectl
= Kubernetes CLI
```

## Namespace

```text
Namespace
= logical scope for Kubernetes resources
```

## CSI

```text
CSI
= Container Storage Interface
```

## Ceph connection

```text
Kubernetes
   ↓
Storage abstractions
   ↓
CSI
   ↓
Ceph CSI Driver
   ↓
Ceph
```

---

# End of Day 1

The most important achievement today was not installing Minikube.

It was moving from:

```text
"I know some Kubernetes terms"
```

to:

```text
"I have a real Kubernetes cluster running,
I can communicate with its API,
I can see its Node,
I can inspect its Pods,
and I can map those real Pods back to
the Control Plane architecture."
```

The next session should build directly on this cluster and move into:

```text
POD
  ↓
DEPLOYMENT
  ↓
REPLICASET
  ↓
SERVICE
  ↓
ACCESS APPLICATION
```

while continuously connecting each practical command back to the architecture.
