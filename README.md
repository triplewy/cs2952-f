# CS2952-F Papers

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

This paper describes Google's approach to scheduling jobs across their fleet of servers. Some of the most notable challenges Google faced in this endeavor was increasing resource utilization, defining a scheduling algorithm that respects soft and hard constraints, and providing fault tolerance.

*Observations*

- **Architecture**
  
  For each Google cell, which consists of at least 10,000 machines, a cluster of Borgmasters are responsible for storing all the cluster config events. Each borgmaster consists of a persistent paxos store to ensure durability and high availability. A separate process called the scheduler frequently pools for new config events from the Borgmasters and updates the cluster state. The scheduler sends the cluster state back to the Borgmasters who then communicate with each Borglet. Each machine in the cell contains an agent called the Borglet that receives scheduling requests from the Borgmaster and reports back monitoring metrics via heartbeats. 

- **Utilization**
  
  After much experimentation, the Borg developers chose *cell compaction* as their primary scheduling metric. This measures the smallest cell a scheduler places a certain workload in. The authors used **Fauxmaster**, a complete simulator of the Borgmaster, to test recorded production workloads numerous times to ensure consistent results. In their conclusion, they stated that a combination of best and worst fit gave the best results. 

*Limitations*

Borg is known for catering toward power users, and thus provides a wide array of options in their configuration language. For those who have experience with the system over numerous years, this API can provide the perfect scheduling constraints for long-running services and short-running jobs. However, this API impeded "casual" users from fully utilizing the system and most likely lead to inaccurate config policies from such users. 

*Comaprison to Prior Papers*

This is by far the biggest system we've covered so far and it's interesting to see its centralized architecture be able to scale to so many machines. Some of the earlier papers we read lauded the decentralized nature of microservices, but it seems that strong consistency and centralization is still much easier to comprehend, develop on top of, and is capable of scaling.

*Future Directions*

I'd love to learn more about the communication model between the borglets and the Borgmasters and the sharding mechanism of the Borgmasters. Google must have made numerous optimizations to scale such a system to tens of thousands of machines.

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

This paper touts its ability to support stateful applications, but who is responsible for moving storage disks when services get preempted? The paper also does not touch upon inter-datacenter clusters. Overall, fully decentralized distributed systems are incredibly difficult to program and the Microsoft team must have encountered numerous edge cases over the years when developing Service Fabric.

*Comaprison to Prior Papers*

Compared to Borg, Service Fabric employs a completely different approach to cluster management. The authors also mention the downsides of using lots of compute for the scheduling algorithm which disagrees with Weave's intention to generate the most optimal scheduling policy.

*Future Directions*

Microsoft employs many other datastore solutions such as Corfu, a distributed replicated log. Are these newer data-stores run on Service Fabric or do they have their own dedicated servers? Essentially, which core infrastructure services do not run on SF and why?

### [Hey You, Get Off My Cloud: Exploring Information Leakage in Third-Party Compute Clouds](https://hovav.net/ucsd/dist/cloudsec.pdf)

*Short Summary*

For cloud compute, cloud providers such as AWS, Microsoft Azure, GCP, etc provide instant-access VMs. Multiple VMs are packed inside the same physical machines to increase server utilization. However, this poses a security threat where attackers can potentially obtain a VM placement on the same physical machine as their target and then use VM side-channel attacks to steal sensitive information. This paper describes first, the patterns and heuristics of VM placement in AWS, and second, the potential ways to perform side-channel attacks.

*Observations*

Since this paper was written more than 10 years ago, it's interesting to see whether cloud providers have taken measures to prevent the security threats proposed in this paper. With the addition of dedicated instances on EC2, AWS now gives the customer a choice of being placed on their own machine, isolated from other VMs. This echoes the paper's suggestions in the conclusion where the authors stated that the best solution was to give customers a choice upfront. 

*Limitations*

The major limitation of this paper is that keystroke-timing experiment was run on a local environment rather than directly on EC2. Furthermore, their experiment depended on the machines being completely idle other than the test code which does not reflect real-world behavior.  

*Comaprison to Prior Papers*

This is the first paper we've read so far that dives deep into security threats that have arisen from the recent popularity of cloud computing. 

### [Performance Analysis of Cloud Applications](https://www.usenix.org/system/files/conference/nsdi18/nsdi18-ardelean.pdf)

*Short Summary*

