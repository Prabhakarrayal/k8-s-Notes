# Kubernetes Mentorship — Day 1 Revision Notes

## Concept-first revision rule

These notes are designed so that reading a concept later should rebuild the mental model, not merely remind you of a definition.

For each major component, ask: 

```text
What is it?
   ↓
Where does it fit?
   ↓
Why does Kubernetes need it?
   ↓
What does it do?
   ↓
How does it interact with the other components?
```

For example, do not remember only:

```text
API Server = API
```

Remember:

```text
API Server
= Control Plane component
= central Kubernetes API/communication surface
= receives/serves requests from clients and components
= provides access to cluster resources/state
```

A topic marked **INTRODUCED** means we understand its role but have not completed its deeper lesson. A **FUTURE** topic is intentionally parked for later.


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

The **Kubernetes API Server is a Control Plane component that provides the main communication interface between clients and the Kubernetes cluster.**

In simple words:

> **The API Server is the front door of the Kubernetes Control Plane.**

When we use a tool such as `kubectl`, the request is sent to the API Server. Kubernetes components also use the Kubernetes API to interact with cluster resources.

```text
User
  │
  │ kubectl command
  ▼
kubectl
  │
  │ HTTPS/API request
  ▼
Kubernetes API Server
  │
  ├── Kubernetes resources
  ├── communication with Control Plane components
  └── persistent cluster state
             │
             ▼
            etcd
```

### Why is the API Server needed?

Without a common API, clients and internal components would need many direct communication paths:

```text
kubectl ─────► Scheduler
kubectl ─────► etcd
kubectl ─────► Controller
Scheduler ───► etcd
Controller ──► etcd
```

Kubernetes instead provides a central API surface:

```text
                  API Server
                /     |      \
               /      |       \
          kubectl  Scheduler  Controllers
```

This gives Kubernetes a consistent way to work with cluster resources.

### Example: `kubectl get nodes`

When we run:

```bash
kubectl get nodes
```

the simplified flow is:

```text
You
 │
 ▼
kubectl
 │
 │ API request
 ▼
API Server
 │
 ▼
Kubernetes cluster state
 │
 ▼
Response
 │
 ▼
kubectl displays result
```

### Our real cluster

We verified this with:

```bash
kubectl cluster-info
```

and our actual cluster reported:

```text
Kubernetes control plane is running at https://192.168.49.2:8443
```

So `kubectl` successfully reached the Kubernetes Control Plane.

### Important distinction

```text
kubectl ≠ API Server
kubectl ≠ Kubernetes
```

Instead:

```text
kubectl
   │
   ▼
API Server
   │
   ▼
Kubernetes cluster
```

`kubectl` is the **client**.

The API Server is the **Kubernetes Control Plane component that exposes the main API used to communicate with the cluster**.
# 10. Scheduler

The **Kubernetes Scheduler is a Control Plane component that decides which Node should run a Pod that has not yet been assigned to a Node.**

In simple words:

> **The Scheduler answers: "This Pod needs to run somewhere. Which Node should it run on?"**

```text
Pod created
   │
   │ no Node selected yet
   ▼
Scheduler
   │
   │ evaluates available Nodes
   ▼
Suitable Node selected
   │
   ▼
Pod is assigned to that Node
```

### Why is the Scheduler needed?

Suppose a cluster has:

```text
Node 1
Node 2
Node 3
```

and a new Pod needs to run.

Something must make the placement decision:

```text
             New Pod
                │
                ▼
           Scheduler
          /    |    \
         /     |     \
     Node 1  Node 2  Node 3
```

The Scheduler performs that placement decision.

### Scheduler vs Controller — important correction

A common confusion is:

```text
Scheduler = maintains desired state
```

That is not the correct mental model.

Instead:

```text
Scheduler
→ selects a Node for an unscheduled Pod

Controller
→ watches state and works toward desired state
```

For example:

```text
Desired = 3 Pods
Actual  = 2 Pods
```

The controller/reconciliation system works toward restoring the missing replica.

The Scheduler becomes relevant when a newly created Pod needs a Node.

### Our real cluster

We saw:

```text
kube-system   kube-scheduler-minikube
```

