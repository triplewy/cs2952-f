# CS2952-f Papers

### [The Hidden Dividends of Microservices](https://queue.acm.org/detail.cfm?id=2956643)

*Short Summary*

This paper discusess multiple advantages of microservices over monoliths which include:

1. **Permissionless Innovation**: High rate of experimentation and a low rate of cross-team meetings
2. **Enable Failure**: (Not sure about this one) Having microservices leads to discussing failure scenarios upfront  
3. **Disrupt Trust**: Similar to #1
4. **You build it, you own it**: Being on-call for your own service
5. **Accelerate deprecations**: Can upgrade more easily if microservices are only bound to their communication interface
6. **End centralized metadata**: Each microservice gets its own DB
7. **Concentrate the pain**: Tailored security and compliance for each microservice
8. **Test differently**: Clearer definition of scope leads to better unit testing

*Observations*

You're not doing microservices well if:
- Different services do coordinated deployments
- You ship client libraries
- A change in one service has unexpected consequences or requires a change in other services
- Services share a persistence store
- You cannot change your service's persistence tier without anyone caring
- Engineers need intimate knowledge of designs and schemeas of other teams' services
- You have compliance controls that apply uniformly to all services
- Your infrastructure isn't programmable
- You can't do one-click deployments and rollbacks

*Limitations*

Some of the proposed dividends of microservices do not manifest in practice. 
- It's difficult/time-consuming to reason about multiple failures scenarios. 
- In lieu of client libraries, devs often result in copying-pasting bug fixes to multiple services
- Forget about ACID transactions in microservices
- Microservice versioning can become a large hassle when deprecating APIs

*Future Directions*