This paper describes techniques to measure performance across hundreds or thousands of microservices in a real-world environment. Due to Google's unpredicatable and large request load, it's close to impossible to transfer performance results from a controlled test environment to the production environment. As a result, Google developed a bursty-coordinated kernel tracing framework to measure request latency across their service mesh.

*Observations*

- **Coordinated bursty tracing**
  
  Google partitions its users into tracing and non-tracing segments for services such as Gmail when performing coordinated bursty tracing. Due to Google's TrueTime technology, they can stitch together requests by simply using wall-clock time on each server. This allows Google to correlate latency across code layers.

- **Vertical Context Injection**
   
  Interleaving requests via wall-clock time is not enough to fully capture entire request paths. So, Google utilized existing kernel system calls to inject high-level events into the kernel trace. This allows them to link high-level problems with low-level problems and pinpoint the exact location of latency or utilization issues.

*Limitations*

The biggest limitation to this paper is Google's reliance on TrueTime technology. It makes their coordinated bursty tracing technique useless for any other large scale company.

*Comparison to Prior Papers*

This is the first paper that we've read that discusses context propagation as a means to evaluate microservice performance.

*Future Directions*

How does paper relate to Dapper?

*Clarification Questions*

Doesn't this approach still require a distributed tracing framework in order to stitch together RPCs? Time, even at microsecond level, is most likely not enough to fully contextualize an entire request. Vertical context tracing only relates processes within a single machine.

### [Overload Control for Scaling WeChat Microservices](https://www.cs.columbia.edu/~ruigu/papers/socc18-final100.pdf)

*Short Summary*

Tencent has developed a service-agnostic, decentralized, and cooperative load-shedding framework to effectively control overloaded servers. 

*Observations*

- **Overload Detection**
  
  Tencent utilizes per-server request queueing time as opposed to request latency or per-server CPU utilization to measure server load. This is a great metric since request latency for upstream services depends on downstream services, which makes it a non-independent variable. CPU utilization is also not useful since high CPU utilization does not necessarily equate to difficulties in servicing requests.

- **Service Admission Control**
  
  Tencent uses Business-level and User-level priorities when rejecting requests at overloaded servers. Each request's business and user priorities are set at the entry service and subsequently propagated for the rest of its lifetime. This allows for more important requests to finish in high-load scenarios.

- **Collaborative Admission Control**
  
  Even with fine-grained admission control, requests are still prone to be rejected midway since each server could be experiencing different load. To prevent this waste of network I/O, downstream services piggyback their admission control to upstream servers in their responses. This serves as a scalable method to propagate admission control across services. 

*Limitations*

Tencent does not discuss service failure or potential admission control inconsistencies for downstream services. For instance, if a service is load balanced across multiple servers, each server could have a different load. However, DAGOR does service-level collaborative admission control rather than server-level. Since routing is not discussed in the paper, we don't know how Tencent deals with these inconsistencies.

*Comparison to Prior Papers*

This paper relates to cluster schedulers in detecting varying resource utilization among machines. However, instead of scheduling containers across machines, DAGOR "schedules" requests across services.

*Future Directions*

How does this approach cooperate with cluster schedulers which can evict services based on heuristics such as CPU utilization, memory resources, etc. Is it worthwhile to migrate services on overloaded servers? 

*Clarification Questions*

How does each upstream server load balance to downstream services? 

### [Design and Implementation of Hyperupcalls](https://www.usenix.org/system/files/conference/atc18/atc18-amit.pdf)

*Short Summary*

VMWare has implemented an alternative to paravirtualization that allows guests VMs to respond to events from the host kernel via hyperupcalls. Hyperupcalls are verified programs written by guests that are stored in the hypervisor and executed on certain hypervisor events. To summarize, "Hyperupcalls \[allow\] the guest to expose its *logic* to the hypervisor, avoiding a context switch by enabling the hyperupcall to safely execute guest logic directly."

*Observations*

- **eBPF**
  
  The extended Berkely Packet Filter is used extensively to verify and compile guest program code (written in C) into native machine bytecode. VMWare also had to include additional features to safeguard memory aspects and translate addresses between guests and the hypervisor. 

- **Use cases**
  
  One easy-to-comprehend use case for hyperupcalls is when the hypervisor frees memory from the guest. Information on freed memory pages in the guest is unavailable to the hypervisor without paravirtualization or hyperupcalls. According to VMWare's experiments, the hypervisor experienced up to 8x increase in performance under "memory ballooning" events when using hyperupcalls.


*Limitations*

