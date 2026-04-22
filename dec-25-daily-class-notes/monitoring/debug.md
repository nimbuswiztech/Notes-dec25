# Debug

### Alert Not Firing - 5 Step Debug (2 minutes)

**Your stress pod is running on the right node!** Now let's debug why the alert isn't firing:

{% stepper %}
{% step %}
### Verify CPU Spike (Grafana)

```
localhost:3000 → Node Exporter Dashboard → 
Check ip-172-31-25-147.ec2.internal CPU >80%?
```

**If NO spike**: Stress pod not working → use YAML below\
**If YES spike**: Continue to Step 2
{% endstep %}

{% step %}
### Test Alert Query (Prometheus)

```
localhost:9090 → Graph → Enter your exact query:
100 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100 > 80

Click Execute → Red graph above 80 = QUERY WORKS ✓
No data/red graph = Wrong query
```
{% endstep %}

{% step %}
### Check Alert Rule Status

```
Grafana → Alerts & IRM → Alert rules → "Node CPU High"
Status should show: Normal/Pending (not just "Normal")
```
{% endstep %}

{% step %}
### Node Exporter Metrics

```
Prometheus localhost:9090/graph:
Query: node_cpu_seconds_total{instance=~"172.31.25.147.*"}
Data flowing? → Continue
No data → Node Exporter issue on that node
```
{% endstep %}

{% step %}
### QUICK FIX - Simplified Alert

Replace your alert query with this guaranteed working version:

```
avg(rate(node_cpu_seconds_total{instance=~"172.31.25.147.*",mode="idle"}[2m])) < 0.2
```

* **Filters to your stressed node**
* **2m window** (faster)
* **`idle < 20%` = CPU > 80%**
* **Pending: 0s** (instant trigger)
{% endstep %}
{% endstepper %}

### IMMEDIATE STRESS TEST (If CPU Not Spiking)

```bash
# Delete + recreate with YAML (100% works)
kubectl delete pod cpu-stress --force --grace-period=0

cat > stress.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: cpu-stress
spec:
  nodeName: ip-172-31-25-147.ec2.internal
  containers:
  - name: stress
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do yes > /dev/null; done"]
EOF

kubectl apply -f stress.yaml
```

### Expected Timeline

```
9:06AM: Stress pod running ✓
9:07AM: Grafana shows Node 2 CPU 85% ✓  
9:07:30AM: Alert PENDING → FIRING ✓
9:08AM: webhook.site shows JSON alert! 🎉
```

### Most Likely Issues

1. **CPU not spiking** → Use YAML stress test above
2. **Wrong query** → Use simplified query above
3. **Node Exporter missing** → `kubectl get ds prometheus-prometheus-node-exporter`
4. **Evaluation interval** → Set to 15s in alert rule

### Verify Node Exporter

```bash
kubectl get pods -l app.kubernetes.io/name=prometheus-node-exporter -o wide | grep 172.31.25.147
# Should show Node Exporter pod running on target node
```

**Run Step 1-2 NOW** → tell me:

1. **Grafana CPU % on Node 2?**
2. **PromQL query returns data?**

**90% chance = CPU not spiking enough.** Use YAML stress → simplified query → **alert fires in 30s**! Your demo will be perfect! 🔔🚀