Great video on how microservices can fail badly in practice: [10 Tips for failing badly at Microservices by David Schmitz](https://www.youtube.com/watch?v=X0tjziAQfNQ)


### [Contextual Understanding of Microservice Architecture: Current and Future Directions](https://dl.acm.org/doi/pdf/10.1145/3183628.3183631?download=true)

*Short Summary*

This paper discusses the main differences between Service-Oriented Architecture (SOA) and Microservice Architecture. It uses a ticketing web application to illustrate its points.

The main difference is that SOA relies on **orchestration** whereas Microservices rely on **choreography**. The former is centralized where all communication between services is routed to a Service Bus while the latter is decentralized and requires each service to install their own routing logic (or routing proxy).  

*Observations*

**SOA** = dumb services & smart pipes

Services can use any Layer-7 communication protocol (e.g XML, gRPC, HTTP) and can expect the same in response since the centralized Service Bus is responsible for translating between all the protocols. They also don't need to know the exact addresses of other services since the Service Bus is also responsible for that.

**Microservices** = smart services & dumb pipes

Services are mainly limited to the HTTP protocol and must be aware of the exact IP addresses of other services to communicate with them.

*Limitations*

In practice, it appears that industry prefers dumb services & smart pipes as shown by the wide adoption of Envoy and other service mesh solutions. The key difference is that the centralized Service Bus from SOA is decentralized in a service mesh, where each service has its own proxy. 

*Comaprison to Prior Papers*

The previous paper emphasized how Microservices allow for teams to independently develop and release their services. This paper illustrates some of the challenges with such a decentralized approach when it mentions how Microservices require "smart" services.

*Future Directions*

The paper discusses how SOA is known for its centralized management which can impede rapid development. It'd be interesting to see how Microservice deployment is opting for centralization in other forms via schedulers and service meshes. A purely decentralized system is hard to reason about and debug so there must be some standard amongst microservices that communicate with each other.


### [Design Patterns for Container-based Distributed Systems](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45406.pdf)

*Short Summary*

This paper discusses architectural patterns in distributed systems that have emerged from widespread adoption of Linux containers. These patterns essentially act as libraries for application developers. Such patterns include:

- Single-node, multi-container application patterns
  - **Sidecar**: Access local resources from application container (e.g Application writes logs to local disk and log saver sidecar reads from local disk to send to logging system)
  - **Ambassador**: Proxy communication to and from an application container (e.g Envoy)
  - **Adapter**: Standardize output and interfaces (e.g Monitoring sidecar that translates application metrics to a standard form)
- Multi-Node application patterns
  - **Leader election**: Each application container is paired with a leader-election container. The LE container exposes an API to the application container to notify of its current state in the cluster
  - **Work queue**: Each application is paired with a work-receiver container that pulls work from a centralized work coordinator container. Applications are only tasked with developing work execution logic.
  - **Scatter/gather**: Application developer is provided with a coordinator container that sends, receives, and coordinates requests to other downstream application containers. (e.g Sagas coordinator)

*Observations*

All of the single-node patterns are very useful in a microservice architecture. It seems that the industry direction is to provide more functionality in these "smart pipes" so that services can become "dumber."

As for the multi-node patterns, it seems that many companies elect to provide Leader Election or Work Queueing as a centralized service, such as Zookeeper or Chubby for Leader Election and SQS, Kafka, or RabbitMQ for work queueing. 

*Limitations*

For the ambassador and adapter patterns, it is still up to the developer to continuously update the proxy logic for new edge cases as they appear. Furthermore, these proxies are often 3rd-party applications such as Envoy. To add additional functionality, it might be difficult to decipher their code-base and build new internal versions of open-source software.

*Comaprison to Prior Papers*

This paper provides some real world examples of how decentralized microservices can communicate with each other without extensive application logic.

*Future Directions*

These patterns often rely on sidecar proxies to provide extensive communication, logging, and monitoring functionality. What are the challenges/solutions to doing cluster-wide updates on these proxies? 

### [Putting the "Micro" Back in Microservices](https://www.usenix.org/system/files/conference/atc18/atc18-boucher.pdf)

*Short Summary*

FaaS (Functions as a Service) is becoming increasingly because it offers "limitless" scalability with at in instant. Cloud providers have adopted container-based isolation, where each function runs in its own container. This approach makes it difficult to meet demanding latency requirements due to the speed of cold-starting Linux containers.

This paper proposes language level isolation using Rust for FaaS. By using dispatcher and worker processes that import user's **safe** Rust code, and installing a micro-second task pre-empter, the authors were able to achieve **10x** faster Warm-start times and **100x** faster Cold-start times.

*Observations*

- **Rust**
  
  The authors specifically use Rust in this paper because the language design and compiler are incredibly strict to prevent most sources of segmentation faults (e.g accessing null object's field, excessive pointer manipulation, etc). This ensures that user code does not crash worker processes. 

- **Preemption**
  
  Since serverless functions are often expected to complete incredibly quickly, the authors decided to implement their own micro-second scale process preemption. Based on a predefined SLA, the scheduler is responsible for preempting and cleaning tasks that run longer than the bounded requirement.

*Limitations*

Obviously security is a huge limitation in this system since you are allowing user code to have access to system calls. While the authors touch on this extensively, their mentioning of manual approval of popular Rust libraries that use unsafe code and a decent list of uncovered, potential security exploits seems that this solution might be hard to scale for a large cloud provider.

*Comaprison to Prior Papers*

This paper is much more lower-level than the others we've covered so far. As cloud providers abstract more infrastructure away from end-users, the need to optimize OS/Kernel level code becomes more paramount. 

*Future Directions*

- This paper puts a lot of emphasis on microsecond preemption, but serverless functions often make API calls to databases whose response times are in the range of milliseconds. 
- Why not run these worker processes in containers? Wouldn't that solve many security issues?

### [The Architectural Implications of Microservices in the Cloud](https://arxiv.org/pdf/1805.10351.pdf)

*Short Summary*

This paper analyzes microservice performance on 40-core Intel servers to extrapolate meaningful data about processor utilization, CPU cycles, I-cache pressure, and other machine metrics. 

Based on their experiments, the authors conclude that microservices are heavily suited to machines that optimize for single-thread performance and general CPU-level optimizations are difficult. 

*Observations*

The brawny vs wimpy cores section near the end of the paper is particularly interesting when contemplating how cloud providers might design their next generation datacenters. For instance, Facebook has noted in multiple tech talks that their server fleet comprises of mostly small, low-powered machines. It seems that this paper supports Facebook's approach now that microservices are industry-standard.    

*Limitations*

This paper noted in multiple sections that it was hard to propose a general optimization to increase performance of cloud microservices, especially since a variety of microservices are often packed onto the same node.

*Comaprison to Prior Papers*

This paper measures the CPU performance of the microservice patterns established in previous papers. 

*Future Directions*

It would be interesting to see how datacenter architects could design specific clusters to handle specific workloads. Then it would be up to the cluster scheduler to put microservices in machines optimized for their workload.

### [Large-scale Cluster Management at Google with Borg](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf)

*Short Summary*


*Observations*


*Limitations*


*Comaprison to Prior Papers*


*Future Directions*


### [Synthesizing Cluster Management Code for Distributed Systems](http://ryzhyk.net/publications/weave_hotos19.pdf)

*Short Summary*

Cluster schedulers often depend on hand-crafted heuristics to find the most optimum servers to place jobs or services on. This leads to thousands LOC that deal with each specific combination between hard and soft constraints as evidenced by the Kubernetes Pod Scheduler.

Weave, a framework that converts SQL cluster policies to linear equations, tackles the scheduling problem by using already-optimized math software to best solve this *NP-hard combinatorial optimization problem*.

*Observations*

- **Ease of use for application developers**
  
  In existing cluster schedulers, users are either limited by the scope of policy language or must use complex ad-hoc solutions to achieve tailored cluster policies. Weave's use of SQL as its policy language allows developer to reason about containers' hard and soft constraints via familiar concepts such as joins and nested select statements.

- **Powerful compiler**
  
  Cluster scheduling is a combinatorial optimization problem at its core where one must meet all hard constraints while minimizing the penalty soft constraints. Weave's compiler performs the non-trivial task of translating user queries about cluster state into math variables and scalars to feed into dedicated linear equation solvers. As one might imagine, this approach should be far more extensible than hard-coding new combinations in the scheduler logic.

*Limitations*

- **Resource Reclamation & Preemption**
  
  Although Weave can optimize the cluster state very well, it lacks the ability to modify the cluster on-the-fly based on monitoring metrics. This is a crucial part of Kubernetes and Borg in which pods can be evicted or relocated based on higher-priority jobs requiring allocation or more resources.

- **Conflict Resolution**
  
  Since Weave takes in both cluster state and constraints as input from the user, the runtime becomes the bottleneck when multiple users submit jobs concurrently. One could imagine scenarios where the inputted cluster state is not up to date, which could lead to inconsistent states in the cluster. 

*Comaprison to Prior Papers*

Compared to Borg, Weave presents a much more algorithmic approach towards cluster scheduling. Its advantage is that it can generalize to more scenarios and is more user-friendly. However, unlike Borg, Weave makes the assumption that users are always telling the truth. As noted by the Borg paper, users will often try to game the system for the scheduler to place their jobs. Right now, only hand-crafted solutions account for this real-world scenario whereas Weave does not.

*Future Directions*

Is it possible to design this system so that it continuously polls the cluster state and is able to reclaim resources and preempt pods?

### [Service Fabric: A Distributed Platform for Building Microservices in the Cloud](https://dl.acm.org/doi/pdf/10.1145/3190508.3190546?download=true)

*Short Summary*

Service Fabric is Microsoft's cluster scheduler and service mesh solution. Unlike many popular solutions today, such as Kubernetes, Istio, Nomad, Mesos, etc, Service Fabric uses a fully decentralized approach with no centralized metadata store or control plane.

*Observations*

1. **Federation Subsystem**
   
   This system describes how Microsoft keeps node membership consistent across thousands of machines. They employ an algorithm similar to Chord and Pastry except with bidirectional routing. Neighborsets are determined by adjacent clockwise and counter-clockwise nodes which leads to *symmetrical* routing between nodes. This symmetrical routing, while enabling faster proliferation of cluster membership, could lead to cascading failures when nodes leave the cluster. To resolve this, Service Fabric utilizes Arbitrator Groups to make decisions on cluster membership. 

2. **Reliability Subsystem** 
   
   This system is responsible for replicating services across nodes and load balancing. It mainly consists of the Failover Manager, which serves as the cluster scheduler for the Service Fabric ring. It is also responsible for naming and service discovery. Essentially, the Federation subsystem deals purely with mapping nodes and services consistently to the SF ring while the Reliability Subsystem ensures that services are fault tolerant and load balanced via replication. 

3. **Reliable Collections**
   
   Because Service Fabric is comprised of consistent components from the ground-up, application developers can build stateful applications on top of SF. The Failover Manager replicates stateful applications and executes leader election automatically. Due to this architecture, application devs benefit from improved latency since reads are local rather than via an API to some dedicated DB servers.

*Limitations*



*Comaprison to Prior Papers*


*Future Directions*