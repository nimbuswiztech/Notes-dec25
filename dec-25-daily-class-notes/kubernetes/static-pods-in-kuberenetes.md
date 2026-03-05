# Static Pods  in kuberenetes

It is possible to create Pods by writing a file to a directory watched by kubelet. These are called static Pods. Unlike DaemonSet, static Pods cannot be managed with kubectl or other Kubernetes API clients. Static Pods do not depend on the apiserver, making them useful in cluster bootstrapping cases. Also, static Pods may be deprecated in the future.

Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane (for example, a Deployment), the kubelet watches each static Pod and restarts it if it fails.

Key points:

* Static Pods are always bound to one kubelet on a specific node.
* The kubelet automatically tries to create a mirror Pod on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled from there.
* Static Pod YAML files will be present in /etc/kubernetes/manifest on the node.

<details>

<summary>More details</summary>

For more details - https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

</details>
