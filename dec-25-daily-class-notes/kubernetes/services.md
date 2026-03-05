# Services

### What is a Service in Kubernetes?

* A Service enables network access to a set of Pods in Kubernetes.
* Services select Pods based on their labels. When a network request is made to the service, it selects all Pods in the cluster matching the service's selector, chooses one of them, and forwards the network request to it.

Here are general attributes of a Kubernetes service:

* A label selector can find pods that are targeted by a service.
* A service is assigned an IP address (“cluster IP”), which the service proxies use.
* A service can map an incoming port to any targetPort. (The targetPort is set, by default, to the port field’s same value. The targetPort can be defined as a string.)
*   The port number assigned to each name can vary in each backend pod.

    For example, you can change the port number that pods expose in the next version of your backend software, without breaking clients.
* Services support TCP (default), UDP and SCTP for protocols.
* Services can be defined with or without a selector.
* Services support a variety of port definitions.

![service](<../../.gitbook/assets/service route.gif>)

### types of services

<details>

<summary><code>ClusterIP</code> – The default value</summary>

The service is only accessible from within the Kubernetes cluster – you can’t make requests to your Pods from outside the cluster!

</details>

<details>

<summary><code>NodePort</code></summary>

This makes the service accessible on a static port on each Node in the cluster. This means that the service can handle requests that originate from outside the cluster.

</details>

<details>

<summary><code>LoadBalancer</code></summary>

The service becomes accessible externally through a cloud provider's load balancer functionality. GCP, AWS, Azure, and OpenStack offer this functionality. The cloud provider will create a load balancer, which then automatically routes requests to your Kubernetes Service.

</details>

<details>

<summary><code>ExternalName</code></summary>

This type maps the service to the contents of the externalName field (e.g., foo.bar.example.com). It does this by returning a value for the CNAME record.

</details>
