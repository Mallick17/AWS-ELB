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

### Cross-Zone Load Balancing
This is where things get interesting—it’s about how traffic gets spread across targets in different AZs. Each load balancer node handles a portion of the incoming traffic (determined by DNS, via Amazon Route 53, which points clients to node IP addresses). The default routing method is *round robin*, meaning requests are handed out one-by-one in a circular order. Cross-zone load balancing decides whether a node can send traffic to targets in *all* AZs or just its own.

#### Scenario: Two AZs, 10 Targets
Imagine you have two AZs:
- AZ A: 2 targets
- AZ B: 8 targets
- Total: 10 targets
- Two load balancer nodes (one in each AZ), each getting 50% of the client traffic.

##### Cross-Zone Load Balancing Enabled
- Each node can send its traffic (50%) to *all 10 targets* across both AZs.
- The traffic is evenly split: each target gets 10% of the total traffic.
- Why? Because the nodes don’t care about AZ boundaries—they distribute traffic globally across all healthy targets using round robin.
- Result: 2 targets in AZ A and 8 in AZ B each handle 10% of the load, perfectly balanced.

##### Cross-Zone Load Balancing Disabled
- Each node only sends its traffic (50%) to targets in *its own AZ*.
- AZ A node: 50% of traffic split across 2 targets → each gets 25%.
- AZ B node: 50% of traffic split across 8 targets → each gets 6.25% (50 ÷ 8).
- Result: Uneven distribution. Targets in AZ A handle more load per server (25%) than those in AZ B (6.25%), because each node is limited to its own zone.

#### Load Balancer Types and Defaults
- **Application Load Balancer (ALB)**: Cross-zone load balancing is *always on* at the load balancer level. However, you can turn it off at the *target group* level (a group of targets with specific routing rules). This gives you flexibility for specific use cases.
- **Network Load Balancer (NLB) and Gateway Load Balancer**: Cross-zone load balancing is *off by default*. You can enable or disable it anytime after creation.
- **Classic Load Balancer**: The default depends on how you set it up. Via the AWS Console, it’s *on* by default; via API/CLI, it’s *off*. You can toggle it anytime.

### Putting It All Together
Imagine a website with users sending requests. The load balancer:
1. Listens for HTTP requests on port 80.
2. Splits the traffic across its nodes in enabled AZs (e.g., 50% to AZ A’s node, 50% to AZ B’s).
3. Each node uses round robin to pick a target:
   - If cross-zone is on, it picks from all 10 targets, evenly spreading the load.
   - If cross-zone is off, it picks only from its AZ’s targets, leading to uneven loads if target counts differ.
4. Checks target health and skips any that are down.
5. Keeps running even if an AZ fails, as long as other AZs have healthy targets.

This setup ensures your application stays fast, reliable, and available, adapting to traffic spikes or server failures. The choice of cross-zone load balancing depends on whether you want even distribution across all servers or prefer to keep traffic local to each AZ for lower latency or cost reasons.