This framework could introduce new potential security threats. Side-channel attacks come to mind if guests can infer some state from the hypervisor via hyperupcalls.

*Comparison to Prior Papers*

FINELAME also used eBPF extensively in their DDOS prevention framework. It seems like eBPF is quite an useful tool.

*Future Directions*

Can this cooperation between guests and the hypervisor increase VM performance over containers? Since hyperupcalls are precompiled, this should not have an impact on bootup time.

*Clarification Questions*

Is this framework mainly inteded for cloud providers to use?

### [CloudTalk: Enabling Distributed Application Optimisations in the Public Cloud](http://nets.cs.pub.ro/~costin/files/cloudtalk.pdf)

*Short Summary*

Increase cloud application performance by understanding underlying cloud network topology. Because Cloud providers obscure topology to customers, distributed applications frequently communicate over clogged switches and hot paths. 

*Observations*

- **Monitoring Load**
  
  The CloudTalk server will query hundres or thousands of VMs for their current I/O load. This can be both network and disk load. The authors make two key contributions:
  - 1. CloudTalk only needs to communicate with a random subset of VMs (when the cluster grows to thousands of nodes) to ensure optimal data transmission
  - 2. Local monitoring is enough to ensure optimal paths since cloud providers such as AWS employ full-bisection topologies where bottlenecks only occur at the end-hosts links.

- **Novel Dataflow Language**
  
  The purpose of the CloudTalk language is to model the network topology and provide optimal and fast answers to NP-Hard scheduling questions (e.g which worker node should receive work based on its current dataflows). The authors decided to employ a new language in CloudTalk to center around these network and disk dataflows.

*Limitations*

In a multi-tenant environment, network IO can fluctuate rapidly depending. CloudTalk suffers from stale information when IO state is communicated back to the CloudTalk server from other nodes in the cluster.

*Comparison to Prior Papers*

The authors employed similar techniques to the *Hey you, Get Off My Cloud* paper to discover VM placement across nodes on AWS EC2 (namely counting network hops via tracerouting). 

*Future Directions*

Most of the paper's experiments deal with computation frameworks such as Hadoop. In these scenarios, time of completion is not as crucial as online services with SLAs. Can CloudTalk be effectively used in a service mesh to provide increased network performance between microservices?

*Clarification Questions*

None

### [Dapper, A Large-Scale Distributed Systems Tracing Infrastructure](https://static.googleusercontent.com/media/research.google.com/en//archive/papers/dapper-2010-1.pdf)

*Short Summary*

Dapper is an out-of-band tracing system that relies on underlying language RPC libraries and use of sampling to scale out to practically every service running at Google.

*Observations*

- **Out-of-band Processing**
  
  Dapper collects and processes traces in 3 steps. First, services write to local log files on the host machine. Second, a dapper dameon running on each machine constantly pulls from these files and sends the logs to collectors. These collectors are "out-of-band" in that they are not on the same machines as online services. Lastly, the collectors send logs to their corresponding Bigtable shard where traces can then be queried.

- **Adaptive Sampling**
  
  Google employs an adaptive sampling scheme that is "parameterized not by a uniform sampling probability, but by a desired rate of sampled traces per unit time." This seems like a much better alternative to a constant trace rate across the entire cluster, which is the most common form of sampling across tracing systems. It also seems that these rates are being monitored at the gateways of such services since it would be useless to collect partial traces.

*Limitations*

Adaptive sampling seems much more complicated in practice since some downstream services may receive more samples than necessary, thus decreasing performance. However, sampling at each service, regardless of how upstream or downstream they are, leads to incomplete traces which is also unacceptable. 

*Comparison to Prior Papers*

This paper is mentioned in *Principled workflow-centric tracing of distributed systems* and is shown as an example of a tracing system that is best at identifying workflows w/ steady-state problems. In other words, Dapper is best at identifying performance degradations amongst services.

*Future Directions*

Further exploring better alternatives to sampling in distributed tracing would be fairly interesting. This similarly resembles WeChat's overloaded microservices architecture in their attempt to prioritize certain requests over others. The same could be done for tracing systems where some traces or requests are deemed more necessary to record.

*Clarification Questions*

None

### [Principled Workflow-Centric Tracing of Distributed Systems](https://www.rajasambasivan.com/wp-content/uploads/2017/07/sambasivan-socc16.pdf)

*Short Summary*



*Observations*


*Limitations*


*Comparison to Prior Papers*


*Future Directions*


*Clarification Questions*

None