# Kubernetes for beginners

![basic\_objects](<../../.gitbook/assets/107465586 a1d8db00 6b88 11eb 9e05 7829b98cd60a.JPG>)

## kubernetes

### Pod

#### pod is a object which encapsulates or groups one or more containers (such as Docker containers), with shared storage/network, and a specification for how to run the containers.

below is the basic structure of pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-app
  labels:
    type: front-end
spec:
  containers:
    - name: nginx
      image: nginx
```

Commands:

* list the pods

```bash
kubectl get pods
```

* create a pod with image busybox (it actually creates deployment)

```bash
kubectl run my-pod --image=busybox
```

* delete a particular pod

```bash
kubectl delete pod pod_name
```

* if you have created pod using `kubectl run` command but you want to see configurations of pod, to get yaml out of pod use:

```bash
kubectl get pod pod_name -o yaml > my_pod_file.yaml
```

* describe your pod

```bash
kubectl describe pod pod_name
```

* login to a running pod

```bash
kubectl exec -it pod_name /bin/sh
```

* access logs of any container inside pod

```bash
kubectl logs pod_name -c conatiner_name_inside_pod
```

(container name inside pod you will get it by `kubectl describe pod` command)

* edit a pod created with `kubectl run` (opens vi editor; change and save — Kubernetes will automatically update the pod)

```bash
kubectl edit pod pod_name
```

* list pods in another namespace (assuming namespace is `dev`)

```bash
kubectl get pods --namespace=dev
```

***

### ReplicaSet

#### replicaset is an kubernetes object which is used make your pod highly available and any point time it will make sure sepcified replicas are up and running

basic structure replicaset

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-rs
  labels:
    type: front-end
spec:
  template:
    metadata:
      name: my-pod
      labels:
        type: front-end
    spec:
      containers:
        - name: nginx
          image: nginx
  replicas: 6
  selector:
    matchLabels:
      type: front-end
```

when you create replica set, it automatically creates pods

Commands:

* list replicasets

```bash
kubectl get rs
# or
kubectl get replicasets
```

* list replicasets in other namespace (assuming namespace is `dev`)

```bash
kubectl get rs --namespace=dev
```

* delete replicaset

```bash
kubectl delete rs rs_name
```

* get replicaset yaml (if created via `kubectl run` and you want the configuration file)

```bash
kubectl get rs rs_name -o yaml > rs_sample.yaml
```

* edit rs (opens vi editor; changes are applied when saved)

```bash
kubectl edit rs rs_name
```

Notes on updating ReplicaSet:

* If you created an RS with image `busybox777` and replicas `3` and pods failed, then change the RS and apply it back — existing pods may retain the old configuration. Options:
  1. delete all old pods so new pods will be created with the updated configuration
  2. delete the RS and re-create it
* increase or decrease replica count without changing RS yaml config file:

```bash
kubectl scale rs rs_name --replicas=6
```

***

### Deployment

#### A Deployment provides declarative updates for Pods and ReplicaSets.

#### You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

basic structure Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

when you create deployment, it automatically creates ReplicaSet and pods

Commands:

* list deployments

```bash
kubectl get deploy
# or
kubectl get deployment
```

* list deployments in other namespace (assuming namespace is `dev`)

```bash
kubectl get deploy --namespace=dev
```

* delete deployment

```bash
kubectl delete deploy deployment_name
```

* get deployment yaml (if created via `kubectl run` and you want the configuration file)

```bash
kubectl get deploy deployment -o yaml > deploy_sample.yaml
```

* edit deployment (opens vi editor; changes are applied when saved)

```bash
kubectl edit deploy deployment_name
```

* check status of new deployment

```bash
kubectl rollout status deployment_name
```

* see history of deployment

```bash
kubectl rollout history deployment deployment_name
```

* play with undo option

```bash
kubectl --record deployment.apps/deployment_name set image deployment.v1.apps/deployemnt_name conatiner_name=deekshithsn/ui
kubectl --record deployment.apps/deployment_name set image deployment.v1.apps/deployemnt_name conatiner_name=nginx
kubectl rollout undo deployment deployment_name
kubectl rollout undo deployment deployment_name --to-revision=1
```
