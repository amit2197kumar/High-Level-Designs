# Horizontal vs Vertical Scaling

**Horizontal scaling** is adding more machines to deal with increasing requirements. These machines handle requests in parallel to improve user experience.

**Vertical scaling** is replacing the current machines with more advanced machines to improve throughput and hence response time. The techniques are used in conjunction in real world systems.

---

Horizontal and vertical scaling are two approaches to increasing the capacity and performance of a system, typically in the context of computing and database systems. Here's a detailed explanation of both:

### Horizontal Scaling (Scale Out)

**Definition:** Horizontal scaling, also known as scaling out, involves adding more machines or nodes to a system. This means you increase the number of servers or instances that work together to handle the load.

**Advantages:**

1. **Redundancy and Resilience:** Adding more nodes can provide redundancy, so if one node fails, others can take over, enhancing system reliability.
2. **Cost-Effective:** Often more cost-effective as commodity hardware can be used, and you can add resources incrementally.
3. **Improved Performance:** By distributing the load across multiple nodes, the system can handle more simultaneous requests.
4. **Flexibility:** Easier to manage traffic spikes and scale out based on demand.

**Challenges:**

1. **Complexity:** Managing and orchestrating multiple nodes can be complex, requiring sophisticated load balancing and distributed computing strategies.
2. **Consistency:** Maintaining data consistency across multiple nodes can be challenging, often requiring distributed databases or coordination mechanisms like distributed consensus algorithms.

**Examples:**

- Adding more web servers behind a load balancer to handle increased web traffic.
- Using distributed databases like Cassandra or MongoDB that can scale horizontally by adding more nodes.

### Vertical Scaling (Scale Up)

**Definition:** Vertical scaling, also known as scaling up, involves adding more power (CPU, RAM, storage) to an existing machine. This means enhancing the capacity of a single server or instance.

**Advantages:**

1. **Simplicity:** Easier to implement since you are upgrading existing hardware rather than managing multiple machines.
2. **Consistency:** Data consistency is simpler to maintain as it is stored on a single node.
3. **Performance:** For certain applications, single-machine performance can be more efficient due to reduced inter-node communication overhead.

**Challenges:**

1. **Cost:** Upgrading to more powerful hardware can be expensive, and there are physical limits to how much you can scale up.
2. **Downtime:** Scaling up often requires downtime to upgrade hardware or migrate to a more powerful machine.
3. **Single Point of Failure:** If the scaled-up server fails, it can lead to significant downtime and data loss.

**Examples:**

- Increasing the RAM and CPU of a database server to handle more transactions.
- Upgrading a server's storage to SSDs for faster read/write speeds.

### Comparison

| Aspect | Horizontal Scaling | Vertical Scaling |
| --- | --- | --- |
| Approach | Adding more machines/nodes | Adding more resources to a single machine |
| Cost | Can be more cost-effective with commodity hardware | Can be expensive with high-end hardware |
| Complexity | Higher, requires managing distributed systems | Lower, simpler as it involves upgrading existing systems |
| Fault Tolerance | Higher, better redundancy and failover | Lower, single point of failure risk |
| Performance | Can handle more requests simultaneously | Can handle larger individual requests |
| Scalability | Virtually unlimited, easier to scale out | Limited by the hardware's maximum capacity |

In practice, many modern systems use a combination of both horizontal and vertical scaling to optimize performance, reliability, and cost-efficiency. For instance, a web application might use vertical scaling to enhance the performance of its database server while employing horizontal scaling to manage web traffic across multiple web servers.

---

### **Horizontal Scaling a Monolith**

**Scenario:** Imagine a traditional e-commerce application built as a monolith, where all functionalities (product catalog, order processing, payment, user management) are part of a single, unified codebase and share the same database.

**Challenges:**

1. State Management Suppose the application uses session-based authentication. If a user logs in on one instance and the next request is routed to a different instance (due to load balancing), the new instance may not recognize the user's session. Solutions like sticky sessions or distributed session management become necessary.
2. Resource Utilization The application might have components (like the product catalog) that are more heavily used than others (like payment processing). When scaling horizontally, each new instance includes all components, even the less frequently used ones, leading to inefficient resource use.
3. Deployment Complexity Deploying a new version of the monolith requires updating all instances simultaneously. This can lead to significant downtime or require complex rolling update strategies.
4. Load Balancing The application needs an effective load balancer to distribute incoming traffic evenly across all instances, ensuring no single instance becomes a bottleneck.

