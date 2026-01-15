# Load Balancers Basic



A **load balancer** is a network device or software component that distributes inbound client requests across multiple backend servers to:

* **Optimize resource use**, ensuring no server is overloaded
* **Improve application performance** by routing requests to the least-busy server
* **Increase fault tolerance**, automatically redirecting traffic if a server fails
* **Enhance scalability**, allowing you to add or remove servers without downtime

### How It Works

{% stepper %}
{% step %}
### Client Request

A user’s browser sends a request (e.g., HTTP GET) to the application’s public IP or DNS name, which points to the load balancer.
{% endstep %}

{% step %}
### Algorithm Selection

The load balancer applies a chosen algorithm to decide which backend server should handle this request:

* **Round Robin:** Cycles through servers in order
* **Least Connections:** Chooses the server with the fewest active connections
* **IP Hash:** Maps each client IP to a specific server, providing session persistence
{% endstep %}

{% step %}
### Health Checks

Periodically, the load balancer pings or sends HTTP/S requests to each backend server.

* If a server fails to respond or returns an error, it is marked **unhealthy** and removed from rotation until it recovers.
{% endstep %}

{% step %}
### Request Proxying

The load balancer forwards the client’s request to the selected backend server and relays the server’s response back to the client. This proxy mode can be:

* **Layer 4 (TCP/UDP):** Operates at the transport layer, forwarding raw packets
* **Layer 7 (HTTP/S):** Operates at the application layer, inspecting HTTP headers or cookies
{% endstep %}
{% endstepper %}

### Key Concepts

* **Session Persistence (“Sticky Sessions”)**\
  Ensures a client is consistently routed to the same backend—for example, via cookies or source IP hashing.
* **SSL/TLS Termination**\
  Decrypts HTTPS traffic at the load balancer, offloading CPU work from backend servers.
* **Autoscaling Integration**\
  Dynamically adds or removes backend instances based on traffic patterns, often via cloud APIs.
* **Global vs. Local Load Balancing**
  * **Local:** Distributes traffic within a single data center or region.
  * **Global (GSLB):** Routes users to the nearest or best-performing regional deployment.

This foundational overview equips you to both understand and teach the practical session on setting up and demonstrating load balancing in real-world scenarios.

AWS offers four main types of load balancers under its Elastic Load Balancing (ELB) service, each optimized for different use cases and operating at different layers of the OSI model. Below is an overview of each type with real-time use cases:

### Application Load Balancer (ALB)

* **Layer:** 7 (Application layer)
* **Key Features:** Advanced routing (host-based and path-based), SSL termination, WebSocket and HTTP/2 support, integration with ECS/EKS, sticky sessions.
* **Use Case:** Ideal for microservices-based architectures and web applications that require content-based routing.\
  Example:\
  An e-commerce website uses an ALB to route `/product` traffic to a product service, `/cart` traffic to a cart service, and `/checkout` to a checkout microservice—all on different backend containers or EC2 instances. This enables modular scaling and secure, efficient traffic control.

### Network Load Balancer (NLB)

* **Layer:** 4 (Transport layer)
* **Key Features:** Extreme performance (millions of requests/sec), low latency, static IP, preserves source IP, TCP/UDP/TLS support.
* **Use Case:** Best for high-throughput and low-latency applications, and those using non-HTTP protocols.\
  Example:\
  A real-time IoT analytics platform collects data from thousands of devices using persistent TCP connections. An NLB distributes device connections across backend EC2 instances running the analytics engine, ensuring ultra-fast and reliable message handling during traffic spikes.

### Gateway Load Balancer (GWLB)

* **Layer:** 3/4 (Network and Transport layer, mainly used for packet-level routing)
* **Key Features:** Seamless insertion of security appliances (third-party firewalls, intrusion prevention systems), scalable inspection, integration with VPC traffic.
* **Use Case:** Deploying, scaling, and managing virtual appliances (firewalls, security tools).\
  Example:\
  In a regulated banking environment, all virtual network traffic must pass through a series of firewalls and intrusion detection systems. A GWLB automatically routes VPC traffic through these third-party security appliances for inspection and compliance before reaching application workloads.

### Classic Load Balancer (CLB) \[Legacy]

* **Layer:** 4 & 7 (Transport and Application)
* **Key Features:** Basic load balancing for HTTP, HTTPS, and TCP traffic. Supports simple health checks and sticky sessions.
* **Use Case:** Legacy or monolithic applications that don’t need advanced traffic routing.\
  Example:\
  A legacy web app running on old EC2-Classic instances in a “lift-and-shift” migration uses a CLB to distribute HTTP requests across the fleet of backend servers. Modern AWS guidance suggests using ALB/NLB for new workloads.

### AWS Load Balancer Types & Real-World Use Cases

