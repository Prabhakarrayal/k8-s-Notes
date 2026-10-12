# Kubernetes Mentorship — Live Progress Tracker

Last reconciled with available project context: 2026-10-12

> This is the tracking file for the **Kubernetes Mentoring Plan** conversation. It complements, rather than replaces, the detailed revision notes. Update it after meaningful lessons or practical sessions. If the live lab differs from these notes, record the observed state and reconcile the discrepancy before proceeding.

## 1. How we will work

When the user asks for a lesson or notes:

1. Retrieve the relevant prior chat/file context before continuing.
2. Resume from the last verified checkpoint; do not restart from the beginning without reason.
3. Explain theory in depth, with practical examples, architecture/data-flow diagrams where helpful, and a connection to actual lab behaviour.
4. For commands, explain purpose, every meaningful argument, expected output, verification, and common failure modes. Mark read-only, modifying, or destructive actions and distinguish lab-only from production practice.
5. Do not assume command output or cluster state. Ask for current output when the next action depends on it.
6. After a practical task, explain briefly what the task demonstrated about Kubernetes theory.
7. Keep a detailed, cumulative Markdown revision document based on what was actually studied, plus this compact progress tracker.
8. Keep interview/scenario questions and cumulative spaced revision in the learning flow, without letting revision derail the current lesson.
9. When a future concept arises, label it **FUTURE / PARKING LOT** and return to the current lesson.
10. Do not say a Markdown file was updated unless a file write/commit actually succeeded.

## 2. User's learning preferences

- Concept-first: why the component exists → problem solved → definition → location/ownership → internal working → architecture and interactions → practical inspection → troubleshooting → production relevance.
- Beginner-friendly language with accurate technical terminology.
- One major concept at a time; correct misconceptions before advancing.
- Detailed notes suitable for revisiting weeks or months later, not short cheat sheets.
- Practical examples that build on the real lab and include verification.
- Regular spaced recall from older Linux, Docker, networking and Kubernetes topics.
- Interview preparation: beginner, intermediate, scenario, troubleshooting, and cross-technology questions.
- Keep Minikube practice separate from the production-like multi-VM cluster.
- Do not mix Ceph into every lesson; introduce Ceph CSI when persistent storage integration becomes relevant.

## 3. Current recorded lab checkpoint

**Important:** These are the latest recorded notes available, not a live inspection of the machines.

- Lab direction: three Ubuntu VMs (Control Plane, Worker 1, Worker 2) in a KVM/VMM environment.
- Existing Minikube lab: Kubernetes fundamentals and substantial workload practice have been completed.
- Multi-VM preparation recorded as complete: unique node identities, swap preparation, kernel modules, persistent sysctl, Control Plane containerd, CRI configuration, and systemd cgroup configuration.
- A containerd log noted no network configuration in `/etc/cni/net.d`; before CNI installation this was expected.
- The latest detailed file is `kubernetes_revision_notes_current_prebootstrap.md`, dated 2026-09-29. Its next recorded work is worker runtime setup, Kubernetes packages, kubeadm bootstrap, CNI, worker joins and cluster verification.
- Before running any bootstrap command, verify the actual current state in the mentoring chat. Do not repeat completed setup or assume the cluster is still in the recorded state.

### Immediate sequence from the last recorded checkpoint

1. Inspect Worker 1 OS/runtime prerequisites and configure containerd/CRI.
2. Repeat and verify on Worker 2.
3. Verify package/version and node identity/network prerequisites across all three VMs.
4. Install/configure kubeadm, kubelet and kubectl according to the agreed cluster version and current setup.
5. Run `kubeadm init` on the Control Plane only after checking prerequisites and recording the intended Pod CIDR/CNI plan.
6. Configure kubeconfig for the intended administrative user.
7. Install the chosen CNI and verify system Pods/network readiness.
8. Join Worker 1 and Worker 2 using the generated join command.
9. Verify `kubectl get nodes -o wide`, system Pods and node readiness; troubleshoot from events/logs before changing settings.
10. Deploy a small application across the multi-node cluster and verify scheduling and access.

## 4. Progress snapshot

Status meanings: **Completed** = practiced or explained for the present learning stage; **Partial** = introduced or only basic coverage; **Current/Next** = immediate focus; **Future** = intentionally deferred.

