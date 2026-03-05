# Ingress in Kubernetes

An Ingress in Kubernetes is a collection of rules that allow external HTTP and HTTPS traffic to reach services within a Kubernetes cluster. It acts as an entry point to the cluster and controls the routing of external requests to different services based on URL paths or hostnames.

{% hint style="info" %}
**Ingress Controller:** Ingress doesn't do anything by itself; it requires an Ingress Controller to actually implement the routing rules. Ingress controllers are responsible for managing the ingress resources and implementing the rules (for example, NGINX Ingress Controller, HAProxy, etc.).
{% endhint %}

**Ingress Resource:** It defines the rules and configuration for routing external traffic to the appropriate service within the Kubernetes cluster.

### Benefits of Using Ingress

* **Consolidated Routing:** Allows you to define routing rules for all your services in one place, rather than having to expose each service individually.
* **SSL Termination:** Ingress can handle SSL termination, meaning you can configure HTTPS endpoints and offload the SSL decryption to the Ingress Controller.
* **Path-Based and Host-Based Routing:** Allows flexible routing based on URL paths or hostnames, enabling you to serve multiple applications under the same domain.
* **Load Balancing:** Ingress can perform load balancing for the backend services, distributing the traffic evenly.
