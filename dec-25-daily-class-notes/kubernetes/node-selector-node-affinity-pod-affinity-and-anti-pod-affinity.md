# Node Selector, Node Affinity, Pod Affinity and Anti pod Affinity

You can constrain a Pod to only be able to run on particular Node(s), or to prefer to run on particular nodes. There are several ways to do this, and the recommended approaches all use label selectors to make the selection. Generally such constraints are unnecessary, as the scheduler will automatically do a reasonable placement (e.g. spread your pods across nodes, not place the pod on a node with insufficient free resources, etc.) but there are some circumstances where you may want more control on a node where a pod lands, for example to ensure that a pod ends up on a machine with an SSD attached to it, or to co-locate pods from two different services that communicate a lot into the same availability zone.

### nodeSelector

nodeSelector is the simplest recommended form of node selection constraint. nodeSelector is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well). The most common usage is one key-value pair.

* Run below command to get list of nodes

```
kubectl get nodes
```

* If you want label a node then use below command

```
kubectl label nodes <node-name> <label-key>=<label-value>
```

* For example assume node name is **kube-node-1** and my desired label is **'disktype=ssd'**, then I can run

```
kubectl label nodes kube-node-1 disktype=ssd
```

* Now pod templates look like

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
  nodeSelector:
    disktype: ssd

```

### Node affinity

Node affinity is conceptually similar to nodeSelector -- it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on the node.

There are currently two types of node affinity, called

*   requiredDuringSchedulingIgnoredDuringExecution ( Hard )

    specifies rules that must be met for a pod to be scheduled onto a node, similar to node selector
*   preferredDuringSchedulingIgnoredDuringExecution ( Soft )

    scheduler will try to enforce but will not guarantee.

Here's an example of a pod that uses node affinity:

```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

This node affinity rule says the pod can only be placed on a node with a label whose key is kubernetes.io/e2e-az-name and whose value is either e2e-az1 or e2e-az2. In addition, among nodes that meet that criteria, nodes with a label whose key is another-node-label-key and whose value is another-node-label-value should be preferred.

You can see the operator In being used in the example. The new node affinity syntax supports the following operators: In, NotIn, Exists, DoesNotExist, Gt, Lt. You can use NotIn and DoesNotExist to achieve node anti-affinity behavior, or use node taints to repel pods from specific nodes.

If you specify both nodeSelector and nodeAffinity, both must be satisfied for the pod to be scheduled onto a candidate node.

If you specify multiple nodeSelectorTerms associated with nodeAffinity types, then the pod can be scheduled onto a node if one of the nodeSelectorTerms can be satisfied.

### Inter-pod affinity and anti-affinity

Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled based on labels on pods that are already running on the node rather than based on labels on nodes.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
```

The affinity on this pod defines one pod affinity rule and one pod anti-affinity rule. In this example, the podAffinity is requiredDuringSchedulingIgnoredDuringExecution while the podAntiAffinity is preferredDuringSchedulingIgnoredDuringExecution. The pod affinity rule says that the pod can be scheduled onto a node only if that node is in the same zone as at least one already-running pod that has a label with key "security" and value "S1". (More precisely, the pod is eligible to run on node N if node N has a label with key topology.kubernetes.io/zone and some value V such that there is at least one node in the cluster with key topology.kubernetes.io/zone and value V that is running a pod that has a label with key "security" and value "S1".) The pod anti-affinity rule says that the pod should not be scheduled onto a node if that node is in the same zone as a pod with label having key "security" and value "S2".

Interpod Affinity and AntiAffinity can be even more useful when they are used with higher level collections such as ReplicaSets, StatefulSets, Deployments, etc. One can easily configure that a set of workloads should be co-located in the same defined topology, eg., the same node.

A Practical scenario

In a three node cluster, a web application has in-memory cache such as redis. We want the web-servers to be co-located with the cache as much as possible.

Here is the yaml snippet of a simple redis deployment with three replicas and selector label app=store. The deployment has PodAntiAffinity configured to ensure the scheduler does not co-locate replicas on a single node.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```

The below yaml snippet of the webserver deployment has podAntiAffinity and podAffinity configured. This informs the scheduler that all its replicas are to be co-located with pods that have selector label app=store. This will also ensure that each web-server replica does not co-locate on a single node.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.16-alpine
```

If we create the above two deployments, our three node cluster should look like below.

| node-1      | node-2      | node-3      |
| ----------- | ----------- | ----------- |
| webserver-1 | webserver-2 | webserver-3 |
| cache-1     | cache-2     | cache-3     |

As you can see, all the 3 replicas of the web-server are automatically co-located with the cache as expected.

```
kubectl get pods -o wide
```

The output is similar to this:

```
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
redis-cache-1450370735-6dzlj   1/1       Running   0          8m        10.192.4.2   kube-node-3
redis-cache-1450370735-j2j96   1/1       Running   0          8m        10.192.2.2   kube-node-1
redis-cache-1450370735-z73mh   1/1       Running   0          8m        10.192.3.1   kube-node-2
web-server-1287567482-5d4dz    1/1       Running   0          7m        10.192.2.3   kube-node-1
web-server-1287567482-6f7v5    1/1       Running   0          7m        10.192.4.3   kube-node-3
web-server-1287567482-s330j    1/1       Running   0          7m        10.192.3.2   kube-node-2
```
