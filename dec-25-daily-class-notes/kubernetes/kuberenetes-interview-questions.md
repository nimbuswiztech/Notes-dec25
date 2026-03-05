# Kuberenetes Interview questions

<details>

<summary>1. What concepts will you adopt to keep your application highly available?</summary>

High-level practices to achieve high availability:

* Run multiple replicas of each pod (Deployments / ReplicaSets) across nodes.
* Use Deployments with rolling updates and readiness probes so only healthy pods receive traffic.
* Spread pods across nodes/zones using nodeSelectors, nodeAffinity and podAntiAffinity to avoid single-node/zone failure.
* Use Services (ClusterIP/LoadBalancer) and Ingress with multiple backend endpoints and health checks.
* Use StatefulSets for stateful components with stable identities and persistent volumes.
* Ensure persistent storage is replicated/available (cloud-managed storage classes, or replicated PV solutions).
* Use multiple control-plane components (HA control plane) and etcd backups/replication.
* Implement automated health checks, alerts, and runbooks / readiness for failover.
* Use resource limits/requests and Pod Disruption Budgets to limit voluntary disruptions.

</details>

<details>

<summary>2. What default namespaces are created as part of Kubernetes initial setup? Why attach a ResourceQuota to a namespace?</summary>

Default namespaces created by Kubernetes:

* default
* kube-system
* kube-public
* kube-node-lease

Why attach a ResourceQuota to a namespace:

* Enforce limits on resource consumption (CPU, memory, storage, object counts) per namespace.
* Prevent a single team/app from consuming cluster resources and affecting others.
* Provide accounting and quota enforcement for multi-tenant clusters.
* Control and guarantee resource availability for critical workloads.

</details>

<details>

<summary>3. What volume types are available in Kubernetes and which one have you used in your project? Have you used PersistentVolume?</summary>

Common volume types in Kubernetes:

* emptyDir
* hostPath
* configMap / secret (as volume)
* persistentVolumeClaim (PVC) -> PersistentVolume (PV)
* nfs
* awsElasticBlockStore, gcePersistentDisk, azureDisk (cloud provider block storage)
* CSI-backed volumes (any storage with CSI driver)
* ephemeral volumes (ephemeral inline volumes, ephemeral PVCs)

PersistentVolume / PVC:

* Use PersistentVolumes (with PersistentVolumeClaims) for durable data that must survive pod restarts or rescheduling.
* PVCs decouple storage provisioning from pods; use StorageClasses for dynamic provisioning.

(Choose the PV type appropriate to your environment — cloud block storage, NFS, or CSI drivers — and use PVCs for persistence.)

</details>

<details>

<summary>4. Have you ever used adapter containers? If yes, what was the situation?</summary>

Adapter (sidecar) containers are commonly used patterns:

* Used to handle cross-cutting concerns without modifying main app container (examples: logging/sidecar, proxy, metrics exporter, init/backup).
* Typical situations:
  * Sidecar log forwarder shipping logs to central system.
  * Service mesh sidecars (Envoy) for networking/observability.
  * Init containers preparing data or waiting for dependencies before main container starts.

(Use sidecars when you need to extend or support the main container’s behavior in a decoupled way.)

</details>

<details>

<summary>5. While working with ConfigMap did you face any issues copying files to a folder which already has files?</summary>

Behavior and common issue:

* When mounting a ConfigMap as a volume into a container path, the mount hides the existing contents of the target directory — it does not merge. That can appear as "files disappeared". Workarounds:
* Mount ConfigMap into a subdirectory, not directly over a folder with existing files.
* Use an initContainer to copy files from the ConfigMap mount into the desired directory, preserving existing files.
* Alternatively, use files injected into the image or use an application-level mechanism to read multiple locations.

</details>

<details>

<summary>6. Example: I have a database container which should accept requests only from a particular IP or from a specific pod — how would you configure it?</summary>

Options to restrict access:

* NetworkPolicy (pod-to-pod):
  * Create a NetworkPolicy that allows ingress to the DB pod only from pods matching specific labels (namespace and podSelector) and denies other pod traffic. Requires a network plugin that supports NetworkPolicy.
* Service + client IP restriction:
  * If restricting by external IP, use cloud firewall/security group or Kubernetes Ingress/LoadBalancer source-range settings to allow a particular IP.
* Pod-level firewall (iptables) inside pod host:
  * Not recommended generally; prefer network policies or cloud firewalls.
* Service Mesh:
  * Use mTLS/authz policies in a service mesh (Istio, Linkerd) to restrict which services can call the DB.
* Application-level:
  * Configure DB authentication to only permit connections from certain IPs or require client certificates.

Choose NetworkPolicy for pod-to-pod restrictions; cloud/network firewalls or Service externalTrafficPolicy for external IP restrictions.

</details>

<details>

<summary>7. What types of scaling methods have you used?</summary>

Common scaling approaches:

* Horizontal Pod Autoscaler (HPA): scale number of pod replicas based on CPU/memory or custom metrics.
* Vertical Pod Autoscaler (VPA): adjust pod resource requests/limits (caution for downtime with certain modes).
* Cluster Autoscaler: scale worker nodes up/down based on pending pods and utilization.
* Manual scaling: kubectl scale / editing Deployment replica count for controlled changes.
* Cron scaling / scheduled scaling: scale up/down on schedules for predictable load patterns.

Combine HPA with Cluster Autoscaler so pods can be scheduled when HPA increases replicas.

</details>

<details>

<summary>8. In which situations might a pod go to Pending status? If pod status is Running but Ready column shows 0/1 what does that mean?</summary>

Pod Pending reasons:

* No available nodes with sufficient resources (CPU/memory).
* NodeSelector, nodeAffinity, taints/tolerations causing no matching node.
* PVC not bound or storage provisioning failing => pod waiting for volume.
* Image pull failures can sometimes delay scheduling/start.
* Scheduler constraints (podAntiAffinity, topologySpread) preventing placement.

Running but Ready 0/1 meaning:

* Pod is running (container process started) but readiness probe is failing — the container is not yet ready to serve traffic.
* Could be because initialization not complete, application not healthy, misconfigured readiness probe, or dependent service unreachable.
* Liveness probe might be different; readiness determines Service endpoints.

</details>

<details>

<summary>9. I have an Elasticsearch pod that must run on a particular node and that node should not accept any other pod — how would you configure it?</summary>

To ensure Elasticsearch runs on a specific node and reserve that node exclusively:

* nodeAffinity / nodeSelector:
  * Add nodeSelector or nodeAffinity on the Elasticsearch pod to schedule it onto the specific node (use labels on node).
* Make the node unschedulable for other pods:
  * Add a unique taint to the node (kubectl taint nodes key=value:NoSchedule).
  * Add a toleration in the Elasticsearch pod spec for that taint so only it (and pods with that toleration) can run there.
* Combined pattern:
  * Label the node (e.g., role=es-primary).
  * Taint the node to block general scheduling.
  * Add nodeAffinity/toleration to Elasticsearch pod so it schedules on that node and tolerates the taint.

This ensures the pod lands on the target node and other pods are prevented from scheduling there.

</details>

<details>

<summary>10. Do you know PodDisruptionBudgets?</summary>

Yes — PodDisruptionBudget (PDB):

* A PDB limits voluntary disruptions (evictions) for a set of pods (e.g., during node drains, upgrades).
* You define minAvailable (number or percentage) or maxUnavailable to ensure a minimum number of pods remain running.
* PDBs do not protect against involuntary disruptions (node crashes); they only affect evictions initiated by cluster operators/components.
* Useful for stateful or critical workloads to maintain availability during cluster maintenance.

</details>
