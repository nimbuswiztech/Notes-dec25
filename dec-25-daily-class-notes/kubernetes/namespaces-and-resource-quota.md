# Namespaces and resource quota

## Namespaces

Namespaces are intended for use in environments with many users spread across multiple teams or projects. For clusters with a few to tens of users, you should not need to create or think about namespaces at all. Start using namespaces when you need the features they provide.

Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces. Namespaces cannot be nested inside one another and each Kubernetes resource can only be in one namespace.

Namespaces are a way to divide cluster resources between multiple users (via resource quota).

By default, depending on installation method, Kubernetes creates 4 namespaces:

* `default` — The default namespace for objects with no other namespace
* `kube-system` — The namespace for objects created by the Kubernetes system
* `kube-public` — This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
* `kube-node-lease` — This namespace is for the lease objects associated with each node which improves the performance of the node heartbeats as the cluster scales

You can create a namespace using the command line or a YAML file.

* Using the command line:

```
kubectl create ns namespace_name
```

* Using a YAML file:

```
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>
```

Setting the namespace preference

You can permanently save the namespace for all subsequent kubectl commands in the current context:

```
kubectl config set-context --current --namespace=<insert-namespace-name-here>
```

Validate it:

```
kubectl config view --minify | grep namespace:
```

To delete a namespace:

```
kubectl delete ns namespace_name
```

## Resource quota

When several users or teams share a cluster with a fixed number of nodes, there is a concern that one team could use more than its fair share of resources.

Resource quotas are a tool for administrators to address this concern.

A resource quota, defined by a `ResourceQuota` object, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace.

**Enabling Resource Quota**

Resource Quota support is enabled by default for many Kubernetes distributions. It is enabled when the API server `--enable-admission-plugins=` flag has `ResourceQuota` as one of its arguments.

A resource quota is enforced in a particular namespace when there is a `ResourceQuota` in that namespace.
