# Best practices

✅ Use Declarative YAML (not `kubectl run`)

* Define everything as code (Deployment, Service, Ingress, PVC, etc.).
* Use Git to version-control your YAML files (GitOps style).

***

✅ Prefer Deployments over Pods

* Deployments manage rolling updates and restarts automatically.
* Pods alone don’t self-heal or scale.

***

✅ Use Liveness and Readiness Probes

* Readiness: When the app is ready to accept traffic.
* Liveness: When the app needs a restart (e.g. stuck).

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

***

✅ Use ConfigMaps and Secrets

* ConfigMaps: Non-sensitive config like feature flags, environment variables.
* Secrets: API keys, passwords (base64-encoded, but ideally use external secret managers too).
* Don’t bake secrets into images or hardcode configs.

***

✅ Run containers as non-root

```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
```

***

✅ Use RBAC

* Give minimum permissions needed (principle of least privilege).
* Use roles per namespace and service accounts per workload.

***

✅ Network Policies

* Control which pods can talk to which — like firewalls for pods.

```yaml
podSelector:
  matchLabels:
    role: frontend
```

***

✅ Use Horizontal Pod Autoscaling (HPA)

```yaml
minReplicas: 2
maxReplicas: 10
metrics:
- type: Resource
  resource:
    name: cpu
    targetAverageUtilization: 70
```

***

✅ Spread workloads across Availability Zones

* Use topologySpreadConstraints or multiple node groups.

***

<details>

<summary>✅ Use Readiness Gates with external systems (optional)</summary>

If your app depends on an external service, don’t mark it ready until it's usable.

</details>

***

✅ Centralize logs (don’t `kubectl logs` everything) Use logging stacks such as:

* Fluentd / Fluent Bit → Elasticsearch + Kibana (EFK)
* Loki + Grafana
* Cloud-native logging like CloudWatch or Stackdriver

***

✅ Use Prometheus + Grafana

* For metrics, alerts, and dashboards.

***

✅ Set Resource Requests and Limits

* Without them, your app can hog CPU/memory or be evicted.

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"
```

***

✅ Use small base images (Alpine, Distroless)

* Reduces image size and attack surface.

***

✅ Tag images uniquely (e.g., v1.0.2 not latest)

* Helps in rolling back and avoiding caching issues.

***

✅ Use CI/CD pipelines (e.g., GitHub Actions, ArgoCD, Flux)

* Automate testing, building, and deploying.

***

✅ Use Namespaces to isolate environments

* Isolate dev, staging, prod using namespaces.

***

✅ Enable PodDisruptionBudgets (PDBs)

* Prevent all pods from being evicted during upgrades.
