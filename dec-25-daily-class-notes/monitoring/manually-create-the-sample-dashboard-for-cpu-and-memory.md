# manually create the  sample dashboard for cpu and Memory

Here’s a simple, **manually created**, Grafana dashboard example for **CPU, memory, and “other” metrics** using Prometheus (e.g., node-exporter or Kubernetes-style metrics). You can build this step-by-step directly in Grafana.

***

### 1. Dashboard structure (4 panels)

You’ll create a dashboard with:

* Panel 1: Node CPU usage (%)
* Panel 2: Node memory usage (%)
* Panel 3: Node disk usage (%)
* Panel 4: Node load average (1-min)

Assume your Prometheus data source is already added in Grafana.

***

### 2. Manual creation steps (Grafana UI)

{% stepper %}
{% step %}
### Create a new dashboard

In Grafana: click **+ → Dashboard → Create dashboard → Add new panel**.
{% endstep %}

{% step %}
### Configure the panel

In the **Query** tab:

* Select your **Prometheus** data source.
* Choose the **PromQL** editor.
{% endstep %}
{% endstepper %}

***

### 3. Sample PromQL queries for each panel

Paste these queries into separate panels and set visualization as **Time series** (or **Gauge** for single-value style).

#### Panel 1 – CPU usage (%) (node-exporter)

```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

* Visualization: **Time series** or **Gauge**; title: `CPU Usage (%)`.

#### Panel 2 – Memory usage (%) (node-exporter)

```promql
(
  node_memory_MemTotal_bytes - node_memory_MemFree_bytes
  - node_memory_Buffers_bytes - node_memory_Cached_bytes
) / node_memory_MemTotal_bytes * 100
```

* Visualization: **Time series** or **Gauge**; title: `Memory Usage (%)`.

#### Panel 3 – Disk usage (%) (filesystem)

```promql
(
  node_filesystem_size_bytes{mountpoint="/"}
  - node_filesystem_free_bytes{mountpoint="/"}
) / node_filesystem_size_bytes{mountpoint="/"} * 100
```

* Visualization: **Time series** or **Gauge**; title: `Root Disk Usage (%)`.
* Adjust `mountpoint` if you want `/var`, `/home`, etc.

#### Panel 4 – Load average (1-min)

```promql
node_load1
```

* Visualization: **Time series**; title: `Load 1-min`.

After each panel:

* Click **Apply** → **Save dashboard**.
* Arrange panels by dragging them into a grid (e.g., 2×2 layout).

***

### 4. For Kubernetes workloads (optional)

If you want to monitor **Kubernetes pods** instead of bare nodes:

* CPU usage (pod-level, approx.):

```promql
sum by (pod, namespace) (rate(container_cpu_usage_seconds_total{container!=""}[5m])) * 100
```

* Memory usage (working-set):

```promql
sum by (pod, namespace) (container_memory_working_set_bytes{container!=""})
/ (sum by (pod, namespace) (machine_memory_bytes))
* 100 setting
```

Visualize these as **Time series** with **by (pod, namespace)** grouping.

***

If you tell me your exact stack (bare metal, Kubernetes, Docker, or kube-prometheus-stack), I can give you a **complete panel list** with exact labels and a ready-to-paste layout you can follow step-by-step.
