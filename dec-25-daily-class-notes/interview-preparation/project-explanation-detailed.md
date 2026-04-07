# project explanation detailed

## Project Overview — Interview Version

In my current role, I manage **Java-based microservices** running on a **Kubernetes cluster** deployed across multiple AWS Availability Zones. The architecture is designed for **high availability**, **fault tolerance**, and **secure operations**.

## Cluster & Workload Design

* **Node setup:** 10–15 worker nodes split into ASGs, using `m7i.4xlarge` for heavy workloads and `t3.medium` for lighter ones.
* **Pods:** We run 25+ microservices, each deployed as **Kubernetes pods** with 3–5 replicas depending on environment and demand.
* **Resilience methods:**
  * **Pod anti-affinity:** Avoids placing replicas in the same zone.
  * **Resource requests/limits:** Prevents noisy-neighbour issues.
  * **Liveness/Readiness probes:** Ensures only healthy pods serve traffic.
  * **Taints & tolerations:** Critical workloads get dedicated nodes.
* **Scaling:** Both Horizontal Pod Autoscaler (HPA) + Cluster Autoscaler handle fluctuations.

## Networking & Communication

* **Ingress management:** NGINX Ingress Controller integrated with AWS ALB.
* **Service types:** Internal services use ClusterIP; external ones use Ingress + ALB.
* **Network policies:** Strict rules on pod-to-pod communication for security.

## Security & Secrets Management

* **AWS Secrets Manager** for centralized secrets in production.
* **HashiCorp Vault** for encrypted service-to-service communication in certain projects.
* **AWS KMS** for encryption at rest.
* **RBAC policies**: Ensures least privilege across teams.

{% hint style="info" %}
Cross-question alert: Be ready to explain _why_ you chose Vault over Secrets Manager in certain cases — e.g., advanced secret rotation & dynamic secrets.
{% endhint %}

## Observability & Monitoring

* **Logging:** Centralized logging with EFK stack (Elasticsearch, Fluentd, Kibana).
* **Monitoring:** Prometheus for metrics, Grafana for dashboards, Alertmanager for proactive incident response.
* **Dashboards:** Separate views per environment.

## Multi-environment Approach

We maintain **three** key environments: Dev, Staging, Production.

* Each environment runs in its own **Kubernetes namespace**.
* Separate AWS accounts for staging and production to isolate workloads.
* Environment-specific configurations through **Helm charts** & values.yaml overrides.

## CI/CD Pipelines

* **Tool:** Jenkins pipelines for builds, tests & deployments.
* Each environment has its own pipeline branch with:
  * Automated deployment via Helm.
  * Separate credentials/secrets scoped to that environment.
  * Promotion flow from Staging → Production after approvals.
* **Release strategy:** Blue-green deployments for production, rolling updates for dev/staging.

{% hint style="info" %}
Cross-question alert: Be prepared for “How do you ensure one environment’s changes don’t affect another?” Answer: Use isolated namespaces/accounts, environment-specific values, and gated approvals.
{% endhint %}

## Cross-question Readiness Checklist

1. **High Availability:** How multi-AZ design helps against zone failures.
2. **Scaling Decisions:** Difference between HPA & Cluster Autoscaler.
3. **Secret Rotation:** How Vault handles rotating DB credentials without downtime.
4. **Logging Storage:** How Elasticsearch cluster scales & retention strategy.
5. **Pipeline Failures:** How you rollback when a deployment fails.
