# Types of Deployment strategies

In Kubernetes there are a few different ways to release an application, it is necessary to choose the right strategy to make your infrastructure reliable during an application update.

Choosing the right deployment procedure depends on the needs, we listed below some of the possible strategies to adopt:

* **recreate**: terminate the old version and release the new one
* **rolling update**: release a new version on a rolling update fashion, one after the other
* **blue/green**: release a new version alongside the old version then switch traffic
* **canary**: release a new version to a subset of users, then proceed to a full rollout

### recreate

A deployment defined with a strategy of type Recreate will terminate all the running instances then recreate them with the newer version.

![recreate](<../../.gitbook/assets/107481353 09505400 6ba4 11eb 9046 42863db5a338.JPG>)

**Pro:**

* application state entirely renewed
* not suited for higher environments like prod

**Cons:**

* downtime that depends on both shutdown and boot duration of the application

### Rolling update ( default )

A secondary ReplicaSet is created with the new version of the application, then the number of replicas of the old version is decreased and the new version is increased until the correct number of replicas is reached.

![rolling update](<../../.gitbook/assets/107481354 09e8ea80 6ba4 11eb 94cd 6f1404b695ae.JPG>) ![rollingupdate-1](<../../.gitbook/assets/107481356 0a818100 6ba4 11eb 8d4a 0491df3ece15.JPG>)

When setup together with horizontal pod autoscaling it can be handy to use a percentage based value instead of a number for maxSurge and maxUnavailable.

If you trigger a deployment while an existing rollout is in progress, the deployment will pause the rollout and proceed to a new release by overriding the rollout.

**Pro:**

* version is slowly released across instances
* convenient for stateful applications that can handle rebalancing of the data

**Cons:**

* rollout/rollback can take time
* supporting multiple APIs is hard
* no control over traffic

### Blue/green

A blue/green deployment differs from a ramped deployment because the "green" version of the application is deployed alongside the "blue" version. After testing that the new version meets the requirements, we update the Kubernetes Service object that plays the role of load balancer to send traffic to the new version by replacing the version label in the selector field.

![bluegreen](<../../.gitbook/assets/107481436 2c7b0380 6ba4 11eb 9f5a fc93a6253bc0.JPG>)

**Pro:**

* instant rollout/rollback
* avoid versioning issue, change the entire cluster state in one go

**Cons:**

* requires double the resources
* proper test of the entire platform should be done before releasing to production
* handling stateful applications can be hard

### Canary

A canary deployment consists of routing a subset of users to a new functionality. In Kubernetes, a canary deployment can be done using two Deployments with common pod labels. One replica of the new version is released alongside the old version. Then after some time and if no error is detected, scale up the number of replicas of the new version and delete the old deployment.

![canray](<../../.gitbook/assets/107481347 08b7bd80 6ba4 11eb 85d5 bcfee2a57a9d.JPG>)

![canary](<../../.gitbook/assets/107481438 2e44c700 6ba4 11eb 85d7 8274c1705ad0.JPG>)

**Pro:**

* version released for a subset of users
* convenient for error rate and performance monitoring
* fast rollback

**Cons:**

* slow rollout
* fine tuned traffic distribution can be expensive (99% A/ 1%B = 99 pod A, 1 pod B)
