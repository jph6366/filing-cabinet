Monolithic applications are designed to handle multiple related tasks. A software application that is designed as a single service.
- more simple than correct interface
- one tightly-coupled code repository for all components
- code-base scalability limited by vertical scaling cost and none to little usable modularity.
##### Scaling
- **Horizontal**: Requests are handled by a load balancer, which then distributes the requests to the application.
- **Vertical**: Increasing CPU, Memory, Network, Storage, etc, for higher performance, allows you to perform more work and handle more load.
##### Load Balancing
- we can create load balancing rules to create groups of servers called **Target Groups** that will only handle requests for a specific boundary. 
	- You can decide how to scale and route traffic with a load balancer to a target-group that handles that boundary
### Modular Monolith
a software design approach in which a monolith is designed with an emphasis on interchangeable (and potentially reusable) modules.
- blends the simplicity and robustness of traditional monoliths with the flexibility and scalability of micro-services.
A well-structured modular monolith offers a clear path to a micro-services architecture. You can gradually extract modules into separate services when the need arises.
#### communication pattern
1. Synchronous Communication With Method Calls
   - synchronous in-memory method calls between modules
   - each module exposes a public API 
	   - modules depend on the interfaces at compile-time. At runtime, dependency injection will provide the respective implementation.
   - Introduces strong coupling and fault tolerance debt
2. Asynchronous Communication With Messaging
   - asynchronous messaging between modules
   - Message contracts are the public API of a module
   - Introduces increased complexity and infrastructure debt
