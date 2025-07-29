# Resiliency in Software Architecture

## Introduction to Resiliency

As part of resiliency engineering, what we prepare is to make sure that the whole system doesn't stop functioning when one component stops working as expected. It is being prepared or it is accepting the fact that one or more components can break down and we prepare the whole system to not come down when one of the components goes down.

## Four Concepts of Resiliency

There are four concepts for resiliency:

1. **Rebound**
2. **Robustness**
3. **Graceful Extensibility**
4. **Sustained Adaptability**

### Rebound
Rebound is when something bad happens. How easily are you able to recover?

### Robustness
Robustness is the ability to absorb perturbations - that is how we handle a slight abnormality and move on with normal operations. An example is when a pod in a Kubernetes cluster dies, and when we have more than one replica, the end user will not notice the difference and the pod automatically restarts and everything goes smoothly.

The catch with robustness is that it really only works if we can model the expected perturbations. That is, if we know that the pod can crash and we prepare our system to automatically restart the pod, then we can build a robust system. But if you don't know or don't expect a pod to fail, then we cannot build a system which recovers.

A trade-off that we usually face is when we increase the complexity to improve robustness, this can lead to more issues.

### Graceful Extensibility
Graceful extensibility is the handling of surprise, or what happens when the perturbations happen outside the understood model. The question to ask yourself is: how well do you handle the unexpected?

### Sustained Adaptability
Sustained adaptability takes the concept of adaptability one step further. It calls for continuous learning and transformation, using crises as windows of opportunity to evolve and innovate.

## Chaos Engineering

Chaos engineering is the mechanism of injecting known perturbations so that we can identify if the system can handle those perturbations efficiently.

As we build more complex systems and we keep adding more things, the probability on a given day that something is going to fail increases.

## Distributed Architecture Complexities

While dealing with distributed architecture, there are three complexities that we need to deal with:

1. You can't beam information between two points instantly
2. Sometimes you can't reach the thing you want to talk to
3. Resources are not infinite

## Building Stability - Redundancy Pattern

There are multiple ways to build stability and one of the patterns is redundancy. In this pattern, we provision more resources than needed to ensure that a component failing doesn't interrupt normal operations.

Let's take an example where a load balancer is balancing the load between three resources and one of the resources goes down. In the normal situation when all three are up, the resources are handling 1/3 of the whole traffic. But when one of them is down, the two resources are handling half of the traffic, so they must be provisioned to handle half of the whole traffic. The best practice is to have another additional node, so that instead of 3, provision 4 nodes and in case of a failure, the load is actually distributed between three nodes.

## Timeouts

Setting a timeout is a crucial part of building resilient systems. Keeping the timeout too short will result in users not getting their requests fulfilled, and setting a timeout too long will result in resources being held up for a long time and resulting in resource exhaustion.

To set the correct timeout, do a performance test and find out the 99th percentile of your latency graph. So when you set a timeout for 99th percentile, you accept the trade-off that one percent of your users will not get their requests fulfilled, but 99% of your requests will be fulfilled successfully.

Also keep the timeouts as a configuration file whenever possible so that we don't have to do a code change whenever we are changing the timeout.

## Retries

Setting up a retry is also important. Without a retry, we lose an opportunity to serve the request. Setting the number of retries and the time between retries are crucial. If there is no difference between the timeouts between retries, all the retry attempts will be done in a hammering manner, which can put a lot of constraints on the resource. So the retries must have a random spacing or planned spacing between the attempts.

When configuring a retry, we should be careful with the requests that are not idempotent. For example, when you retry a transfer request, we must make sure the transfer is not initiated multiple times.

### Making Requests Idempotent

**Option 1: Request ID**
One way to make a request idempotent is by using request ID. How this works is that a client will send a request ID and in case of retry, the client sends the same request ID and the server checks for the request ID and processes the request only once per request ID. The server must check the request ID and send the same response that it would have sent for the first request.

**Option 2: Server-side Fingerprinting**
The second option to make a request idempotent is to use server-side fingerprinting. A fingerprint of the request on the server side will check for duplicate requests. Keep in mind that adding a timestamp in the request creates different fingerprints.

Some organizations like Amazon use both fingerprinting and request ID to make sure the API is idempotent.

## Handling Too Much Load

Options for handling too much load are:

1. Just fall over
2. Scale up your systems
3. Load shedding (throw work away)
4. Back pressure (reduce the work coming in)

Options 3 and 4 are part of rate limiting.

### Drawbacks of Fall Over
- High chance of in-flight customer work being interrupted
- Data loss
- Difficult to predict the impact

### Scaling Up
Scaling up should be done carefully to avoid scaling up during a DDoS attack.

There are various JVMs that have different time to load, and if you have a system that scales up regularly, find a JVM which loads fast and starts accepting requests quickly.

## Bulkhead Pattern

Adding a bulkhead for the services prevents the impact on one service from cascading to other services. Bulkhead is a way of isolating systems to avoid the side effects of one system onto another.

## Rate Limiting

Rate limiting is a way of reducing the rate that work is added to the system to keep the system stable. The trade-off here is you want to keep your systems stable and reject a few requests rather than accepting all the requests and making your system unstable.

### Load Shedding
Load shedding is one way of doing rate limiting. In load shedding we reject the load when the server is at maximum capacity and cannot take any more requests. To know the limit when you should start doing load shedding, we need to perform a benchmarking test to understand how much load the system can take.

### Back Pressure
Another way of handling rate limiting is back pressure. In back pressure there can be two ways of rate limiting: the first is on the client side and the second on the server side.

**Client-side Back Pressure:**
On the client side, we can implement circuit breaker which will limit the number of requests sent to server when it knows the server cannot handle the load.

We should be careful while implementing the circuit breaker because if one of the nodes is not responding and say 25% of the requests are not being served by one of the four nodes, we should not be limiting the load for all servers. One way to handle this is to send only the retries to circuit breaker and all other requests to the normal service.

Another way to handle rate limiting is to have a leaky bucket concept where tokens leave the bucket at a steady rate. If the bucket is full, no requests will be served.

**Server-side Back Pressure:**
It can also be handled at server side by sending a 429 HTTP response and in the response header the server can add a parameter that says "retry after X seconds."

To summarize: load shedding is when the server rejects requests when a threshold is reached, and back pressure is to tell the client to slow or stop sending requests.

## Error Budgets

Error budgets can have parameters like SLA, SLO, and SLI:

- **SLA** usually involves legal teams and has overall health indicators for the system
- **SLO** is more at a team level which indicates the service can have a maximum of X minutes downtime in a quarter. This helps to decide how much risk or changes that we can take in this quarter or year
- **SLI** is how to measure the system uptime using observability tools

## CAP Theorem

CAP theorem says that out of three, only two guarantees can be met. Consistency, availability, and partition tolerance are the three guarantees. In a distributed system, partition tolerance is always to be guaranteed, as there can be network drops at any nodes.

Partition tolerance says the system continues to operate despite an arbitrary number of messages being dropped by the network between nodes.

### Real-world Example
An example of a real-world system can be an e-commerce website that sells a product and has three microservices: one microservice for payments, second microservice for inventory, and third for catalogue management. Suppose inventory microservice is down - we have two options: either stop accepting the orders and payments, or we take the orders and when inventory system is up, update the inventory. This is an example of choosing consistency or availability.