**Monolithic Architecture:**

1. State Management: Distributed Cache: Technologies like Redis or Memcached are used to manage session state across multiple instances of a monolith.
2. Resource Utilization: Containerization: Using Docker or similar technologies to containerize a monolith can make it more efficient to deploy and scale on demand.
3. Deployment Complexity:
- Blue-Green Deployment: This technique involves running two identical production environments, only one of which serves live production traffic. It allows for smooth deployment and easy rollback in case of issues.
- Canary Releases: Gradually rolling out changes to a small subset of users before making them available to everybody.
1. Load Balancing: Advanced Load Balancers: Using sophisticated load balancers (like AWS ELB, NGINX, or HAProxy) that can handle sticky sessions and distribute traffic effectively.

### **Horizontal Scaling Microservices**

Scenario: Consider the same e-commerce application, but now re-architected into microservices, **with separate services for product catalog, order processing, user management, and payment processing.**

**Challenges:**

1. Service Discovery As you scale out the order processing service to handle high demand, the product catalog service must dynamically discover these new instances to communicate with them for stock checks.
2. Network Latency The user management service might need to communicate with the order processing service. More instances could mean increased network calls, leading to higher latency.
3. Data Integrity and Transaction Management If a customer places an order, this transaction might involve the order service, payment service, and inventory service. Ensuring consistency across these services if one part of the transaction fails (like payment failure) is complex.
4. Monitoring and Logging Each microservice generates its own logs. With more instances, aggregating these logs into a central monitoring tool becomes crucial to diagnose issues.
5. Dependency Management If the payment processing service is scaled up due to high demand, its dependent services, like the order processing service, might also need to scale up, leading to a cascading effect.

### **Microservices Architecture**

1. Service Discovery:
- Service Mesh: Implementing a service mesh like Istio or Linkerd helps in managing service discovery, load balancing, and failure recovery in microservices.
- Consul or Eureka: These tools provide a DNS or HTTP interface to discover the IPs of services.
1. Network Latency:
- API Gateway: Using an API gateway to optimize request routing and reduce network hops.
- Asynchronous Communication: Implementing message queues like RabbitMQ or Kafka for communication between services.
1. Data Integrity and Transaction Management:
- Distributed Transaction Patterns: Implementing patterns like Saga which manage transactions across multiple microservices.
- Eventual Consistency: Accepting that the system won't always be consistent at all times and designing for eventual consistency.
1. Monitoring and Logging:
- Centralized Logging: Tools like ELK Stack (Elasticsearch, Logstash, Kibana) or Splunk for aggregating logs from all microservices.
- Distributed Tracing: Using tools like Jaeger or Zipkin for tracing requests across multiple services.
1. Dependency Management:
- Versioning: Carefully managing service versions and dependencies.
- Container Orchestration: Tools like Kubernetes for managing and scaling dependent services effectively.

### **Common Solutions for Both Architectures**

- Infrastructure as Code (IaC): Tools like Terraform or AWS CloudFormation for automating the provisioning and scaling of infrastructure.
- Continuous Integration and Continuous Deployment (CI/CD): Jenkins, GitLab CI, or GitHub Actions for automating the deployment process.
- Scalable Cloud Infrastructure: Utilizing cloud services (AWS, Azure, Google Cloud) that offer auto-scaling and efficient resource management.
- Security: Implementing robust security practices including regular security audits, using Web Application Firewalls (WAF), and ensuring all instances are updated with the latest security patches.

---

In case of horizontal scaling different servers need to interact with each other which involves RPC (newtork calls) which are slow. why do different servers need to interact with each other in the first place? They are essentially same services deployed on different servers, right? So shouldn't the interaction be just between clients and servers and not among servers themselves?

There can be different reasons , they need to communicate with each other. One reason is to maintain the data consistency, if say user profile is updated in one server instance that needs to be replicated in other instances too. It can be for session state management or caching too , as you are not guaranteed that you request will be always redirected to same server (unless you are using sticky session), so other servers will need you session and cached data too. In microservice architecture you may decide to deploy different services in different servers to endure fault tolerance which will lead to server to server communications. There can be other scenarios too.
