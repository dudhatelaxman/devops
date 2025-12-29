# Kubernetes for DevOps Engineers — Production & Interview Guide

**Version:** 1.0  
**Last updated:** 2025-12-29  
**Audience:** DevOps / SRE engineers with ~3–4 years experience who want production-ready knowledge and interview preparation.

This document is a complete, practical reference covering Kubernetes architecture, commands, resource types, production patterns, troubleshooting runbooks, CI/CD/GitOps integration, security & hardening, observability, and interview Q&A. It focuses on "how" and "why" as much as "what", with real-world examples and command snippets ready for operations.

---

## Table of Contents

1. [Core definition & value proposition](#core-definition--value-proposition)  
2. [Detailed Kubernetes Architecture — explanation & ASCII diagram](#detailed-kubernetes-architecture----explanation--ascii-diagram)  
3. [Control plane components — responsibilities & operational concerns](#control-plane-components----responsibilities--operational-concerns)  
4. [Node components & container runtime — behaviour & debugging tips](#node-components--container-runtime----behaviour--debugging-tips)  
5. [kubectl cheat sheet (table)](#kubectl-cheat-sheet-table)  
6. [Pod vs Deployment vs StatefulSet vs DaemonSet — comparison table and use cases](#pod-vs-deployment-vs-statefulset-vs-daemonset---comparison-table-and-use-cases)  
7. [Service types — comparison table and traffic patterns](#service-types---comparison-table-and-traffic-patterns)  
8. [Networking & CNI — concepts, service discovery, DNS and policies](#networking--cni---concepts-service-discovery-dns-and-policies)  
9. [Storage: PV, PVC, StorageClass, snapshots — patterns & production examples](#storage-pv-pvc-storageclass-snapshots---patterns--production-examples)  
10. [ConfigMaps, Secrets, and external secret stores — best practices](#configmaps-secrets-and-external-secret-stores---best-practices)  
11. [Workloads & patterns — Deployments, Jobs, CronJobs, DaemonSets, StatefulSets](#workloads--patterns---deployments-jobs-cronjobs-daemonsets-statefulsets)  
12. [Deploy strategies: rolling, canary, blue/green, A/B — how to implement safely](#deploy-strategies-rolling-canary-bluegreen-ab---how-to-implement-safely)  
13. [CI/CD & GitOps integration — concrete pipelines and ArgoCD example](#cicd--gitops-integration---concrete-pipelines-and-argocd-example)  
14. [Observability: logs, metrics, traces — stack suggestions & examples](#observability-logs-metrics-traces---stack-suggestions--examples)  
15. [Security & hardening — RBAC, NetworkPolicy, Pod Security, image & runtime security](#security--hardening---rbac-networkpolicy-pod-security-image--runtime-security)  
16. [Scaling & autoscaling — HPA, VPA, Cluster Autoscaler, best practices](#scaling--autoscaling---hpa-vpa-cluster-autoscaler-best-practices)  
17. [Backup, DR & cluster upgrades — tested playbooks & commands](#backup-dr--cluster-upgrades---tested-playbooks--commands)  
18. [Real-world troubleshooting scenarios & runbooks (detailed)](#real-world-troubleshooting-scenarios--runbooks-detailed)  
19. [Production best practices checklist (operational)](#production-best-practices-checklist-operational)  
20. [Interview Questions & Answers — Beginner / Intermediate / Advanced (7+ each)](#interview-questions--answers---beginner--intermediate--advanced-7-each)  
21. [Quick revision notes — 20+ one-liners for interviews](#quick-revision-notes--20-one-liners-for-interviews)  
22. [Further reading & references](#further-reading--references)

---

## Core definition & value proposition

- Kubernetes is an open-source system that automates deployment, scaling, and management of containerized applications across a cluster of machines. It provides an API-driven control plane that reconciles the desired state defined by users with the cluster's actual state.
- Why it matters in production: Kubernetes abstracts infrastructure heterogeneity (cloud/on-prem/kvm) and operational tasks (scheduling, health checks, auto-scaling, rolling updates), allowing teams to deliver features faster while maintaining stability.
- The platform encourages declarative operations: you define "what" you want (manifests), and the control plane ensures the cluster converges to that state — this enables GitOps and reproducible operations.
- Kubernetes is extensible through CRDs (Custom Resource Definitions) and operators — this is how you run databases, storage controllers, or multi-step apps reliably.
- In production, Kubernetes is not just for “running containers” — it's the basis for platform automation (self-healing, service discovery, resource isolation, network segmentation, observability pipelines).
- Key trade-offs: operational complexity and learning curve vs. benefits in scale, portability, and ecosystem (service mesh, operators, GitOps).
- For SREs, Kubernetes provides primitives to codify runbooks (readiness/liveness), enforce SLO-oriented automation (auto-scaling, PDBs), and centralize observability.

---

## Detailed Kubernetes Architecture — explanation & ASCII diagram

Kubernetes can be viewed as a set of cooperating components split between the control plane (masters) and worker nodes. The control plane exposes the Kubernetes API and enforces desired state; worker nodes run workloads and provide compute resources.

ASCII architecture diagram (detailed):

```
                                           External Clients
                                                |
  +---------------------------------------------+---------------------------------------------+
  |                                            LB (L4/L7)                                     |
  +---------------------------------------------+---------------------------------------------+
                                                |
                                 +--------------v--------------+
                                 |   kube-apiserver (HA set)   |
                                 +--------------+--------------+
                                                |
                   +----------------------------+-----------------------------+
                   |                            |                             |
           +-------v-------+            +-------v--------+           +--------v--------+
           | kube-scheduler|            | controller mgr |           |  admission webhooks |
           | (scheduling)  |            | (reconcilers)  |           | (OPA, Kyverno, etc) |
           +-------+-------+            +-------+--------+           +--------------------+
                   |                            |
      Scheduling decisions -> Pod placement     |  Controllers (ReplicaSet, Deployment,
                   |                            |  StatefulSet, DaemonSet, Job) reconcile
                   |                            |
           +-------v----------------------------v---------------------------+
           |                        etcd (distributed KV)                     |
           |  - cluster state (manifest objects)                              |
           |  - must be secured, backed up, and with consistent latency       |
           +-----------------------------------------------------------------+
                                                |
  +---------------------------------------------+---------------------------------------------+
  |                                                                                       |
  |                      Worker Nodes (kubelet + runtime + kube-proxy)                  |
  |                                                                                       |
  |  +----------------+    +----------------+   +----------------+   +----------------+    |
  |  |  Node A        |    |  Node B        |   |  Node C        |   |  Node D        |    |
  |  |  kubelet       |    | kubelet        |   | kubelet        |   | kubelet        |    |
  |  |  containerd    |    | containerd     |   | containerd     |   | containerd     |    |
  |  |  kube-proxy    |    | kube-proxy     |   | kube-proxy     |   | kube-proxy     |    |
  |  |  CNI (Calico)  |    |  CNI (Calico)  |   |  CNI (Calico)  |   |  CNI (Calico)  |    |
  |  |  Pod(s)        |    |  Pod(s)        |   |  Pod(s)        |   |  Pod(s)        |    |
  |  +----------------+    +----------------+   +----------------+   +----------------+    |
  +-----------------------------------------------------------------------------------------+

External dependencies:
- CI/CD systems call kube-apiserver (or update Git in GitOps)
- Monitoring stacks scrape metrics (Prometheus)
- Logging agents collect stdout/stderr (Fluentd / Fluent Bit)
- Storage via CSI drivers for PVs
```

Why each component exists and how it interacts:
- **kube-apiserver** — central REST interface. All writes/read go here. Every change (kubectl apply, controllers) goes through the API server which validates, authenticates, authorizes, and persists objects to etcd.
  - Operational concern: high availability (multiple apiservers behind LB) and rate limits. Use API auth (OIDC, certs) and RBAC for access control.
  - Commands: `kubectl get --raw /`, `kubectl auth can-i ...`
- **etcd** — strongly-consistent key-value store holding cluster state. It's the "single source of truth" for Kubernetes.
  - Why: controllers and scheduler read from etcd; any inconsistency leads to unhealthy clusters.
  - Operational concern: backup snapshots, compaction, disk I/O, secure endpoints with mTLS.
  - Commands: `etcdctl snapshot save`, `etcdctl endpoint status`.
- **Scheduler** — decides which node to place a new pod on, evaluating resources, taints/tolerations, affinity, topology constraints, and custom scheduling rules.
  - Real-world note: scheduling latency can spike under heavy API server load; monitor scheduler metrics.
- **Controller Manager** — a process running controllers (replica, job, endpoint controllers) that make actual changes to reach desired state.
  - Why: reconciler pattern — controllers continuously compare desired vs actual and take action.
- **kubelet** — node-level agent that ensures containers are running as described in Pod specs. Talks to container runtime to start/stop containers and reports status to the API server.
  - Debugging tip: check `journalctl -u kubelet` when nodes misbehave.
- **kube-proxy** — implements Service networking on each node using iptables or IPVS. Routes traffic to pod endpoints.
  - Production tip: use IPVS mode for large scale clusters to reduce CPU overhead.
- **CNI (Container Network Interface)** — plugin interface for configuring pod networks. Examples: Calico, Cilium, Flannel.
  - Why: Provides pod-to-pod networking, network policies, and integration with cloud networking.
  - Production note: choose CNI with eBPF support for performance (Cilium).
- **Container runtime** — containerd or CRI-O; runs OCI containers. Docker engine is deprecated as runtime in kube, but docker images are still compatible.
  - Operational tip: ensure runtime's socket and storage drivers are monitored.

---

## Control plane components — responsibilities & operational concerns

- **kube-apiserver**
  - Role: Authenticate, validate, authorize requests; persist manifests to etcd.
  - Production concerns: TLS cert rotation, admission webhook performance, request throttling, audit logging.
  - Example: `kubectl apply -f deployment.yaml` → API server validates and writes Deployment to etcd.
  - How to monitor: Prometheus metrics `apiserver_request_latencies_seconds`, audit logs.
- **etcd**
  - Role: Persistent strongly-consistent store of Kubernetes manifests and cluster state.
  - Why critical: Losing etcd means losing cluster state. Recovery requires known tested procedures.
  - Operations: snapshot, compaction, secure TLS endpoints, isolate disk IOPS.
  - Example commands:
    ```bash
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/etcd/ca.crt --cert=/etc/etcd/etcd.crt --key=/etc/etcd/etcd.key \
      snapshot save /backup/etcd-snap.db
    ```
- **kube-scheduler**
  - Role: Place pods on nodes. Supports inter-pod affinity/anti-affinity, resource requests, taints/tolerations.
  - Why: Efficient placement matters for binpacking and availability.
  - Tuning: custom scheduler or priority/weighting can be used for multi-tenancy.
- **controller-manager**
  - Role: Run controllers (replicaset, deployment, endpoints, token controllers).
  - How: Watches resources in API server and reconciles resources.
  - Production issue: controllers slow if API server or etcd slow.
- **Admission controllers**
  - Role: Validate or mutate requests to the API server before persistence. Example: PodSecurity, ResourceQuota, OPA Gatekeeper.
  - Use: Enforce organization policies (immutable images, resource limits).
- **Cloud-controller-manager** (when running in cloud)
  - Role: Integrate with cloud provider APIs (load balancers, node lifecycle, persistent volumes).
  - Production concern: cloud IAM/perms must be correct for controllers to provision cloud resources.

---

## Node components & container runtime — behaviour & debugging tips

- **kubelet**
  - Role: Watches API server for PodSpecs assigned to the node, interfaces with CRI to run containers, reports status (NodeReady, conditions).
  - Debugging: `journalctl -u kubelet` OR `systemctl status kubelet`; inspect `/var/log/syslog`.
  - Why: Problems here cause NodeNotReady, pod failure to start, failed liveness/readiness.
- **container runtime (containerd / CRI-O)**
  - Role: Pull images, manage container lifecycle via the OCI runtime (runc, crun).
  - Commands for containerd: `ctr images ls`, `crictl ps`, `crictl logs <containerId>`.
  - Troubles: disk/overlay storage full, image corruption; runtime issues manifest as `ImagePullBackOff` or `ErrImagePull`.
- **kube-proxy**
  - Modes: `iptables` or `ipvs`. `ipvs` recommended at scale for deterministic performance.
  - Debugging: check iptables rules, `iptables -t nat -L -n`, or `ipvsadm -L`.
  - Why: misconfiguration can cause service traffic errors.
- **CNI plugin**
  - Role: Program host networking for pods, implement NetworkPolicy support.
  - Popular: Calico, Cilium, Flannel. Choose by features: eBPF offload (Cilium) vs policy-first (Calico).
  - Debugging: check CNI daemonset logs (`kubectl -n kube-system logs daemonset/calico-node`).
- **Node local components (kube-proxy, kubelet, runtime) interaction**
  - Example issue: kubelet reports `Image garbage collection` when disk low, leading to crash loops.
  - Fix: increase node root partition, set `imageGCLowThreshold`, or run `docker system prune`.
- **System resource limits**
  - Nodes need headroom for kube-system pods. Use taints/tolerations to isolate critical workloads.
  - Commands: `kubectl top nodes`, `kubectl describe node <node>` to inspect allocatable vs capacity.

---

## kubectl cheat sheet (table)

A compact, structured table of commonly used kubectl commands with explanations and practical flags.

| Category | Command | Explanation / Example |
|---|---:|---|
| Cluster info | `kubectl version --short` | Show client/server versions |
| | `kubectl cluster-info` | Show API server & services |
| Nodes | `kubectl get nodes -o wide` | List nodes, IPs, status |
| Pods | `kubectl get pods -A` | List all pods across namespaces |
| | `kubectl get pods -n prod -o wide` | Pods with nodes & IPs |
| Inspect | `kubectl describe pod mypod -n prod` | Events, reasons, container states |
| Logs | `kubectl logs mypod -c container -n prod` | Current logs |
| | `kubectl logs mypod -c container -p -n prod` | Previous instance logs (after restart) |
| Exec | `kubectl exec -it mypod -c container -n prod -- /bin/sh` | Shell into container |
| Apply | `kubectl apply -f k8s/` | Declarative apply of manifests |
| Diff | `kubectl diff -f k8s/deploy.yaml` | Show server-side diff |
| Rollout | `kubectl rollout status deployment/web -n prod` | Wait for rollout completion |
| Rollback | `kubectl rollout undo deployment/web -n prod` | Rollback to previous revision |
| Scale | `kubectl scale --replicas=5 deployment/web -n prod` | Scale replicas |
| Debug | `kubectl debug node/<node> -it --image=busybox` | Ephemeral debug pod on node |
| Events | `kubectl get events -n prod --sort-by=.lastTimestamp` | Sorted events |
| Port-forward | `kubectl port-forward svc/web 8080:80 -n prod` | Local access to service |
| Metrics | `kubectl top pod -n prod` | Resource usage (requires metrics-server) |
| Config | `kubectl config get-contexts` | List kubeconfig contexts |
| Secrets | `kubectl create secret generic db-secret --from-literal=DB_PASS=... -n prod` | Create secret |
| Apply Kustomize | `kubectl apply -k overlays/prod` | Apply Kustomize dir |
| Clean | `kubectl delete -f k8s/ --grace-period=0 --force` | Force delete resources (careful) |

Practical tips:
- Use `--dry-run=client` or `--server-dry-run` to validate changes.
- Use `-o yaml` to grab current manifest, edit, and reapply for safe changes.
- Use contexts to manage multiple clusters: `kubectl config set-context`.

---

## Pod vs Deployment vs StatefulSet vs DaemonSet — comparison table and use cases

| Concern | Pod | Deployment | StatefulSet | DaemonSet |
|---|---:|---:|---:|---:|
| Primary abstraction | Single or group of co-located containers | Controller for stateless workloads (replicasets) | Controller for stateful apps with stable identity | Ensures pod runs on each (or selected) node |
| Replica management | No | Yes (ReplicaSet) | Yes, ordered | No replicas; runs per-node |
| Stable network ID | No | No (pods are ephemeral) | Yes — stable DNS names (pod-ordinal) | No |
| Stable storage | No | No | Yes — volumeClaimTemplates produce per-pod PVCs | Can mount host paths or per-node volumes |
| Use case | Single container task, sidecars | Web services, stateless microservices | Databases, message queues, stateful systems | Node-level services (logs, metrics, CNI, storage) |
| Rolling updates | N/A | Yes (strategy configurable) | Rolling but ordered (pod-0, pod-1) | Replace pods across nodes; update strategy available |
| Example | `kubectl run --image=busybox` | nginx web frontend with autoscaling | Postgres cluster requiring persistent identity | Prometheus node-exporter, FluentD, Filestore mount |
| Command example | `kubectl create pod` (rare) | `kubectl create deploy web --image=nginx` | `kubectl apply -f statefulset.yaml` | `kubectl apply -f daemonset.yaml` |

Why choose one over another:
- Choose **Deployment** for horizontally scalable stateless services with simple lifecycle and rolling updates.
- Choose **StatefulSet** when pods need persistent identity, ordered scaling, or stable storage (e.g., leader election via stable DNS).
- Choose **DaemonSet** for per-node agents that collect logs/metrics or manage local storage.
- Pods are the basic unit and are mostly managed by controllers (Deployments/StatefulSets).

---

## Service types — comparison table and traffic patterns

| Service Type | Namespace | Load-balanced | Use case | Production notes |
|---|---:|---:|---|---|
| ClusterIP | Internal only | No | Internal microservice communication | Default; used with Ingress for external access |
| NodePort | External via node port | No (single port per node) | Quick expose to external without cloud LB | Limited control/port collisions; usually for dev |
| LoadBalancer | External cloud LB | Yes (cloud-managed) | Expose service publicly with managed LB | Cloud-specific costs and security groups; use for production ingress if LB managed by cloud |
| ExternalName | DNS alias to external service | N/A | Map service name to external DNS | Useful for legacy or external services; no endpoints managed by K8s |

Traffic patterns:
- Internal-to-internal: Services + ClusterIP (DNS resolves to ClusterIP) and kube-proxy/IPVS load balances to endpoints.
- Ingress: For HTTP(S), use Ingress + IngressController (e.g., nginx, Traefik) to route to services; offload TLS at ingress or use service mesh for mTLS.
- External traffic: Prefer LoadBalancer + Ingress; for global traffic use Global LBs with external DNS and health checks.

Production example:
- Use NGINX Ingress Controller with Cert-Manager for TLS and Let's Encrypt automation. Use external WAF and geo-LB in front for DDoS protection.

---

## Networking & CNI — concepts, service discovery, DNS and policies

- **Pod networking:** every pod gets an IP; connectivity should be flat by design (pods can talk directly to pods in other nodes). CNI plugins implement this.
- **Service discovery:** CoreDNS provides cluster DNS. Services are reachable by `<service>.<namespace>.svc.cluster.local`.
  - Commands: `kubectl exec -it <pod> -- nslookup web -n default`
- **kube-proxy and routing:** kube-proxy programs iptables/ipvs rules. At scale use IPVS for high throughput.
- **NetworkPolicy:** defines allowed traffic in/out of pods. Default is allow-all unless CNI enforces deny-by-default policy model (some CNIs do).
  - Example: deny-by-default pattern — create an explicit `allow` NetworkPolicy for desired traffic.
- **Ingress vs Service:** Ingress manages HTTP(S) routing and host/path rules; Service (ClusterIP) is used as backend target.
- **Egress controls:** Use egress rules or egress gateways (service mesh) to restrict outbound traffic and enforce proxies.
- **Multitenancy & network isolation:** Use namespaces + NetworkPolicies + resource quotas to isolate teams.
- **Observability:** Monitor CNI metrics, CoreDNS latency, kube-proxy packet drops, and iptables rule count.
- **Troubleshooting commands:**
  - `kubectl get pods -n kube-system -o wide` (CNI, kube-proxy pods)
  - `kubectl logs -n kube-system <cni-pod>`
  - `kubectl exec -it <pod> -- ping <pod-ip>` to test connectivity
  - `kubectl exec -it <pod> -- curl -v http://service.namespace.svc.cluster.local:80`

---

## Storage: PV, PVC, StorageClass, snapshots — patterns & production examples

- **PersistentVolume (PV)** — cluster resource representing provisioned storage (capacity, access mode).
- **PersistentVolumeClaim (PVC)** — user's claim for storage using size, access mode, storageClass.
- **StorageClass** — describes provisioner (CSI driver), parameters (type, iops), reclaimPolicy, and binding mode; supports dynamic provisioning.
  - Example StorageClass YAML for AWS EBS (gp3):
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast-ssd
    provisioner: ebs.csi.aws.com
    parameters:
      type: gp3
      fsType: ext4
    volumeBindingMode: WaitForFirstConsumer
    reclaimPolicy: Delete
    ```
- **Access modes:** `ReadWriteOnce` (RWO), `ReadOnlyMany` (ROX), `ReadWriteMany` (RWX). Choose based on app needs.
- **Snapshots & backups:** Use CSI snapshots (VolumeSnapshot) for point-in-time backups; use cloud provider snapshots for larger DR.
- **Backup strategies:** daily PV snapshots + weekly full offsite backups. Test restores frequently.
- **Stateful workloads:** use StatefulSet + PVC templates; ensure `volumeBindingMode: WaitForFirstConsumer` to bind PVs to correct zone.
- **Performance tuning:** match IO profile to disk type (ssd/gp3/io1) and set appropriate filesystem options.
- **Troubleshooting PV/PVC binding:**
  - `kubectl describe pvc <pvc>`
  - `kubectl get pv` to inspect underlying PV
  - Check provisioner logs (e.g., CSI driver daemonset logs)
  - Check cloud provider quotas & IAM permissions for dynamic provisioning

Real-world example:
- Production Postgres: provision `fast-ssd` StorageClass with iops, use pgBackRest for logical backups and EBS snapshots for physical backups. Use standby replicas across AZs.

---

## ConfigMaps, Secrets, and external secret stores — best practices

- **ConfigMap**: store non-sensitive configuration and inject via env vars or mount as file. Keep environment-specific overrides in different K8s overlays (Kustomize).
  - Example: mount `ConfigMap` as a volume to read app config.
- **Secret**: store sensitive data (DB creds, TLS keys). K8s stores Secrets in base64 in etcd — use encryption at rest and limit RBAC access.
  - Avoid exposing secrets via logs or `kubectl describe`.
- **External secret managers**: integrate Vault, AWS Secrets Manager, Azure Key Vault via CSI or External Secrets Operator. This avoids having secrets in etcd or Git.
- **SealedSecrets**: use sealed-secrets to encrypt secret manifests in Git (controller unseals in cluster).
- **Best practices:**
  - Never commit plaintext secrets to Git.
  - Prefer mounting secrets as files — easier rotation and less likely to be leaked in `ps`/env dumps.
  - Restrict which service accounts may access secrets using RBAC.
  - Rotate secrets periodically and support secret versioning.
- **Practical commands:**
  - Create secret: `kubectl create secret generic db-secret --from-literal=DB_PASS=myPass -n prod`
  - View secret (base64): `kubectl get secret db-secret -o yaml -n prod` (decode with `base64 --decode`)
- **Why external store:** central audit, revocation, key rotation independent of cluster lifecycle.

---

## Workloads & patterns — Deployments, Jobs, CronJobs, DaemonSets, StatefulSets

- **Deployments**: best for stateless replicas. Use readiness/liveness probes; set resource requests/limits; define `strategy.rollingUpdate` parameters for controlled rollouts.
  - Example: `kubectl set image deployment/web web=myorg/web:v1.2.3` then `kubectl rollout status`.
- **Jobs & CronJobs**: for one-off tasks or scheduled maintenance (migrations, backups). Ensure appropriate `backoffLimit`, `successfulJobsHistoryLimit`.
  - Example CronJob for nightly snapshot; always test idempotency.
- **DaemonSets**: for per-node services (node exporters, log collectors). Use `hostPath` and `hostPID` carefully; secure with NodeSelector to limit run nodes.
- **StatefulSets**: for databases and clustered workloads needing stable identity and persistent storage. Use headless services for stable DNS and `volumeClaimTemplates` per pod.
- **Sidecars & Init containers**: use sidecars for logging, proxying, or config refreshers. Init containers run before main container and can perform schema migrations or downloads.
- **Patterns:**
  - Sidecar for TLS termination or secrets refresh.
  - Ambassador pattern for edge proxy.
  - Adapter pattern for log shipping.
- **PodDisruptionBudget (PDB)**: protects availability during voluntary disruptions (node upgrades). Define minAvailable or maxUnavailable depending on SLA.
  - Example: `minAvailable: 2` ensures at least two pods are available.
- **Why these patterns matter:** they make deployments safer, lifecycle predictable, and easier to automate.

---

## Deploy strategies: rolling, canary, blue/green, A/B — how to implement safely

- **Rolling updates**: default for Deployments; configure `maxSurge` and `maxUnavailable`. Use readiness probes to ensure traffic moves only to healthy pods.
  - Example:
    ```yaml
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
    ```
- **Canary deployments**:
  - Gradually shift traffic using weighted routing (Ingress, service mesh, or traffic manager).
  - Tools: Flagger, Istio VirtualService, Argo Rollouts.
  - Workflow: deploy canary, run smoke tests & synthetic checks, increase traffic weight, promote on success.
- **Blue/Green**:
  - Deploy new version to separate environment. Swap traffic at LB level. Use DB migrations with backward-compatible changes.
  - Ensure traffic cutover has rollback plan.
- **A/B testing**:
  - Route subset of users to new variant; measure metrics and promote based on success.
- **Automated promotion & rollback**:
  - Use observability metrics (error rate, latency, SLO breaches) as criteria for promotion.
  - Integrate with GitHub Actions/Jenkins to trigger deployment and monitor via Prometheus queries.
- **Why careful rollout matters**:
  - Avoid cascading failures; use canary analysis and SLO-aware decisions to control risk.
- **Practical commands**:
  - `kubectl rollout status deployment/web -n prod`
  - `kubectl set image deployment/web web=repo/image:sha -n prod`
  - Argo Rollouts: `kubectl argo rollouts get rollout web-rollout -n prod`

---

## CI/CD & GitOps integration — concrete pipelines and ArgoCD example

- **CI pipeline** (build → test → image registry):
  1. Checkout code, run unit tests.
  2. Build container image, tag with semantic version + SHA.
  3. Scan image for vulnerabilities (Trivy).
  4. Push image to registry (ECR, GHCR).
  5. Publish SBOM and artifacts.
- **Deployment pipeline** (CD):
  - Option A: Traditional CD — CI triggers a pipeline to `kubectl apply` manifests or Helm charts.
  - Option B: GitOps — CI updates manifests in a Git repo; ArgoCD/Flux reconciler applies changes to cluster.
- **GitHub Actions example (build & update manifests)**:
  ```yaml
  name: build
  on: [push]
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: Build image
          uses: docker/build-push-action@v4
          with:
            push: true
            tags: ghcr.io/myorg/app:${{ github.sha }}
        - name: Update k8s manifest
          run: |
            yq -i '.spec.template.spec.containers[0].image = "ghcr.io/myorg/app:${{ github.sha }}"' k8s/deploy.yaml
            git add k8s/deploy.yaml
            git commit -m "ci: bump image ${{ github.sha }}" || true
            git push origin HEAD:gitops
  ```
- **ArgoCD flow**:
  - ArgoCD watches `gitops` repo path. When CI commits new manifest, ArgoCD automatically syncs cluster to desired state.
  - Provides UI, drift detection, rollback, and diffing.
- **Security gating**:
  - Add image scanning and manifest policy checks (OPA/Gatekeeper) before promotion to production.
- **Production example**:
  - For a microservices platform: CI pushes images to ECR and updates Helm values in `gitops` repo. ArgoCD syncs `production` overlay which performs `helm upgrade` and health checks.
- **Rollback strategy**:
  - Store previous manifests/tags in Git history and roll back via ArgoCD; or perform `kubectl rollout undo deployment/<name>`.

---

## Observability: logs, metrics, traces — stack suggestions & examples

- **Logging pipeline**:
  - Applications log to stdout/stderr. Deploy Fluent Bit/Fluentd as DaemonSet to collect logs and forward to Elasticsearch/Opensearch, Loki, or hosted logging (Datadog, Splunk).
  - Example Fluent Bit config that sends logs to Elasticsearch is a typical production pattern.
- **Metrics pipeline**:
  - Use Prometheus Operator (kube-prometheus-stack) to deploy Prometheus, node-exporter, kube-state-metrics, and a Grafana instance.
  - Create alerting rules (Alertmanager) tied to SLO thresholds (e.g., error rate > 1% for 5 minutes).
- **Tracing**:
  - Use OpenTelemetry SDKs to instrument services; export traces to Jaeger or hosted tracing (Lightstep, Datadog).
  - Ensure trace context propagation across services via HTTP headers.
- **Correlation**:
  - Include trace ID in logs to correlate a request across services. Use structured logging (JSON) with fields for trace/span and request id.
- **Dashboards & SLOs**:
  - Key dashboards: cluster health, node resource utilization, pod restarts, service latency, error budget burn rate.
  - SLOs example: p99 latency < 500ms, availability 99.95%.
- **Examples & commands**:
  - Install kube-prometheus-stack:
    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
    ```
  - Query example in Prometheus:
    ```promql
    sum(rate(http_requests_total{job="api",status!~"5.."}[5m])) by (instance)
    ```
- **Alerting**:
  - Alertmanager routes critical alerts to PagerDuty and others to Slack. Design on-call playbooks for alerts that cross thresholds.

---

## Security & hardening — RBAC, NetworkPolicy, Pod Security, image & runtime security

- **RBAC**
  - Principle of least privilege: grant minimal permissions to service accounts and users.
  - Use namespace-scoped Roles & RoleBindings for apps; use ClusterRoles sparingly with admin controls.
  - Check permissions with `kubectl auth can-i`.
  - Example to create a `read-only` role for a namespace:
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata: { name: read-only, namespace: staging }
    rules:
      - apiGroups: [""]
        resources: ["pods", "services", "endpoints"]
        verbs: ["get", "list", "watch"]
    ```
- **Pod Security**
  - Enforce non-root containers, disallow privileged, and restrict hostPath usage.
  - Use Pod Security admission (builtin) with `restricted`/`baseline`/`privileged` modes.
  - Example snippet to require non-root:
    ```yaml
    securityContext:
      runAsNonRoot: true
      readOnlyRootFilesystem: true
    ```
- **NetworkPolicy**
  - Default deny ingress/egress and explicitly allow traffic between trusted services.
  - Provides micro-segmentation at the pod level. Example to allow traffic only from `frontend` pods to `db` pods:
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata: { name: allow-frontend-db, namespace: prod }
    spec:
      podSelector: { matchLabels: { app: db } }
      policyTypes: [Ingress]
      ingress:
        - from:
          - podSelector: { matchLabels: { app: frontend } }
    ```
- **Image security**
  - Use signed images (cosign), enforce image provenance, scan images in CI (Trivy/Grype), minimize base images.
  - Example scanning: `trivy image ghcr.io/org/app:sha`
- **Runtime security**
  - Use AppArmor / SELinux profiles, seccomp policies, drop Linux capabilities and set `no_new_privileges`.
  - Example run flags in Pod spec:
    ```yaml
    securityContext:
      capabilities:
        drop: ["ALL"]
      allowPrivilegeEscalation: false
    ```
- **Secrets encryption & management**
  - Encrypt secrets in etcd (API server `--encryption-provider-config`), rotate KMS keys, and use external secret store integrations (Vault/Secrets Store CSI).
- **Audit logging & compliance**
  - Enable API audit logs with proper rules and ship to secure storage for forensic analysis.
- **Practical hardening checklist**
  - No `hostPath` unless necessary; use network policies; limit cluster-admin bindings; enable PodSecurity; scan images; rotate certs; back up etcd.

---

## Scaling & autoscaling — HPA, VPA, Cluster Autoscaler, best practices

- **Horizontal Pod Autoscaler (HPA)**
  - Scales replicas based on metrics (CPU, memory, custom metrics).
  - Example HPA config:
    ```yaml
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata: { name: web-hpa, namespace: prod }
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: web
      minReplicas: 2
      maxReplicas: 20
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 60
    ```
- **Vertical Pod Autoscaler (VPA)**
  - Adjusts resource requests/limits for pods. Use carefully in production with eviction mode or recommender mode.
- **Cluster Autoscaler**
  - Adds or removes nodes based on pending/waiting pods. Cloud provider specific; use node groups and taints to control scaling.
- **Best practices**
  - Set resource requests properly — HPA uses requests to calculate scaling; under-requested pods cause overcommit and instability.
  - Use PDB to prevent excessive voluntary eviction during node drains.
  - Use scaling policies (cooldown periods) and use metrics server or Prometheus Adapter for custom metrics.
  - Test autoscaling under load in staging to ensure stable scaling behavior.
- **Commands**
  - `kubectl get hpa -n prod`
  - `kubectl top pods -n prod`
  - `kubectl describe hpa web-hpa -n prod`

---

## Backup, DR & cluster upgrades — tested playbooks & commands

- **etcd backup & restore**
  - Backup:
    ```bash
    ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F).db \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/etcd/ca.crt --cert=/etc/etcd/etcd.crt --key=/etc/etcd/etcd.key
    ```
  - Restore: tested procedure with restored cluster bootstrapping; always test in staging.
- **PV backup**
  - Use CSI snapshots or provider snapshots. Test restore frequencies.
- **Cluster upgrade best practices**
  - Control plane first (masters), then nodes in a controlled draining sequence.
  - Use `kubectl drain <node> --ignore-daemonsets --delete-local-data` then upgrade kubelet/kube-proxy/runtime and uncordon.
  - Validate node after upgrade: `kubectl get nodes` and `kubectl describe node <node>`.
- **Disaster recovery playbook**
  - Document contact list, timeline, steps and rollback commands.
  - Have script to restore etcd snapshot to new control plane nodes and rejoin workers.
- **Testing**
  - Schedule regular DR drills, measure RTO/RPO, and refine runbooks.
- **Automation**
  - Automate backups and snapshot retention with cronjobs or external backup management systems, store in offsite/immutable storage (S3).

---

## Real-world troubleshooting scenarios & runbooks (detailed)

Each scenario includes symptoms, diagnosis steps, root cause categories, remediation steps and preventative measures.

### Scenario: CrashLoopBackOff after deployment (web service restarts repeatedly)

- Symptoms:
  - Pod in `CrashLoopBackOff`.
  - Logs show stacktrace or immediate exit.
- Diagnosis:
  1. `kubectl describe pod <pod> -n <ns>` — check events and last state.
  2. `kubectl logs <pod> -c <container> -n <ns>` to view application logs.
  3. `kubectl logs <pod> -c <container> -p -n <ns>` for previous instance logs if restarted.
  4. `kubectl get pod <pod> -o yaml` to inspect `lifecycle`, `readinessProbe`, `livenessProbe`.
- Common causes:
  - Application configuration error, missing env var, wrong secret.
  - Health check misconfiguration causing premature kill.
  - Missing dependency (DB not reachable) causing app to exit.
  - Low resource limits causing OOMKilled before initialization.
- Remediation:
  1. Run container interactively: `kubectl run -it --rm debug --image=busybox -- sh` or use ephemeral debug pod.
  2. Validate ConfigMaps/Secrets: `kubectl get configmap app-config -o yaml`.
  3. Increase `initialDelaySeconds` of readiness/liveness or separate readiness and liveness logic.
  4. Fix image or env and `kubectl rollout restart deployment/web -n prod`.
- Prevention:
  - Graceful startup logic in app (backoff, retry).
  - Readiness vs liveness separation.
  - Integration tests for config & connection strings prior to release.

### Scenario: ImagePullBackOff / ErrImagePull

- Symptoms:
  - Pods pending with `ImagePullBackOff`.
  - Events show `Back-off pulling image` or `unauthorized: authentication required`.
- Diagnosis:
  1. `kubectl describe pod <pod>` to inspect event details.
  2. Validate image name/tag: `docker pull <image>` locally.
  3. Check imagePullSecrets in namespace and ServiceAccount.
  4. Check registry auth: expired token or IAM permission.
- Remediation:
  1. Correct image tag in manifest; re-apply.
  2. Update `imagePullSecrets` or service account credentials.
  3. If rate-limited (Docker hub), use mirror registry or authenticated pulls.
  4. For private registries (ECR), ensure `aws ecr get-login-password` is used in workflow or use IRSA (IAM roles for service accounts).
- Prevention:
  - CI pushes images and updates manifests in GitOps; use immutable tags (SHA).
  - Monitor registry quotas and set up credential rotation automation.

### Scenario: OOMKilled (container terminated due to OOM)

- Symptoms:
  - Pod status shows `OOMKilled`.
  - `kubectl describe pod` shows `oom_kill` in container status.
- Diagnosis:
  1. `kubectl top pod <pod> -n <ns>` to view current usage.
  2. Check `/var/log/kubelet` and app logs to see memory spike.
  3. Check node memory availability: `kubectl top node`.
- Remediation:
  1. Increase container memory limit (`resources.limits.memory`) and requests.
  2. Tune JVM/java/Go memory flags to prevent sudden growth.
  3. Use `oom_score_adj` patterns or sidecar memory limits if using multiple containers in pod.
  4. Investigate memory leaks using heap dumps and pprof.
- Prevention:
  - Set requests to a realistic baseline, set limits to avoid runaway scenarios, and use monitoring alerts for memory pressure.

### Scenario: Node NotReady after upgrade or kernel update

- Symptoms:
  - Node shows `NotReady`.
  - Many pods moved to `Pending` or rescheduled.
- Diagnosis:
  1. `kubectl describe node <node>` to see conditions & events.
  2. SSH to node; `systemctl status kubelet`, `journalctl -u kubelet -xe`.
  3. Verify runtime (`containerd`) status and disk space (`df -h`).
  4. Check `kubectl get cs` or metrics for control-plane issues.
- Remediation:
  1. Cordon node: `kubectl cordon <node>`; drain if necessary: `kubectl drain <node> --ignore-daemonsets --delete-local-data`.
  2. Fix kernel issue, restart kubelet or container runtime, or reboot host.
  3. Uncordon after validation: `kubectl uncordon <node>`.
- Prevention:
  - Pre-check upgrades in a staged pool, run node upgrade script with proper kernel modules and CRI compatibility.

---

## Production best practices checklist (operational)

- Use immutably-tagged images (semantic version + git SHA) and never rely on `latest` in production.
- Store manifests in Git and use GitOps for reconciliation (ArgoCD/Flux).
- Implement RBAC and avoid cluster-admin for application service accounts.
- Scan images in CI (Trivy/Grype) and require vulnerability policy gates.
- Use Pod Security admission policies; disallow privileged containers and hostPath unless necessary.
- Enforce resource requests & limits, and monitor via Prometheus.
- Use readiness & liveness probes to prevent unhealthy pods from receiving traffic.
- Implement PDBs for critical services to preserve availability during upgrades.
- Collect logs (Fluentd/Fluent Bit) and metrics (Prometheus) and define SLO-driven alerts.
- Backup etcd regularly and test restore; store backups offsite and version snapshots.
- Use multi-AZ deployments and StorageClass with zone-aware provisioning for HA.
- Plan DB schema migrations to be backward compatible; use migration jobs and feature flags.
- Use network policies for micro-segmentation; default deny inbound when possible.
- Keep cluster control plane patches current; test upgrades in staging.
- Use monitoring to detect noisy neighbors and overcommit situations.
- Limit API server access — use bastion host and fine-grained OIDC or cert-based auth.

---

## Interview Questions & Answers — Beginner / Intermediate / Advanced (7+ each)

### Beginner (7)
1. **Q:** What is a Pod?  
   **A:** The smallest scheduling unit in Kubernetes, representing one or more co-located containers sharing network and storage.

2. **Q:** What is a Deployment?  
   **A:** A controller that manages ReplicaSets and provides declarative updates for pods (rolling updates/rollbacks).

3. **Q:** How do Services enable service discovery?  
   **A:** Services create stable endpoints (ClusterIP) and DNS entries (CoreDNS) so pods can communicate by name.

4. **Q:** What is a ConfigMap?  
   **A:** A Kubernetes object for storing non-sensitive configuration data and injecting into pods via env vars or volume mounts.

5. **Q:** How do you view pod logs?  
   **A:** `kubectl logs <pod> [-c container] -n <ns>` and for previous instance use `-p`.

6. **Q:** What is a ReplicaSet?  
   **A:** A controller ensuring a set number of pod replicas are running (usually managed indirectly by Deployments).

7. **Q:** Why use namespaces?  
   **A:** To logically partition cluster resources and isolate teams or environments (dev/staging/prod).

### Intermediate (7)
1. **Q:** How does a readiness probe differ from a liveness probe?  
   **A:** Readiness determines if a pod should receive traffic; liveness determines whether a pod should be restarted by kubelet.

2. **Q:** What causes a pod to be `Pending`?  
   **A:** Common causes include insufficient resources, unsatisfied nodeSelector/taints, or missing PVC binding.

3. **Q:** How do you restrict network access to a pod?  
   **A:** Create `NetworkPolicy` objects defining allowed ingress/egress sources.

4. **Q:** How do you implement zero-downtime deployment?  
   **A:** Use rolling updates with readiness checks, PDBs, and canary or blue/green patterns for risk mitigation.

5. **Q:** What is the role of etcd and how do you backup it?  
   **A:** etcd stores cluster state; backup using `etcdctl snapshot save` with mTLS and store snapshots off-cluster.

6. **Q:** How do you debug an OOMKilled pod?  
   **A:** Check container logs, `kubectl describe pod`, `kubectl top pod`, inspect memory usage and adjust requests/limits or fix leaks.

7. **Q:** What is a DaemonSet and when would you use it?  
   **A:** Ensures a pod runs on all (or selected) nodes — used for node-level agents like log collectors.

### Advanced (7)
1. **Q:** How does Kubernetes scheduling work under the hood?  
   **A:** Scheduler evaluates pending pods, applies predicates (taints/tolerations, node resources), ranks nodes using priorities and binds the pod to chosen node via API server.

2. **Q:** How to perform an etcd restore in a multi-master environment?  
   **A:** Take snapshot, bring up new etcd member(s) from snapshot, reconfigure control plane to point at restored cluster, ensure correct certs and cluster IDs.

3. **Q:** How to enforce image provenance and signing?  
   **A:** Use cosign to sign images, set admission controller policies to require signed images, and verify signatures at deployment time.

4. **Q:** How to scale cluster control plane for large clusters?  
   **A:** Scale etcd with dedicated IAAS, separate control plane nodes, use apiserver horizontal scaling behind LB, optimize controller-manager concurrency, and tune etcd compaction.

5. **Q:** Explain how CNI plugins like Cilium use eBPF for networking.  
   **A:** Cilium leverages eBPF to run in-kernel packet processing, enabling faster forwarding, L7 policies, and visibility without iptables overhead.

6. **Q:** How to handle large ConfigMaps or many secrets for application configuration?  
   **A:** Use mounted volumes (config files), compress large artifacts, or externalize configuration into object storage and use sidecars or init containers to fetch them.

7. **Q:** How to debug high API server latency?  
   **A:** Inspect etcd metrics, audit logs for large writes, check API server request latencies via Prometheus metrics, and investigate heavy watchers.

---

## Quick revision notes — 20+ one-liners for interviews

1. Kubernetes is a declarative orchestration platform that manages container lifecycles and cluster state.
2. The API server is the cluster gateway — every change goes through it.
3. etcd stores cluster state and must be backed up and secured.
4. kube-scheduler decides pod placement based on resources, affinity, and taints.
5. kubelet runs on each node and manages pods & containers through the runtime.
6. Deployments manage stateless pods and provide rolling updates and rollbacks.
7. StatefulSets provide stable network IDs and persistent storage for stateful services.
8. DaemonSets ensure a pod runs on each eligible node (logging/monitoring).
9. Services provide stable endpoints (ClusterIP, NodePort, LoadBalancer).
10. Ingress handles HTTP(S) routing with a controller (nginx, Traefik).
11. NetworkPolicy enables pod-level traffic control for zero-trust networking.
12. PVCs abstract storage consumption; StorageClass defines dynamic provisioning.
13. Readiness probes control traffic; liveness probes control restarts.
14. PodDisruptionBudget prevents too many pods from being disrupted during maintenance.
15. Horizontal Pod Autoscaler scales pods based on metrics (CPU, custom).
16. Cluster Autoscaler adds/removes nodes based on unschedulable pods.
17. Use GitOps (ArgoCD/Flux) to make manifests the source of truth.
18. Image scanning (Trivy/Grype) is essential in CI to maintain supply chain security.
19. Secrets should be externalized and never committed to Git in plaintext.
20. Monitor via Prometheus/Grafana; aggregate logs via FluentBit/Fluentd to Elasticsearch/Loki.
21. Use Pod Security admission and OPA Gatekeeper to enforce security policies.
22. Use immutable image tags and record deployed image SHAs for reproducibility.
23. Test DR by restoring etcd snapshots in staging regularly.

---

## Further reading & references

- Official Kubernetes docs — https://kubernetes.io/docs/  
- Prometheus operator / kube-prometheus-stack — https://github.com/prometheus-operator/kube-prometheus  
- ArgoCD — https://argo-cd.readthedocs.io  
- OPA Gatekeeper — https://open-policy-agent.github.io/gatekeeper/  
- CNCF production tooling and best-practices guides

---

**End of file — kubernetes/kubernetes-for-devops.md**  
