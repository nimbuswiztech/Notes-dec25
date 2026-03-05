# Taints and Tolerations

Node affinity is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite — they allow a node to repel a set of pods.

Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

```
kubectl taint nodes node1 key1=value1:NoSchedule
```

places a taint on node node1. The taint has key key1, value value1, and taint effect NoSchedule. This means that no pod will be able to schedule onto node1 unless it has a matching toleration.

To remove the taint added by the command above, you can run:

```
kubectl taint nodes node1 key1=value1:NoSchedule-
```

You specify a toleration for a pod in the PodSpec. Both of the following tolerations "match" the taint created by the kubectl taint line above, and thus a pod with either toleration would be able to schedule onto node1:

```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

```
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```

Here's an example of a pod that uses tolerations:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

The default value for operator is Equal.

A toleration "matches" a taint if the keys are the same and the effects are the same, and:

* the operator is Exists (in which case no value should be specified), or
* the operator is Equal and the values are equal.

The above example used effect of NoSchedule. Alternatively, you can use effect of PreferNoSchedule. This is a "preference" or "soft" version of NoSchedule — the system will try to avoid placing a pod that does not tolerate the taint on the node, but it is not required. The third kind of effect is NoExecute, described later.

You can put multiple taints on the same node and multiple tolerations on the same pod. The way Kubernetes processes multiple taints and tolerations is like a filter: start with all of a node's taints, then ignore the ones for which the pod has a matching toleration; the remaining un-ignored taints have the indicated effects on the pod. In particular,

* if there is at least one un-ignored taint with effect **NoSchedule** then Kubernetes will not schedule the pod onto that node
* if there is no un-ignored taint with effect NoSchedule but there is at least one un-ignored taint with effect **PreferNoSchedule** then Kubernetes will try to not schedule the pod onto the node
* if there is at least one un-ignored taint with effect **NoExecute** then the pod will be evicted from the node (if it is already running on the node), and will not be scheduled onto the node (if it is not yet running on the node).

The node controller automatically taints a Node when certain conditions are true. The following taints are built in:

* `node.kubernetes.io/not-ready:` Node is not ready. This corresponds to the NodeCondition Ready being "False".
* `node.kubernetes.io/unreachable:` Node is unreachable from the node controller. This corresponds to the NodeCondition Ready being "Unknown".
* `node.kubernetes.io/out-of-disk:` Node becomes out of disk.
* `node.kubernetes.io/memory-pressure:` Node has memory pressure.
* `node.kubernetes.io/disk-pressure:` Node has disk pressure.
* `node.kubernetes.io/network-unavailable:` Node's network is unavailable.
* `node.kubernetes.io/unschedulable:` Node is unschedulable.
* `node.cloudprovider.kubernetes.io/uninitialized:` When the kubelet is started with "external" cloud provider, this taint is set on a node to mark it as unusable. After a controller from the cloud-controller-manager initializes this node, the kubelet removes this taint.

To get list of taints use below command

```
kubectl get nodes -o go-template='{{printf "%-50s %-12s\n" "Node" "Taint"}}{{- range .items}}{{- if $taint := (index .spec "taints") }}{{- .metadata.name }}{{ "\t" }}{{- range $taint }}{{- .key }}={{ .value }}:{{ .effect }}{{ "\t" }}{{- end }}{{- "\n" }}{{- end}}{{- end}}'
```