so the Scheduler is actually running in our Minikube Control Plane.
# 11. Controllers and Reconciliation

A **Kubernetes Controller is a component that watches the state of resources and takes action when the actual state does not match the desired state.**

In simple words:

> **A Controller continuously works to make the real cluster match what we asked Kubernetes to maintain.**

### Desired state vs actual state

Suppose we declare:

```text
Desired state:
3 application Pods
```

but the cluster currently has:

```text
Actual state:
2 application Pods
```

A controller observes the difference:

```text
Desired: 3
Actual:  2
         │
         ▼
   Controller notices
         │
         ▼
   Reconciliation
         │
         ▼
  Work toward 3 Pods
```

### What is reconciliation?

**Reconciliation is the process of comparing the desired state with the actual state and taking action to reduce the difference.**

```text
Desired State
     │
     │ compare
     ▼
Actual State
     │
     ▼
Difference?
  │        │
 No       Yes
  │        │
  ▼        ▼
Keep      Correct
watching  difference
```

### Example

If a workload should have three replicas and one Pod disappears:

```text
Before:
Pod 1
Pod 2
Pod 3

Failure:
Pod 2 disappears

Actual:
Pod 1
Pod 3
```

The appropriate controller can work toward:

```text
Pod 1
Pod 3
Pod 4
```

This is the basic idea behind Kubernetes self-healing behavior.

### Controller vs Scheduler

Remember:

```text
Controller
→ "What should exist and what state should be maintained?"

Scheduler
→ "Which Node should this unscheduled Pod run on?"
```

Controllers become especially important when we study Deployments and ReplicaSets.
# 12. etcd

**etcd is a distributed key-value store used by Kubernetes to persist important Kubernetes cluster state.**

In simple words:

> **etcd is a persistent storage system for Kubernetes' cluster state and configuration information.**

A simplified relationship is:

```text
kubectl / clients
      │
      ▼
  API Server
      │
      ▼
     etcd
      │
      ▼
Persistent Kubernetes cluster state
```

### Why does Kubernetes need etcd?

Kubernetes manages many resources and needs persistent information about the cluster.

Examples include:

```text
Deployments
Services
Namespaces
Pod configuration
ConfigMaps
...
```

The Control Plane needs this persistent cluster state to operate correctly.

### Important distinction

etcd is **not normally the database for your application's business data**.

```text
Kubernetes cluster state
        │
        ▼
       etcd

Application business data
        │
        ▼
MySQL / PostgreSQL / another database
```

These have different responsibilities.

### Our real cluster

We saw:

```text
kube-system   etcd-minikube
```

This is the actual etcd component running in our Minikube Control Plane.
# 13. Node / Worker Node

A **Kubernetes Node is a physical or virtual machine that participates in the cluster and provides compute resources on which workloads can run.**

In simple words:

> **A Node is a machine that Kubernetes can use to run Pods.**

A Node provides resources such as:

- CPU
- memory
- storage
- networking capacity

A simplified Node-side architecture is:

```text
Kubernetes Node
│
├── kubelet
├── Container Runtime
├── Networking components
└── Pods
    ├── Container(s)
    └── Container(s)
```

### Cluster vs Node

```text
Cluster
= complete Kubernetes environment

Node
= one machine participating in that cluster
```

Therefore:

```text
Cluster
│
├── Node 1
├── Node 2
└── Node 3
```

is possible.

Our Minikube environment is a local single-node cluster:

```text
Minikube Cluster
       │
       └── minikube Node
```

That Node currently has the `control-plane` role.

Do not generalize the Minikube single-node layout to every production Kubernetes cluster.
# 14. kubelet

The **kubelet is the Kubernetes agent that runs on a Node and is responsible for managing the Pods assigned to that Node.**

In simple words:

> **The kubelet is the Node-side Kubernetes agent that helps ensure the workloads assigned to the Node are actually managed on that machine.**

A simplified relationship is:

```text
Control Plane
      │
      │ workload / Pod information
      ▼
    kubelet
      │
      ▼
Container Runtime
      │
      ▼
Containers
```

The kubelet does not replace the container runtime.

```text
kubelet
   │
   ▼
container runtime
   │
   ▼
container
```

