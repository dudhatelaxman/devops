# Kubernetes for DevOps — Comprehensive Guide

Version: 1.0
Last updated: 2025-12-29

This guide targets DevOps engineers working with Kubernetes in production. It covers architecture, commands, common manifests, troubleshooting, CI/CD and GitOps, security, observability, interview Q&A and revision notes.

---

## Table of Contents

- Architecture (ASCII)
- kubectl command quick reference
- Common manifests (examples)
- Troubleshooting scenarios & runbook
- CI/CD and GitOps
- Security best practices
- Observability & logging
- Interview Q&A
- Revision notes & further reading

---

## Architecture (ASCII)

A concise ASCII diagram showing components and interactions.

Control plane & worker layout:

```
                     +---------------------------+
                     |       Load Balancer       |
                     +------------+--------------+
                                  |
                    +-------------v-------------+
                    |      API Server (kube-apiserver)
                    +------+--+--+-------------+
                           |  |  |             
               +-----------+  |  +-----------+
               |              |              |
     +---------v-----+  +-----v------+  +----v---------+
     | kube-scheduler|  | controller |  | etcd cluster |
     |               |  | manager    |  | (state)      |
     +---------------+  +------------+  +--------------+

                 Control Plane (masters) - Highly Available


                +-------------------------------------+
                |             Worker Nodes           |
                |  +-----------+    +-----------+     |
                |  | kubelet   |    | kubelet   |     |
                |  | kube-proxy|    | kube-proxy|     |
                |  | containerd|    | containerd|     |
                |  +---+-------+    +---+-------+     |
                |      |                |            |
                |  +---v---+        +---v---+        |
                |  | Pod   |        | Pod   |        |
                |  | (app) |        | (app) |        |
                |  +-------+        +-------+        |
                +-------------------------------------+

External systems:
- CI/CD (GitHub Actions, GitLab CI, Jenkins)
- GitOps (ArgoCD, Flux)
- Monitoring (Prometheus, Grafana)
- Logging (EFK/Opensearch)
- Service Mesh (Istio/Linkerd)

```

Notes:
- etcd stores cluster state; secure and backup.
- API Server is the gateway for all writes/reads.
- Controller Manager and Scheduler make decisions and ensure desired state.
- kubelet runs on each node to manage pods.

---

## kubectl command quick reference

A compact cheat sheet for everyday tasks.

General:

- kubectl version                # show client & server version
- kubectl cluster-info          # cluster info
- kubectl get nodes             # list nodes
- kubectl get pods -A           # list all pods in all namespaces
- kubectl get svc               # list services
- kubectl get deployments       # list deployments

Inspect resources:

- kubectl describe pod/<name> -n <ns>
- kubectl logs pod/<name> -c <container> -n <ns>
- kubectl exec -it <pod> -n <ns> -- /bin/sh
- kubectl get events -n <ns>

Create/Apply:

- kubectl apply -f <file.yaml>
- kubectl delete -f <file.yaml>
- kubectl replace -f <file.yaml>

Diff & dry-run:

- kubectl diff -f <file.yaml>
- kubectl apply --server-dry-run -f <file.yaml>

Scale & rollout:

- kubectl scale --replicas=3 deployment/<name>
- kubectl rollout status deployment/<name>
- kubectl rollout undo deployment/<name>

Namespace & context:

- kubectl get ns
- kubectl config get-contexts
- kubectl config use-context <context>
- kubectl config set-context --current --namespace=<ns>

Port-forward & proxy:

- kubectl port-forward svc/<service> 8080:80 -n <ns>
- kubectl proxy --port=8001

Top & metrics:

- kubectl top nodes
- kubectl top pod -n <ns>

Advanced debugging:

- kubectl cp <pod>:/path /local/path -n <ns>
- kubectl auth can-i create pods --as system:serviceaccount:<ns>:<sa>
- kubectl get --raw "/api/v1/namespaces/<ns>/pods/<pod>/proxy/"

---

## Common manifests (examples)

Below are concise, ready-to-use examples. Adapt image names, namespaces, labels, and resource limits.

1) Deployment + Service (ClusterIP)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.25
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: "500m"
              memory: "256Mi"
            requests:
              cpu: "100m"
              memory: "128Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: web-svc
  namespace: default
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

2) Ingress (using ingress-nginx)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: example.k8s.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80
```

3) ConfigMap & Secret

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_MODE: "production"
  LOG_LEVEL: "info"
---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: default
type: Opaque
stringData:
  DB_USER: admin
  DB_PASS: s3cr3t
```

4) StatefulSet (example for MySQL/Postgres)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pg
  namespace: default
