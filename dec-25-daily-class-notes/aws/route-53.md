# route 53

## Route 53

Since the advent of the internet, IP addresses and domain names have been a core component of its ecosystem. You literally cannot use the internet without making use of IP addresses and domain names. Today, we intend to explore Amazon Route 53 which is a Domain Name System (DNS) service provided by AWS. If you are unfamiliar with the term Domain Name System, this section first explains DNS to put Route 53 into perspective.

#### What is a Domain Name System?

A Domain Name System (DNS) is a decentralized system used to translate human-friendly domain names, like http://www.example.com/, into machine-readable numeric IP addresses, such as 192.0.2.1. Think of DNS as a phonebook for the internet: domain names are the human-friendly names and IP addresses are the numeric locations computers use to connect to resources.

A DNS service works behind the scenes, ensuring that when you enter a domain name in your web browser, your request is routed to the correct IP address associated with the website. DNS servers translate queries for domain names into IP addresses, controlling which server an end user reaches when they type a domain name into a browser. DNS provides a hierarchical and distributed naming system that connects users to online resources.

#### What is Amazon Route 53?

Route 53 is a scalable and highly available DNS, domain name registration, and health-checking web service offered by AWS. It helps developers and businesses reliably route end users to Internet applications by translating domain names like http://example.com/ into numeric IP addresses that computers use to connect to each other.

With Route 53 you can:

* Register and manage domain names.
* Create DNS records to map domain names to IP addresses.
* Distribute traffic across multiple resources for high availability and scalability.
* Perform health checks to monitor resource availability.
* Use advanced routing options to optimize performance and resilience.

Route 53 connects requests to infrastructure running in AWS—such as Amazon EC2, Elastic Load Balancing, or S3—or to resources outside AWS.

We’ll explore Hosted Zones, Route 53 record types and name servers, record TTL (time to live), and routing policies.

#### What are Hosted Zones?

A hosted zone is a container for records that holds information about how you want to route traffic for a specific domain and its subdomains. It stores DNS records (A, AAAA, CNAME, etc.) for your domain.

Route 53 offers two types of hosted zones:

* Public Hosted Zones: contain records that specify how to route traffic on the internet. They resolve domain names to publicly routable IP addresses and are used for websites and services accessible worldwide.
* Private Hosted Zones: contain records for routing traffic within one or more VPCs or private networks. They are for internal use, map internal domain names to private IPs, and are associated with specific VPCs.

{% hint style="info" %}
Using hosted zones in Route 53 is not free. You are charged $0.50 per month for each hosted zone you create.
{% endhint %}

#### Record Types

Common Route 53 record types and their purposes:

* A (Address) Record: maps a domain or subdomain to an IPv4 address.
* AAAA (IPv6 Address) Record: maps a domain or subdomain to an IPv6 address.
* CNAME (Canonical Name) Record: maps an alias or nickname to a canonical domain name (works for subdomains, not the apex/root domain).

Difference between CNAME and Alias records (conceptual):

* CNAME: acts like a redirect/signpost from one domain to another; cannot be used for the apex/root domain.
* Alias: works like a direct connection/shortcut to AWS resources, and can be used for both subdomains and the apex/root domain. Alias records are specific to AWS resources and offer seamless integration.

#### Name Servers (NS)

Name servers are the servers responsible for handling DNS resolution for your domain. When you create a hosted zone in Route 53, you are assigned a set of four or more Route 53 name servers. To use Route 53 for DNS resolution, update your domain registrar’s name server records to the Route 53 name servers. After propagation, queries for your domain will be directed to Route 53’s authoritative name servers.

#### Record TTL (Time To Live)

TTL is the length of time a DNS record is cached by DNS resolvers or clients before requesting a fresh copy from the authoritative DNS server. TTL is specified in seconds and affects how quickly DNS changes propagate and how much load is placed on DNS infrastructure.

* Short TTL: faster propagation, increased DNS query load.
* Long TTL: reduced DNS query load, slower propagation.

Tip: For frequently changing records (e.g., failover), use shorter TTLs. For stable records, use longer TTLs.

{% hint style="info" %}
Route 53 does not have a default TTL for any record type. You must specify a TTL for each record.
{% endhint %}

#### Routing Policies

When you create a record, you choose a routing policy that determines how Route 53 responds to queries. Common routing policies:

