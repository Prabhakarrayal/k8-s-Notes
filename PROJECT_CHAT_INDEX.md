# Project Chat & Knowledge Index

Last reviewed: 2026-10-12

## Purpose

This index keeps the related learning conversations separate while preserving cross-topic connections. The original conversations remain the source of truth for what was actually taught or practiced. Revision files must record verified progress, not assume that a topic is complete merely because it appears in a roadmap.

## Conversation Register

| Track ID | Conversation / chat family | Scope | Tracking rule | Current known position |
|---|---|---|---|---|
| K8S-01 | Kubernetes Mentoring Plan / Kubernetes | Kubernetes concepts, Minikube, production architecture, kubeadm multi-VM lab | Maintain one cumulative roadmap and revision notes; separate Minikube from production setup | Fundamentals and substantial Minikube work covered; moving toward a 3-VM kubeadm cluster |
| LNX-01 | Linux company learning | Linux administration, troubleshooting, networking, storage, SSH and operational practice | Resume from actual progress; explain commands and risks; update detailed Markdown notes after sessions | Ongoing; existing troubleshooting and revision-note work must be preserved |
| DKR-01 | Docker Learning / Docker Lab | Container concepts, images, runtime, CLI, lifecycle and practical labs | Concept-first teaching, spaced revision, command breakdown and safety labels | Core concepts covered through Docker Engine, images, containers, Hub, CLI and basic `docker run` |
| CEPH-01 | Ceph Learning / Ceph Practical Lab | Ceph theory and hands-on cluster/storage work | Keep theory separate from practical lab; after tasks explain what was demonstrated; identify local files vs Ceph objects | Single-node Ubuntu lab; cephadm/Ceph Squid; one monitor and manager; currently 0 OSDs; `/dev/vdb` is the 20 GB candidate disk |
| OSP-01 | OpenStack / OpenStack Assistant / Production | Horizon/Wahda, domains, projects, networks, routers, instances, volumes, security groups and floating IPs | Distinguish lab/UI tasks from production; preserve exact project and resource context | Several project/network/instance tasks completed; use the original chat for the latest live state |
| PY-01 | Python 1 / Python 2 / Python learning with Linux | Python fundamentals, scripting and operations automation | Continue progressively; connect Python to Linux/operations when relevant | Ongoing; consult the specific Python chats before claiming exact lesson completion |
| COM-01 | Community / general infrastructure discussion | Cross-topic questions, career/production discussion and shared concepts | Treat as supporting context, not the primary record for a technical track | Reference when directly relevant |

## Kubernetes Status Snapshot

This is a summary of the most recent retrieved Kubernetes revision note, not a live cluster check.

- **Environment:** Ubuntu 24.04 VMs in a KVM/VMM learning environment.
- **Previously completed lab:** Minikube with Docker driver; cluster, node, system Pods and basic kubectl inspection.
- **Newer lab direction:** Three-VM Kubernetes learning environment. Control-plane containerd/CRI setup is recorded as completed.
- **Next recorded steps:** Finish worker runtime setup on Worker 1 and Worker 2; install kubeadm/kubelet/kubectl where required; initialize the control plane; install CNI; join workers; verify the three-node cluster; deploy an application.
- **Roadmap items to track explicitly:** DaemonSet and worker routing are partial/current; StatefulSet, Job/CronJob, DNS, Ingress, NetworkPolicy, ConfigMap/Secret, RBAC, scheduling constraints, PV/PVC/StorageClass/CSI, HPA and CRD/Operators remain to be completed or revisited according to the detailed tracker.
- **Important:** Verify current VM/cluster state in the Kubernetes chat before running bootstrap or destructive commands. The note may lag behind the live lab.

## Files Found During This Review

### Kubernetes revision files

- `kubernetes_revision_notes_current_prebootstrap.md` — latest retrieved revision note; describes the pre-bootstrap state and the three-VM kubeadm direction.
- `kubernetes_revision_notes_updated.md` and `kubernetes_revision_notes_updated (1).md` — detailed Day 1 revision notes; these appear to be duplicate/versioned copies.
- `kubernetes_day1_revision_notes.md` — detailed early-lab notes, including Minikube setup, cluster inspection, architecture and progress tracking.
- `Pasted markdown(1).md` — another copy/version of the Day 1 notes.
- `Pasted text(20260921-080129).txt` — Kubernetes mentorship requirements and roadmap, including safety rules, cross-topic connections, spaced revision and interview questions.

### Cross-topic roadmap files

- `Enterprise_OpenStack_DevOps_Roadmap_Updated.pdf`
- `Enterprise_OpenStack_DevOps_Roadmap(1).pdf`
- `Enterprise_OpenStack_DevOps_Roadmap.pdf`
- `Enterprise_OpenStack_DevOps_Roadmap_v2.pdf`
- `Enterprise_OpenStack_DevOps_Roadmap_Final.pdf`
- `Essential_Technologies_for_OpenStack_and_DevOps.pdf`

These are roadmap documents, not proof that every listed technology or topic has been studied. Several are near-duplicates; do not treat them as separate learning tracks.

## How We Will Maintain the Tracks

After each learning session:

1. Identify the exact conversation/track and the latest point actually reached.
2. Extract relevant attached or Library files before relying on memory for file-dependent questions.
3. Update that track's status as **Completed**, **Partially covered**, **Current**, or **Future**.
4. Record verified lab facts separately from plans or assumptions.
5. Add commands, troubleshooting outcomes, and important corrections to that track's Markdown notes.
6. Preserve original chats and history; consolidate duplicates only after comparing their contents.
7. Keep cross-topic links concise and intentional (for example, Linux networking in Kubernetes, Docker runtime in Kubernetes, OpenStack VMs hosting Kubernetes nodes, and Ceph through CSI when storage is reached).
8. Never claim that a file was extracted, updated, or pushed unless the operation actually succeeded.

## Important Teaching Preferences

- Start from the reason/problem, then explain the concept, internal working, architecture, commands and verification.
- Use simple English without weakening technical accuracy.
- Teach one major concept at a time; correct misunderstandings before moving on.
- Explain commands and their arguments; label read-only, modifying and destructive actions.
- Clearly separate lab-only actions from production practice.
- Use spaced revision across older topics, not only the immediately previous lesson.
- Include relevant scenario-based and interview/viva questions.
- Maintain detailed, cumulative Markdown notes based on what was actually studied.