spec:
  serviceName: pg-svc
  replicas: 2
  selector:
    matchLabels:
      app: pg
  template:
    metadata:
      labels:
        app: pg
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
          envFrom:
            - secretRef:
                name: db-secret
  volumeClaimTemplates:
    - metadata:
        name: pgdata
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 10Gi
```

5) DaemonSet (node exporter)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostPID: true
      containers:
        - name: node-exporter
          image: prom/node-exporter:1.6.0
          ports:
            - containerPort: 9100
          volumeMounts:
            - name: proc
              mountPath: /host/proc
              readOnly: true
      volumes:
        - name: proc
          hostPath:
            path: /proc
```

6) Job & CronJob

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  namespace: default
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: myorg/migrate:latest
      restartPolicy: Never
  backoffLimit: 4
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
  namespace: default
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: myorg/backup:stable
          restartPolicy: OnFailure
```

7) NetworkPolicy (restrict traffic)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

8) PodDisruptionBudget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
  namespace: default
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```

---

## Troubleshooting scenarios & runbook

Common failure modes and steps to diagnose and remediate.

1) Pods CrashLoopBackOff
- kubectl describe pod/<pod> -n <ns>
- kubectl logs <pod> -c <container> -n <ns>
- If init containers failing, check init container logs
- Check resource limits/requests — OOMKilled indicates memory pressure
- kubectl get events -n <ns>
- If image pull error: check image name and registry credentials (imagePullSecrets)

Remediation checklist:
- Fix application error or config
- Increase resources if OOM
- Add readiness/liveness probes to avoid serving traffic prematurely

2) Node NotReady
- kubectl describe node/<node>
- Check kubelet logs on the node (journalctl -u kubelet)
- Check disk pressure: df -h, check inode exhaustion
- Check network (CNI) status (e.g., calico/flannel pods)
- Restart kubelet if necessary

3) High API Server Latency
- kubectl top apiserver (if available) or monitor metrics
- Check etcd health: etcdctl --endpoints=... endpoint status
- Check API server logs for errors
- Check audit logs for high-volume requests

4) CrashLoop/Pod Scheduling Fails (Pending)
- kubectl describe pod/<pod> shows events; common reasons: Unschedulable (insufficient cpu/memory), nodeSelector/taints
- kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}:{.status.allocatable.cpu},{.status.allocatable.memory}\n{end}'
- Check taints: kubectl describe node <node> | grep Taints
- Consider adding resources or adjusting requests

5) Service unreachable
- kubectl get endpoints <svc> -n <ns> to see backing pods
- kubectl describe svc/<svc>
- Validate port mapping and targetPort
- For ClusterIP: try curl from BusyBox pod
- For DNS issues: kubectl exec -it <pod> -- nslookup <service> or dig

6) DNS issues (CoreDNS)
- kubectl get pods -n kube-system -l k8s-app=kube-dns
- kubectl logs -n kube-system <coredns-pod>
- Verify /etc/resolv.conf within pod
- Restart coredns deployment if necessary

7) PersistentVolume/PVC Problems
- kubectl describe pvc <pvc> -n <ns>
- kubectl get pv
- If pending, check storage class and provisioner
- Check cloud provider quotas and node-to-PV binding

8) Performance bottleneck
- kubectl top nodes / pods
- Use Prometheus metrics and flamegraphs
- Profiling app CPU/memory
- Check disk I/O, network bandwidth, and latency

Runbook tips:
- Maintain a minimal list of kubectl commands for emergencies (get pods, describe, logs, exec)
- Use namespaces to limit blast radius
- Keep kubectl access controlled and audit logs enabled

---

## CI/CD and GitOps

Patterns:
- CI: build artifacts -> run tests -> push images to registry
- CD: deploy images to clusters (manual gate or automated)
- GitOps: desired manifests stored in Git; a reconciler (ArgoCD/Flux) ensures cluster matches Git.

Example: GitHub Actions workflow snippet (build & push + manifest update)

```yaml
name: CI
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: ghcr.io/myorg/myapp:${{ github.sha }}
      - name: Update manifests
        run: |
          sed -i "s~image: myorg/myapp:.*~image: ghcr.io/myorg/myapp:${{ github.sha }}~" k8s/deployment.yaml
      - name: Commit manifests
        run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git add k8s/deployment.yaml
          git commit -m "ci: update image to ${{ github.sha }}" || echo "no changes"
          git push
```

GitOps flow (ArgoCD/Flux):
- Store canonical manifests or Kustomize/Helm chart in a Git repo
- Install ArgoCD in cluster and point to repo & path
- ArgoCD pulls and applies changes; provides UI and health checks
- Use App-of-Apps pattern for multi-cluster

Blue/Green and Canary:
- Use tools: Argo Rollouts, Flagger, Istio + VirtualServices
- Canary example: shift traffic gradually and promote on success

Security gating:
- Run image scanning (Trivy, Clair) in CI
- Use admission controllers (OPA Gatekeeper) to deny unsafe manifests

