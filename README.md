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

*Comaprison to Prior Papers*

N/A

*Future Directions*

Great video on how microservices can fail badly in practice: [10 Tips for failing badly at Microservices by David Schmitz](https://www.youtube.com/watch?v=X0tjziAQfNQ)

*Clarification Questions*

N/A

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

*Future Directions*

*Clarification Questions*

### Design Patterns for Container-based Distributed Systems

- Short Summary of the Paper:


- Discuss aspects of the paper that you found interesting. These could be observations, assumptions, design choices, or insights introduced by this paper:



- Describe Limitations of the Proposed Systems/Measurements/Vision:

    Some of the proposed dividends of microservices do not manifest in practice.

- Discuss this Paper Relative to Prior Papers:

- Future Directions (How would you extend this paper? Use bullet points to enumerate your directions):

- Clarification Questions About the paper that you wanted answered in class. (Use bullet points to enumerate each question):


### Design Patterns for Container-based Distributed Systems

- Short Summary of the Paper:


- Discuss aspects of the paper that you found interesting. These could be observations, assumptions, design choices, or insights introduced by this paper:



- Describe Limitations of the Proposed Systems/Measurements/Vision:

    Some of the proposed dividends of microservices do not manifest in practice.

- Discuss this Paper Relative to Prior Papers:

- Future Directions (How would you extend this paper? Use bullet points to enumerate your directions):

- Clarification Questions About the paper that you wanted answered in class. (Use bullet points to enumerate each question):

### Putting the "Micro" Back in Microservices

- Short Summary of the Paper:


- Discuss aspects of the paper that you found interesting. These could be observations, assumptions, design choices, or insights introduced by this paper:



- Describe Limitations of the Proposed Systems/Measurements/Vision:

    Some of the proposed dividends of microservices do not manifest in practice.

- Discuss this Paper Relative to Prior Papers:

- Future Directions (How would you extend this paper? Use bullet points to enumerate your directions):

- Clarification Questions About the paper that you wanted answered in class. (Use bullet points to enumerate each question):

### The Architectural Implications of Microservices in the Cloud

- Short Summary of the Paper:


- Discuss aspects of the paper that you found interesting. These could be observations, assumptions, design choices, or insights introduced by this paper:



- Describe Limitations of the Proposed Systems/Measurements/Vision:

    Some of the proposed dividends of microservices do not manifest in practice.

- Discuss this Paper Relative to Prior Papers:

- Future Directions (How would you extend this paper? Use bullet points to enumerate your directions):

- Clarification Questions About the paper that you wanted answered in class. (Use bullet points to enumerate each question):

### Large-scale Cluster Management at Google with Borg

- Short Summary of the Paper:


- Discuss aspects of the paper that you found interesting. These could be observations, assumptions, design choices, or insights introduced by this paper:



- Describe Limitations of the Proposed Systems/Measurements/Vision:

    Some of the proposed dividends of microservices do not manifest in practice.

- Discuss this Paper Relative to Prior Papers:

- Future Directions (How would you extend this paper? Use bullet points to enumerate your directions):

- Clarification Questions About the paper that you wanted answered in class. (Use bullet points to enumerate each question):

### Synthesizing Cluster Management Code for Distributed Systems

- Short Summary of the Paper:


- Discuss aspects of the paper that you found interesting. These could be observations, assumptions, design choices, or insights introduced by this paper:



- Describe Limitations of the Proposed Systems/Measurements/Vision:

    Some of the proposed dividends of microservices do not manifest in practice.

- Discuss this Paper Relative to Prior Papers:

- Future Directions (How would you extend this paper? Use bullet points to enumerate your directions):

- Clarification Questions About the paper that you wanted answered in class. (Use bullet points to enumerate each question):

### The Architectural Implications of Microservices in the Cloud

- Short Summary of the Paper:


- Discuss aspects of the paper that you found interesting. These could be observations, assumptions, design choices, or insights introduced by this paper:



- Describe Limitations of the Proposed Systems/Measurements/Vision:

    Some of the proposed dividends of microservices do not manifest in practice.

- Discuss this Paper Relative to Prior Papers:

- Future Directions (How would you extend this paper? Use bullet points to enumerate your directions):

- Clarification Questions About the paper that you wanted answered in class. (Use bullet points to enumerate each question):