| Type | Layer | Protocols      | Key Use Cases                               | Real-Time Example                                                             |
| ---- | ----- | -------------- | ------------------------------------------- | ----------------------------------------------------------------------------- |
| ALB  | 7     | HTTP/HTTPS     | Web apps, REST APIs, Microservices          | Routes different URL paths (e.g., `/api`, `/cart`) to different microservices |
| NLB  | 4     | TCP/UDP/TLS    | Real-time, gaming, IoT, legacy protocols    | Distributes persistent TCP connections for IoT device messaging               |
| GWLB | 3/4   | IP packets     | Security appliances, deep packet inspection | Routes all traffic through firewall appliances for compliance in a VPC        |
| CLB  | 4/7   | HTTP/HTTPS/TCP | Legacy, simple web or app workloads         | Distributes web traffic in legacy VM-based infrastructure                     |

AWS recommends:

* **ALB** for advanced web apps & microservices,
* **NLB** for ultra-fast, high-volume, low-latency workloads,
* **GWLB** for bringing in third-party security or networking appliances,
* **CLB** only for legacy workloads or simple setups.

This approach ensures the right balance of performance, security, and features for different cloud architectures.

## AWS Target Groups

AWS **target groups** are logical groupings of endpoints—such as EC2 instances, IP addresses, Lambda functions, or another load balancer—to which an Elastic Load Bal (ELB) routes incoming traffic. Target groups enable you to define how requests are routed, monitor health, and support advanced deployment patterns.

### Target Group Types and Targets

Each target group is created with a **target type** that determines the kinds of endpoints it can contain. Once set, the target type cannot be changed for that group:

| Target Type | Description                                                                                       | Use Case                                                                                       |
| ----------- | ------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| instance    | Targets are specified by EC2 instance ID.                                                         | Traditional EC2 workloads behind ALB or NLB.                                                   |
| ip          | Targets are specified by IPv4 or IPv6 addresses from allowed CIDR blocks (VPC subnets, RFC 1918). | On-premises servers via Direct Connect/VPN, ECS tasks on custom ports, or multi-port services. |
| lambda      | Targets are AWS Lambda functions (ALB only).                                                      | Serverless backends handling HTTP(s) requests.                                                 |
| alb         | Targets are another Application Load Balancer (NLB only).                                         | Combine NLB’s static IP or PrivateLink with ALB’s Layer 7 routing.                             |

(Protocols per target type vary by load balancer type; see section 3.)

### Health Checks

Health checks are defined _per target group_, allowing the load balancer to monitor and route only to **healthy** targets:

* **Protocol & Path**: HTTP, HTTPS, TCP, TLS, UDP, or gRPC (depending on load balancer and target group).
* **Interval & Thresholds**: Customize how frequently checks run and how many successes/failures constitute healthy/unhealthy.
* **Advanced Failover**: Configure DNS failover and routing–failover thresholds to improve resiliency when a zone’s healthy target count falls below a minimum.

### Protocols and Ports

Target groups support these listener-to-target protocols and ports:

Application Load Balancer (ALB) target groups:

* Protocols: HTTP, HTTPS
* Ports: 1–65535
* Optional HTTP/2 or gRPC between ALB and targets

Network Load Balancer (NLB) target groups:

* Protocols: TCP, TLS, UDP, TCP\_UDP
* Ports: 1–65535
* TLS target groups bypass certificate validation

### Routing Algorithms and Stickiness

For ALB target groups with `instance` or `ip` types, you can choose the routing algorithm:

| Algorithm                    | Behavior                                                                       |
| ---------------------------- | ------------------------------------------------------------------------------ |
| round\_robin                 | Evenly distributes requests sequentially.                                      |
| least\_outstanding\_requests | Sends new requests to targets with fewest in-flight requests.                  |
| weighted\_random             | Randomly distributes requests according to weights (no slow start/stickiness). |

Enable **sticky sessions** to bind a client to the same target across requests via application or load balancer cookies.

### Use Cases

{% stepper %}
{% step %}
### Blue/Green Deployment

Create two target groups representing different application versions. Adjust weights (ALB weighted target groups) to shift traffic gradually, enabling safe rollouts and easy rollback.
{% endstep %}

{% step %}
### Hybrid Architectures

Use `ip` target groups to register on-premises resources (via Direct Connect or VPN) alongside EC2 targets for unified load balancing.
{% endstep %}

{% step %}
### Serverless Routing

Register Lambda functions in an ALB target group to integrate HTTP(s) endpoints with serverless logic without API Gateway.
{% endstep %}

{% step %}
### Advanced Security and Networking

Configure NLB with an ALB-type target group to combine static IPs, PrivateLink, and Layer 7 routing (e.g., multimedia services requiring TCP and HTTP signaling).
{% endstep %}
{% endstepper %}

By organizing endpoints into target groups, AWS ELB provides granular control over traffic routing, health monitoring, and scaling for diverse architectures.