Secrets management:
- Avoid storing plain secrets in Git
- Use SealedSecrets, External Secrets, or Vault integration

---

## Security best practices

Cluster hardening checklist:

- RBAC:
  - Use least privilege for service accounts
  - Avoid cluster-admin for apps
  - Example: create Role and RoleBinding for namespace-scoped access

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: read-pods
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get","list","watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: myapp-sa
    namespace: default
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io
```

- Admission control:
  - Use Pod Security Admission (PSA) to enforce baseline/restricted policies
  - Use OPA Gatekeeper or Kyverno for policy-as-code

- Network policies:
  - Use NetworkPolicy to restrict pod-to-pod communication

- Image security:
  - Use minimal base images, scan images, sign images (cosign)
  - Run images as non-root, set read-only root filesystem when possible

- Secrets:
  - Use KMS-backed secrets (AWS KMS, GCP KMS), ExternalSecrets, or Vault
  - Use SealedSecrets for safe Git storage

- etcd encryption & backups:
  - Enable etcd encryption at rest for secrets
  - Regular etcd backups and restore drills

- Audit logging:
  - Enable API Server audit logs and ship to a central store

- Node hardening:
  - Ensure nodes are patched, use CIS benchmarks, limit SSH access
  - Use runtime security (Falco) to detect suspicious behavior

- Supply chain security:
  - Reproducible builds, lock dependency versions
  - Use SBOMs, code signing, provenance

---

## Observability & logging

Metrics:
- Use Prometheus to scrape kube-state-metrics, node-exporter, cAdvisor, and application metrics.
- Grafana for dashboards. Use Alertmanager for alerts.

Tracing:
- Use OpenTelemetry or Jaeger for distributed tracing.
- Instrument applications to emit traces and correlate with logs.

Logging:
- Centralized logging: EFK (Elasticsearch/Fluentd/Kibana) or Loki/Promtail/Grafana.
- Ensure logs are structured (JSON) and include trace/span IDs.

Examples:
- Prometheus operator / kube-prometheus-stack simplifies setup.
- Alerting rules to watch for OOM, node disk pressure, high restart count, high API latency.

Debugging with observability:
- Correlate pod logs with traces and metrics to locate latency sources.
- Use flamegraphs and pprof to find hot paths.

SLOs and SLIs:
- Define key SLIs (availability, latency), set SLOs and error budgets.
- Integrate with alerting to avoid noisy alerts.

---

## Interview Q&A (concise)

Q: What is the difference between Deployment and StatefulSet?
A: Deployment is for stateless workloads with interchangeable pods and scaling; StatefulSet manages stateful apps providing stable network IDs and ordered, stable storage with persistent identity and ordering guarantees.

Q: How does Kubernetes service discovery work?
A: Services create DNS records (CoreDNS) and ClusterIP. kube-proxy configures packet forwarding (iptables/ipvs). DNS maps service name to ClusterIP.

Q: Explain liveness vs readiness probes.
A: Liveness probe determines if a container should be restarted. Readiness probe indicates whether the container should receive traffic. A failing readiness probe removes pod from service endpoints.

Q: How to secure access to Kubernetes API?
A: Use TLS, RBAC, API server flags (authorization, admission), audit logs, network ACLs, and restrict access via bastion/jump hosts.

Q: What is an admission controller?
A: A plugin that intercepts requests to the API server after authentication and authorization; can mutate or validate requests (e.g., PodSecurityAdmission, OPA Gatekeeper).

Q: How do you perform a zero-downtime deploy?
A: Use readiness probes, rolling updates, appropriate pod disruption budgets, and canary/blue-green techniques.

Q: How to debug network connectivity between pods?
A: Use kubectl exec to a debug pod, tcpdump, check NetworkPolicy, inspect CNI plugin logs and kube-proxy.

---

## Revision notes & further reading

Quick revision checklist:
- Understand control plane vs nodes
- Know kubectl basics (get/describe/logs/exec/apply)
- Be able to write/Edit simple manifests (Deployment/Service/Ingress)
- Know troubleshooting steps for pods, nodes, and networking
- Understand CICD patterns, GitOps, and rollouts
- Familiar with RBAC, NetworkPolicy, secrets handling
- Know monitoring basics: Prometheus, Grafana, logging

Further reading:
- Kubernetes official docs: https://kubernetes.io/docs/
- Production-Grade Kubernetes: guides & CNCF resources
- Argo CD / Flux docs for GitOps patterns
- Prometheus & Grafana docs
- OPA Gatekeeper / Kyverno for policy-as-code

---

Appendix: sample commands for emergency access

- kubectl get pods -A
- kubectl describe pod <pod> -n <ns>
- kubectl logs -f <pod> -n <ns>
- kubectl exec -it <pod> -n <ns> -- /bin/sh
- kubectl get events -n <ns>


---

End of document.