We have only introduced kubelet today. Detailed Pod lifecycle, health checks, probes, and runtime interaction are future topics.
# 15. Container Runtime

A **container runtime is the software responsible for actually creating and running containers on a machine.**

In simple words:

> **Kubernetes manages/orchestrates workloads, while the container runtime is the component that actually executes the containers.**

A simplified relationship is:

```text
Kubernetes
    │
    ▼
Node / kubelet
    │
    ▼
Container Runtime
    │
    ▼
Container
    │
    ▼
Application Process
```

### Our Docker environment

Our actual `docker info` output showed:

```text
Runtimes:
io.containerd.runc.v2
runc

Default Runtime:
runc
```

This shows that Docker is using modern runtime components underneath its interface.

### Important Kubernetes point

Modern Kubernetes does **not** require Docker Engine as its container runtime.

Common runtime technologies include:

```text
containerd
CRI-O
```

The detailed **CRI (Container Runtime Interface)** topic is future.

For now remember:

```text
Kubernetes
→ orchestrates workloads

Container Runtime
→ actually executes containers
```
# 16. Pod — First Introduction

A **Pod is the smallest deployable unit in Kubernetes and represents one or more containers that are managed together as a single workload unit.**

In simple words:

> **A Pod is the Kubernetes unit in which containers run together.**

For a simple application:

```text
Pod
└── Nginx container
```

A Pod can also contain more than one container:

```text
Pod
├── Main application container
└── Supporting container
```

### Why does Kubernetes use Pods?

Kubernetes needs a unit that it can schedule and manage as a workload.

Therefore:

```text
Kubernetes
    │
    ▼
   Pod
    │
    ├── Container A
    └── Container B
```

### Important distinction

Do not think:

```text
Pod = Container
```

Instead:

```text
Pod
  └── one or more containers
```

For many simple applications:

```text
1 Pod
└── 1 container
```

is common.

### Current learning status

We have only introduced Pods today.

The complete Pod lesson will later cover lifecycle, phases, restart behavior, Pod IPs, ephemeral behavior, multiple containers, storage, resources, and probes.
# 17. Deployment — First Introduction

A **Deployment is a Kubernetes workload object that provides a higher-level way to manage application Pods and maintain the desired version and number of those Pods.**

In simple words:

> **A Deployment tells Kubernetes how an application workload should be maintained instead of making us manage individual Pods manually.**

The basic relationship is:

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

For example, if we eventually declare:

```text
replicas: 3
```

the Deployment/ReplicaSet controller chain works toward maintaining three matching Pods.

Deployments are also used for application lifecycle operations such as:

- rolling updates
- revision history
- rollback
- replacement of Pods

### Why not simply create a Pod?

A directly created Pod is a workload unit, but application workloads usually need higher-level management.

A Deployment provides a desired-state object from which Kubernetes can manage the application's Pods.

We will demonstrate this practically in the next lesson.

### Current status

Deployment is **introduced, not completed**. We have not yet created one in the lab.
# 18. ReplicaSet — First Introduction

A **ReplicaSet is a Kubernetes workload controller whose job is to maintain a specified number of matching Pod replicas.**

In simple words:

> **A ReplicaSet helps make sure the required number of matching Pods exists.**

For example:

```text
Desired replicas = 3
```

The ReplicaSet works toward:

```text
ReplicaSet
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

If one matching Pod disappears, the controller can work toward restoring the desired number.

The relationship we introduced is:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ▼
Pods
```

### Why do we usually use a Deployment?

A Deployment provides higher-level application lifecycle management such as version updates and rollback.

So the common application model is:

```text
We manage:
Deployment

Deployment manages:
ReplicaSet

ReplicaSet manages:
Pods
```

Detailed ReplicaSet selectors, ownership, revisions, and controller behavior are future topics.
# 19. Service — First Introduction

A **Kubernetes Service is an object that provides a stable network endpoint for reaching a group of Pods.**

In simple words:

> **A Service gives clients a stable way to access application Pods even though individual Pods can be replaced and their IP addresses can change.**

Suppose:

```text
Pod A → 10.x.x.1
Pod B → 10.x.x.2
Pod C → 10.x.x.3
```