| Area | Status | Notes |
|---|---|---|
| Why Kubernetes / orchestration | Completed | Desired state, reconciliation, scaling and failure handling |
| Cluster, Node, Control Plane, Worker | Completed / basic | Revisit in multi-node operational context |
| API Server, Scheduler, etcd, controllers | Basic / partial | Deep internal mechanics can be revisited as needed |
| Minikube and kubectl basics | Completed / practical | Existing local lab |
| Pods, Deployments, ReplicaSets | Practical | Includes scaling and self-healing |
| Services, ClusterIP, NodePort, port-forward | Practical / internals partial | Revisit service routing on multi-node cluster |
| Labels/selectors and endpoints | Basic / practical | Validate with real workload |
| Logs, describe, exec, events | Basic / practical | Continue using for troubleshooting |
| YAML and rollout/rollback | Practical | Previously exercised in Minikube |
| ConfigMap and environment variables | Practical | Secret remains partial |
| CPU/memory requests and limits | Basic practical | QoS and allocatable remain partial |
| StorageClass, PV/PVC and dynamic provisioning | Demonstrated in Minikube | Revisit carefully in multi-node setup |
| Taints/tolerations, scheduler basics | Introduced/practical | Deeper scheduling remains |
| Three-VM identities and prerequisites | Recorded complete | Verify live state before next change |
| Control Plane containerd/CRI/cgroups | Recorded complete | Verify live state if relevant |
| Worker 1 containerd/CRI | Next recorded task | First immediate practical checkpoint |
| Worker 2 containerd/CRI | Next | Follow Worker 1 verification |
| kubeadm bootstrap | Future / next phase | Not recorded as completed |
| CNI installation and worker joins | Future / next phase | Must verify before workload deployment |
| DNS, CNI, kube-proxy and networking internals | Partial / future depth | Build on working multi-node cluster |
| Secret, probes, StatefulSet, DaemonSet, Jobs/CronJobs | Partial / future | Teach in roadmap order |
| Ingress, TLS, RBAC, ServiceAccounts, NetworkPolicy | Future | Production foundations |
| HPA, observability, upgrades, HA and etcd recovery | Future | Advanced operations |
| Ceph CSI | Future | Only when storage integration is in scope |
| Complete application project | Planned | User prefers prioritizing app deployment and important roadmap concepts |

## 5. Revision-note register

| File | Role | Maintenance rule |
|---|---|---|
| `kubernetes_revision_notes_current_prebootstrap.md` | Latest recorded multi-VM/pre-bootstrap notes | Preserve as historical checkpoint; add a continuation or create a newer current revision after bootstrap work |
| `kubernetes_revision_notes_updated.md` | Detailed Minikube/practical revision notes | Compare before editing; likely overlaps another version |
| `kubernetes_revision_notes_updated (1).md` | Versioned copy of detailed notes | Compare before consolidation; do not assume it is more current based on filename alone |
| `kubernetes_day1_revision_notes.md` | Day 1 foundational notes | Preserve foundational history; avoid overwriting it with later progress |
| `Pasted markdown(1).md` | Additional copy/continuation of revision notes | Compare against other copies before deciding the canonical content |
| `Pasted text(20260921-080129).txt` | Original mentorship requirements/roadmap | Treat as teaching contract and roadmap, not proof of completion |
| `PROJECT_CHAT_INDEX.md` | Cross-track chat/file index | Keep track IDs and file inventory synchronized |

### Canonical note policy

- Use `kubernetes_revision_notes_current_prebootstrap.md` as the latest **historical checkpoint** until a newer revision is created.
- Keep the detailed revision notes cumulative: concepts, why, internal working, architecture, examples, commands, expected output, troubleshooting, production connection, recall questions, interview questions and progress.
- Do not silently merge or delete duplicate files. Compare first and preserve history.
- After the user requests a detailed Markdown file, create or update the appropriate canonical file and provide a verified link.
- Record the date and what changed in the tracker.

## 6. Cross-technology connections

- Linux: processes, namespaces, cgroups, systemd, kernel modules, filesystems, networking and logs.
- Docker/containerd: images, containers, runtime, CRI and cgroups; Kubernetes orchestrates workloads rather than simply replacing a runtime.
- OpenStack: virtual machines provide infrastructure on which Kubernetes nodes can run; do not equate a Minikube single-node lab with production cloud architecture.
- Ceph: persistent storage can connect through PV/PVC/StorageClass and CSI/Ceph CSI when that topic is reached.
- Networking: node IPs, routes, firewall rules, Pod CIDR, Service CIDR, CNI and DNS must be considered in the cluster's actual network plan.

## 7. Session update template

Append or update these fields after each meaningful session:

- Date:
- Lesson / practical task:
- Theory completed:
- Commands/actions performed:
- Actual output and verification:
- Troubleshooting/corrections:
- Status changes:
- Revision file updated:
- Next exact step:
- Future topics parked:
- Recall/interview questions to revisit:

## 8. Source of truth and uncertainty

This tracker is a navigational summary, not a substitute for the actual conversation or terminal output. If files conflict, prefer the newest verified practical record over an older roadmap or summary. If no current output is available, say the state is unverified and request only the information needed to proceed safely.