* Simple Routing: associate a single resource record set with a domain/subdomain. No load balancing or traffic distribution; responds with the same record set for all queries.
* Failover Routing: configure primary and secondary resources and automatically route to the secondary when the primary is unhealthy.
* Geolocation Routing: route traffic based on the geographic location of the DNS resolver making the request.
* Geoproximity Routing: route traffic based on the geographic location of users and resources, allowing radius-based or prioritized routing.
* Latency-based Routing: route traffic to the AWS region that provides the lowest latency to the end user.
* Weighted Routing: associate multiple resources with a domain and distribute traffic according to assigned weights.
* IP-based Routing: upload user-IP-to-endpoint mappings to Route 53 for granular control using knowledge of your network and clients.
* Multi-value Answer Routing: return multiple healthy resources (IP addresses) in response to queries, allowing clients to choose.

Below are details and examples for some of these policies.

**Failover Routing policy**

This policy handles failover scenarios between a primary and a secondary resource.

{% stepper %}
{% step %}
### Define primary and secondary resources

Create two resource record sets: one for the primary resource and one for the secondary resource. The primary is the main endpoint; the secondary is the backup.
{% endstep %}

{% step %}
### Health checks

Configure Route 53 health checks for the primary resource. Route 53 periodically checks the health of the primary.
{% endstep %}

{% step %}
### Automatic failover

If the primary's health check fails, Route 53 automatically starts routing traffic to the secondary resource.
{% endstep %}

{% step %}
### Recovery

Route 53 continues monitoring and automatically switches traffic back to the primary once it becomes healthy again.
{% endstep %}
{% endstepper %}

**Geolocation Routing policy**

* Create resource record sets for geographic regions or countries.
* Map each record set to the regions or countries it should serve.
* Route 53 determines the resolver’s location from the resolver’s IP and selects the appropriate record set.
* Useful for directing users to language- or region-specific resources.

**Geoproximity Routing policy**

* Define locations for your resources and associate them with resources (AWS or IP addresses).
* Route 53 estimates distance between the end user and resources and routes to the nearest resource within configured parameters.

**Latency-based Routing policy**

* Create latency records for resources in multiple AWS regions.
* Route 53 uses latency measurements to send users to the region with the lowest latency (not simply the closest by distance).

**Weighted Routing policy**

* Associate multiple resource record sets with a domain and assign a weight to each.
* Route 53 routes traffic proportionally to the weights, useful for canary deployments or gradual traffic shifts.

**IP-based Routing policy**

* Upload user-IP-to-endpoint mappings to Route 53.
* Enables granular routing decisions based on specific knowledge of customer ISPs, networks, or client IP characteristics.

**Multi-value Answer Routing policy**

* Configure multiple healthy resources and return multiple values (e.g., IPs) in DNS responses.
* Notes:
  * If a multi-value record has an associated health check, Route 53 only returns that IP when the check is healthy.
  * Without a health check, Route 53 considers the record healthy.
  * If there are eight or fewer healthy records, Route 53 returns all healthy records.
  * If all records are unhealthy, Route 53 returns up to eight unhealthy records.

#### Final Words

This article explored Route 53’s DNS features and capabilities: hosted zones, record types, name servers, TTL, and routing policies. Route 53 helps simplify DNS management while providing tools for resilient, highly available applications. Use the appropriate routing policies, health checks, and record configurations to optimize performance and availability.

Happy routing!

***

#### Scenario-Based Questions

At the end of your session, challenge participants with the scenarios below to validate understanding of record types, routing policies, and health checks.

|  # | Scenario                                                                                                                       |
| -: | ------------------------------------------------------------------------------------------------------------------------------ |
|  1 | Your web app must remain available if an EC2 instance in **us-east-1** crashes. How configure DNS failover?                    |
|  2 | Users in Asia experience high latency. How use Route 53 to send them to the nearest AWS region?                                |
|  3 | You must gradually shift 20% of traffic to a new version of your app for canary testing. How set this up?                      |
|  4 | You host static assets in an S3 bucket behind CloudFront and dynamic API servers in EC2. How create DNS records?               |
|  5 | Your company wants to serve a different landing page to European users. Which routing policy fits best?                        |
|  6 | You need to block DNS queries from a specific country—how can Route 53 help?                                                   |
|  7 | An on-premises database must be used as a primary endpoint, with AWS as backup. How configure geoproximity or hybrid failover? |

Use these scenarios to validate participants’ understanding of record types, routing policies, and health checks.
