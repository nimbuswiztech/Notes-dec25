# Pod Disruption Budget

## Voluntary and Involuntary Disruptions

Pods do not disappear until someone (a person or a controller) destroys them, or there is an unavoidable hardware or system software error.

We call these unavoidable cases involuntary disruptions to an application. Examples are:

* a hardware failure of the physical machine backing the node
* cluster administrator deletes VM (instance) by mistake
* cloud provider or hypervisor failure makes VM disappear
* a kernel panic
* the node disappears from the cluster due to cluster network partition
* eviction of a pod due to the node being out-of-resources

Except for the out-of-resources condition, all these conditions should be familiar to most users; they are not specific to Kubernetes.

We call other cases voluntary disruptions. These include both actions initiated by the application owner and those initiated by a Cluster Administrator. Typical application owner actions include:

* deleting the deployment or other controller that manages the pod
* updating a deployment’s pod template causing a restart
* directly deleting a pod (e.g. by accident)

Cluster Administrator actions include:

* Draining a node for repair or upgrade.
* Draining a node from a cluster to scale the cluster down (learn about Cluster Autoscaling ).
* Removing a pod from a node to permit something else to fit on that node.

## Dealing with Disruptions

Here are some ways to mitigate involuntary disruptions:

* Ensure your pod requests the resources it needs.
* Replicate your application if you need higher availability.
* For even higher availability when running replicated applications, spread applications across racks (using anti-affinity) or across zones (if using a multi-zone cluster.)

## PodDisruptionBudget

An Application Owner can create a PodDisruptionBudget object (PDB) for each application. A PDB limits the number pods of a replicated application that are down simultaneously from voluntary disruptions. For example, a quorum-based application would like to ensure that the number of replicas running is never brought below the number needed for a quorum. A web front end might want to ensure that the number of replicas serving load never falls below a certain percentage of the total.