If Pod B is replaced, its IP can change.

A client should not have to change its configuration every time a Pod changes.

Instead:

```text
Client
  │
  ▼
Service
  │
  ├── Pod A
  ├── Pod B
  └── Pod C
```

The Service provides the stable access abstraction while Kubernetes handles the relationship to the matching Pods.

### How does a Service know which Pods belong to it?

Kubernetes commonly uses **labels and selectors** for this relationship.

We have not studied labels/selectors in depth yet.

📌 Labels/selectors → upcoming topic.

### Service types we will later study

```text
ClusterIP
NodePort
LoadBalancer
```

We will connect Services to DNS, kube-proxy, Pod networking, and Minikube application access.

Today, Service is only an introduction.
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

**CSI stands for Container Storage Interface.**

CSI is a standard interface that allows Kubernetes to communicate with external storage systems through storage-specific CSI drivers.

In simple words:

> **CSI is the standard mechanism Kubernetes can use to request and manage storage from an external storage platform.**

The important distinction is:

```text
CSI
= standard interface

CSI Driver
= software implementation for a specific storage system
```

For example, when Kubernetes eventually needs persistent storage from Ceph:

```text
Kubernetes
     │
     ▼
Kubernetes Storage Objects
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

The CSI driver acts as the bridge between Kubernetes' storage requests and the specific storage platform.

### Why use a standard interface?

Kubernetes can work with many storage platforms. A standard interface prevents Kubernetes from needing completely different storage logic for every storage vendor/platform.

```text
                    Kubernetes
                        │
                  Standard CSI
                   Interface
              ┌─────────┼─────────┐
              ▼         ▼         ▼
          Ceph CSI   Other CSI   Other CSI
              │         │         │
              ▼         ▼         ▼
            Ceph    Storage B   Storage C
```

We have only introduced CSI. Detailed storage objects, CSI workflows, and Ceph CSI configuration are future topics.
# 24. Minikube

**Minikube is a tool that creates and runs a local Kubernetes cluster so that we can learn, develop, and test Kubernetes workloads without building a full production-style cluster.**

In simple words:

> **Minikube gives us a convenient local Kubernetes environment.**

It is important not to confuse the tool with the platform:

```text
Minikube
    │
    └── creates/runs a local environment
              │
              ▼
        Kubernetes Cluster
```

Therefore:

```text
Minikube ≠ Kubernetes
```

For our lab, it lets us practice:

```text
kubectl
   ↓
Cluster
   ↓
Pods
   ↓
Deployments
   ↓
Services
```

without building a production-style multi-node cluster.
# 25. Why Minikube

A production Kubernetes environment can involve multiple machines, networking, storage, security, load balancing, and other infrastructure.

Building all of that just to learn Kubernetes would be unnecessarily complicated.

Minikube gives us a smaller local environment where we can practice the same core Kubernetes concepts.

Our learning path is:

```text
Physical PC
    │
    ▼
Ubuntu VM
    │
    ▼
Minikube
    │
    ▼
Local Kubernetes Cluster
    │
    ▼
Pods / Deployments / Services
```

The key benefit is that we can practice the Kubernetes control model while keeping the environment manageable.
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

A **Minikube driver is the mechanism Minikube uses to create/run its local Kubernetes environment.**

In our lab we explicitly selected:

```bash
minikube start --driver=docker
```

The important part is:

```text
--driver=docker
```

which tells Minikube:

> **Use Docker as the underlying driver for this local Kubernetes environment.**

Our architecture is:

```text
Physical PC
   │
   ▼
VMM / KVM
   │
   ▼
Ubuntu VM
   │
   └── Docker Engine
         │
         └── Minikube
               │
               └── Kubernetes Cluster
```

### Why did we explicitly specify Docker?

We had already verified that Docker was installed and working in the VM. We therefore made our intended driver explicit instead of relying on implicit driver selection/configuration.

### Does this create another traditional VM?

With the Docker driver, we use Docker as the underlying driver rather than intentionally creating another traditional VM layer.

That gives us:

```text
Physical PC
   ↓
VMM/KVM
   ↓
Ubuntu VM
   ↓
Docker
   ↓
Minikube
   ↓
Kubernetes
```
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
