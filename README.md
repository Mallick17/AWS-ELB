# AWS Elastic Load Balancer
Elastic Load Balancing (ELB) is a managed service provided by Amazon Web Services (AWS), designed to automatically distribute incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, and IP addresses, in one or more Availability Zones. This distribution enhances the fault tolerance and availability of applications by ensuring no single server is overwhelmed, which is critical for maintaining performance during traffic spikes. ELB monitors the health of registered targets and routes traffic only to healthy ones, automatically scaling the load balancer capacity in response to changes in incoming traffic.

### Types of ELB
ELB has several types, each for different needs:
- **Application Load Balancer (ALB)**: Best for web apps, supports HTTP/HTTPS, and is great for microservices.
- **Network Load Balancer (NLB)**: Ideal for high-performance apps, like gaming, with support for TCP/UDP.
- **Gateway Load Balancer (GWLB)**: Used for network traffic, like firewalls, managing traffic at a lower level.
- **Classic Load Balancer**: A legacy option, mainly for older systems in EC2-Classic networks.

## How ELB Works:
ELB operates by distributing workloads across compute resources, such as virtual servers, to optimize performance and reliability. It employs configurable health checks to monitor the health of targets, ensuring that traffic is routed only to healthy resources. This is particularly important for maintaining application uptime, as unhealthy instances are automatically excluded from the traffic flow. ELB also offloads tasks like encryption and decryption to the load balancer, allowing compute resources to focus on core application tasks, which improves efficiency.

A load balancer is like a traffic manager for your servers. Its main job is to take incoming requests from clients (like web browsers or apps) and distribute them across a group of servers (called targets, such as Amazon EC2 instances) to ensure no single server gets overwhelmed. It also keeps an eye on the health of these servers and only sends traffic to the ones that are working properly. Here’s how it all comes together:

### Core Functionality
1. **Accepting Traffic**: The load balancer sits in front of your servers and acts as the entry point for all incoming client requests. You set it up by defining *listeners*, which are rules telling the load balancer what kind of traffic to expect (e.g., HTTP on port 80) and where to send it (e.g., to your servers on port 8080). A listener specifies the protocol (like HTTP or TCP) and port for both the client-to-load-balancer connection and the load-balancer-to-target connection.

2. **Routing to Targets**: Once a request comes in, the load balancer forwards it to one of its registered targets. These targets are typically servers running in one or more *Availability Zones* (AZs), which are isolated locations within a data center region (e.g., us-east-1a, us-east-1b). The load balancer decides which target gets the request based on its configuration and routing rules.

3. **Health Monitoring**: The load balancer constantly checks the health of its targets using health checks (e.g., pinging a specific endpoint like `/health`). If a target fails the health check (say, it’s down or unresponsive), the load balancer stops sending traffic to it. Once the target passes the health check again, traffic resumes. This ensures clients only interact with healthy servers.

### Availability Zones and Load Balancer Nodes
- **Nodes in AZs**: When you enable an Availability Zone for your load balancer, a *load balancer node* is created in that AZ. Think of a node as a part of the load balancer that lives in that specific zone and handles traffic distribution. For example, if you enable two AZs, you get two nodes—one in each zone.
- **Target Registration**: You also register your servers (targets) with the load balancer, specifying which AZ they’re in. If you register a target in an AZ but don’t enable that AZ for the load balancer, no traffic goes to that target because there’s no node to route it.
- **High Availability**: For the load balancer to work effectively (and avoid downtime), you should enable multiple AZs. With an *Application Load Balancer* (ALB), you *must* enable at least two AZs—it’s a requirement. If one AZ goes down or has no healthy targets, the load balancer can still send traffic to targets in other AZs, keeping your service up and running.
- **Disabling an AZ**: If you disable an AZ, the targets there stay registered, but they won’t get traffic because there’s no active node in that zone to handle requests.

### Why Use Load Balancers?
Load balancers enhance availability by distributing traffic, improve scalability by adding capacity, and ensure fault tolerance by routing around unhealthy targets. They are critical for high-traffic applications, supporting DevOps goals of automation and resilience.

### Comparative Table

| Load Balancer Type | Layer | Protocols Supported | Key Features | Primary Use Cases |
|--------------------|-------|---------------------|--------------|-------------------|
| Application (ALB)  | 7     | HTTP, HTTPS         | Content-based routing, WAF, Lambda | Web apps, microservices |
| Network (NLB)      | 4     | TCP, UDP, TLS       | High performance, static IPs | Gaming, streaming, IoT |
| Gateway (GLB)      | 3     | GENEVE              | Virtual appliances, traffic inspection | Network security |
| Classic (CLB)      | 4, 7  | HTTP, HTTPS, TCP, SSL | Basic load balancing | Legacy applications |

- This table highlights the diversity, with ALB for application-level routing and NLB for network-level performance, GLB for appliances, and CLB for legacy needs.





