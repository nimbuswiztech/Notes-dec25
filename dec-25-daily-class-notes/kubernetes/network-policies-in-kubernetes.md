# Network policies in kubernetes

* A network policy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints.
* If you want to control traffic flow at the IP address or port level (OSI layer 3 or 4), then you might consider using Kubernetes NetworkPolicies for particular applications in your cluster.

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:

* Other pods that are allowed (exception: a pod cannot block access to itself)
* Namespaces that are allowed
* IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)

Prerequisite to work with this object:

1. kubernetes version more than 1.8
2. networking solution should be calico, weave-net, cilium, kube-router and romana

By default, pods are non-isolated; they accept traffic from any source.

Pods become isolated by having a NetworkPolicy that selects them. Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. (Other pods in the namespace that are not selected by any NetworkPolicy will continue to accept all traffic.)

Network policies do not conflict; they are additive. If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies' ingress/egress rules. Thus, order of evaluation does not affect the policy result.

For a network flow between two pods to be allowed, both the egress policy on the source pod and the ingress policy on the destination pod need to allow the traffic. If either the egress policy on the source, or the ingress policy on the destination denies the traffic, the traffic will be denied.

{% hint style="info" %}
Mandatory Fields: As with all other Kubernetes config, a NetworkPolicy needs apiVersion, kind, and metadata fields. For general information about working with config files, see Configure Containers Using a ConfigMap, and Object Management.
{% endhint %}

## spec

NetworkPolicy spec has all the information needed to define a particular network policy in the given namespace.

## podSelector

Each NetworkPolicy includes a podSelector which selects the grouping of pods to which the policy applies. The example policy selects pods with the label "role=db". An empty podSelector selects all pods in the namespace.

## policyTypes

Each NetworkPolicy includes a policyTypes list which may include either Ingress, Egress, or both. The policyTypes field indicates whether or not the given policy applies to ingress traffic to selected pod, egress traffic from selected pods, or both. If no policyTypes are specified on a NetworkPolicy then by default Ingress will always be set and Egress will be set if the NetworkPolicy has any egress rules.

## ingress

Each NetworkPolicy may include a list of allowed ingress rules. Each rule allows traffic which matches both the from and ports sections. The example policy contains a single rule, which matches traffic on a single port, from one of three sources, the first specified via an ipBlock, the second via a namespaceSelector and the third via a podSelector.

## egress

Each NetworkPolicy may include a list of allowed egress rules. Each rule allows traffic which matches both the to and ports sections. The example policy contains a single rule, which matches traffic on a single port to any destination in 10.0.0.0/24.

## Behavior of to and from selectors

There are four kinds of selectors that can be specified in an ingress from section or egress to section:

<details>

<summary>podSelector</summary>

This selects particular Pods in the same namespace as the NetworkPolicy which should be allowed as ingress sources or egress destinations.

</details>

<details>

<summary>namespaceSelector</summary>

This selects particular namespaces for which all Pods should be allowed as ingress sources or egress destinations.

</details>

<details>

<summary>namespaceSelector and podSelector</summary>

A single to/from entry that specifies both namespaceSelector and podSelector selects particular Pods within particular namespaces.

</details>

<details>

<summary>ipBlock</summary>

This selects particular IP CIDR ranges to allow as ingress sources or egress destinations. These should be cluster-external IPs, since Pod IPs are ephemeral and unpredictable.

</details>
