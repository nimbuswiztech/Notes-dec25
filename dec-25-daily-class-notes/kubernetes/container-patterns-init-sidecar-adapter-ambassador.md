# Container patterns ( init, sidecar, Adapter, Ambassador)

## Init containers

A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

Init containers are exactly like regular containers, except:

* Init containers always run to completion.
* Each init container must complete successfully before the next one starts.

Differences from regular containers:

Init containers support all the fields and features of app containers, including resource limits, volumes, and security settings. However, the resource requests and limits for an init container are handled differently, as documented below.

**Resources**

```
Given the ordering and execution for init containers, the following rules for resource usage apply:

The highest of any particular resource request or limit defined on all init containers is the effective init request/limit
The Pod's effective request/limit for a resource is the higher of:
the sum of all app containers request/limit for a resource
the effective init request/limit for a resource
Scheduling is done based on effective requests/limits, which means init containers can reserve resources for initialization that are not used during the life of the Pod.
The QoS (quality of service) tier of the Pod's effective QoS tier is the QoS tier for init containers and app containers alike.
Quota and limits are applied based on the effective Pod request and limit.
```

Also, init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe because they must run to completion before the Pod can be ready.

If you specify multiple init containers for a Pod, kubelet runs each init container sequentially. Each init container must succeed before the next can run. When all of the init containers have run to completion, kubelet initializes the application containers for the Pod and runs them as usual.

Using init containers

```
Because init containers have separate images from app containers, they have some advantages for start-up related code:

- Init containers can contain utilities or custom code for setup that are not present in an app image. For example, there is no need to make an image 
FROM another image just to use a tool like sed, awk, python, or dig during setup.
- The application image builder and deployer roles can work independently without the need to jointly build a single app image.
- Init containers can run with a different view of the filesystem than app containers in the same Pod. Consequently, they can be given access to Secrets 
that app containers cannot access.
- Because init containers run to completion before any app containers start, init containers offer a mechanism to block or delay app container startup 
until a set of preconditions are met. Once preconditions are met, all of the app containers in a Pod can start in parallel.
- Init containers can securely run utilities or custom code that would otherwise make an app container image less secure. By keeping unnecessary tools 
separate you can limit the attack surface of your app container image
```

## Side car

* A sidecar is just a container that runs on the same Pod as the application container. Because it shares the same volume and network as the main container, it can “help” or enhance how the application operates.
* Common examples of sidecar containers are log shippers, log watchers, monitoring agents, among others. Having a separate container for auxiliary tasks gives you access to health checks, automatic restart and other functionality offered by Kubernetes for containers.
* When following the Sidecar Pattern, you should consider making a small sidecar container that does not consume many resources. If you find that the sidecar container logic is getting more complex and/or becoming more tightly coupled with the main application container, it may be better integrated with the main application’s code instead. Remember, the strong point of a sidecar container lies in its ability to be small and pluggable.

![sidecar](<../../.gitbook/assets/105957719 a1781480 609f 11eb 9355 d0ef077c084b.JPG>)

## Adapter

Adapter pattern is yet another form of single-node, multiple containers application patterns. It advocates usage of additional containers to present a simplified, homogenized view of an application running within a container. These containers can be called adapter containers. The main container communicates with the adapter container through localhost or a shared local volume. This is unlike ambassador containers which simply simplify access to external servers.

A concrete example of the adapter pattern is adapters that ensure all containers in a system have the same monitoring interface. The following diagram represents usage of adapter container for monitoring:

![adaptor](<../../.gitbook/assets/105957712 a046e780 609f 11eb 9579 ab0013499628.JPG>)

## Ambassador/Proxy Pattern (Ambassador Container)

Ambassador pattern is another form of single-node, multiple containers application patterns. This pattern advocates usage of additional containers as proxies to an external group/cluster of servers. The primary goal is to simplify the access of external servers using the ambassador container. This container can be grouped as a logical atomic unit such that the application container can invoke this ambassador/proxy container which can then invoke the cluster of servers. The following is a primary advantage of using an ambassador container:

* Let the ambassador container take care of connecting to a cluster of servers; the application container connects to the ambassador container using the “localhost” paradigm.
* This enables using a standalone server (and not a cluster of servers) in the development environment. The application container opens the connection to the server on “localhost” and finds the proxy without any service discovery.

Examples:

* Ambassador container acting as a proxy to Memcached servers:

Ambassador (memcached) container

![ambasider](<../../.gitbook/assets/105957720 a2a94180 609f 11eb 9fa2 a627aebb40cf.JPG>)

* Ambassador container acting as a proxy to a Redis cluster of servers; the ambassador container becomes responsible for splitting reads and writes and sending them on to the appropriate servers.
