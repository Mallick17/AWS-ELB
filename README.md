# AWS Elastic Load Balancer
Elastic Load Balancing (ELB) is a managed service provided by Amazon Web Services (AWS), designed to automatically distribute incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, and IP addresses, in one or more Availability Zones. This distribution enhances the fault tolerance and availability of applications by ensuring no single server is overwhelmed, which is critical for maintaining performance during traffic spikes. ELB monitors the health of registered targets and routes traffic only to healthy ones, automatically scaling the load balancer capacity in response to changes in incoming traffic.
- The note addresses three key areas: target groups, types of load balancers and their uses, and a detailed exploration of console options for creating a load balancer, drawing from official AWS documentation and comparative analyses.
  
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

<details>
   <summary>Quick Guide of Load Balancers & Their Use Cases</summary>

#### Types of Load Balancers and Their Uses: Comprehensive Overview

**What is Load Balancing?**
Load balancing is the process of distributing network traffic across multiple resources to ensure no single server is overwhelmed, improving responsiveness and availability. AWS ELB facilitates this by acting as a traffic manager, routing requests to registered targets based on health and configuration.

**Types of Load Balancers:**
AWS offers four types, each operating at different layers of the OSI model and serving distinct use cases:

1. **Application Load Balancer (ALB):**
   - **Layer:** 7 (Application Layer, HTTP/HTTPS)
   - **Features:** Advanced request routing based on content (path, host, HTTP headers), WebSocket and HTTP/2 support, integration with AWS WAF, and Lambda targets.
   - **Use Cases:** Web applications, microservices, containerized applications, and scenarios requiring content-based routing. For example, route "/api/*" to one set of instances and "/web/*" to another.
   - **Performance:** Handles millions of requests, with low latency for HTTP/HTTPS traffic, suitable for modern, scalable architectures.

2. **Network Load Balancer (NLB):**
   - **Layer:** 4 (Transport Layer, TCP/UDP)
   - **Features:** High performance, low latency, static IP addresses per Availability Zone, preservation of source IP, and support for TCP, UDP, TLS traffic.
   - **Use Cases:** High-throughput, low-latency applications like gaming, streaming, IoT, and financial trading platforms. For instance, distribute TCP traffic for a gaming server across multiple instances.
   - **Performance:** Handles millions of requests per second, with sub-millisecond latency, ideal for extreme performance needs.

3. **Gateway Load Balancer (GLB):**
   - **Layer:** 3 (Network Layer, GENEVE protocol)
   - **Features:** Transparent traffic inspection, scaling and management of third-party virtual appliances like firewalls, intrusion detection systems, and deep packet inspection tools.
   - **Use Cases:** Network security and monitoring, deploying virtual appliances in a scalable manner. For example, route traffic through a firewall appliance before reaching application servers.
   - **Performance:** Designed for high-volume network traffic, with integration for appliance management.

4. **Classic Load Balancer (CLB):**
   - **Layer:** Both 4 and 7 (Legacy, supports HTTP/HTTPS, TCP/SSL)
   - **Features:** Basic load balancing, direct instance registration, health checks, and stickiness.
   - **Use Cases:** Legacy applications built within EC2-Classic or simple load balancing needs. For example, older web applications not requiring advanced routing.
   - **Performance:** Less feature-rich than ALB and NLB, with AWS recommending migration for new applications ([Migrate your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-migration.html)).
   
</details>

- This table highlights the diversity, with ALB for application-level routing and NLB for network-level performance, GLB for appliances, and CLB for legacy needs.

## Console Options for Creating a Load Balancer: Detailed Exploration

### **Overview:**
When creating a load balancer in the AWS Management Console, the options vary by type (ALB, NLB, GLB, CLB), but generally include naming, network configuration, security, listeners, target groups, and advanced settings. The following details are based on the creation process for ALB, with notes for other types.

**Steps and Options:**

1. **Select Load Balancer Type:**
   - Choose between Application Load Balancer, Network Load Balancer, Gateway Load Balancer, or Classic Load Balancer. Each type has specific use cases, with ALB for HTTP/HTTPS, NLB for TCP/UDP, GLB for appliances, and CLB for legacy.

2. **Basic Configuration:**
   - **Name:** Enter a unique name, up to 32 characters, alphanumeric and hyphens, cannot start/end with hyphen or contain "internal-".
   - **Scheme:** Internet-facing (routes from internet, requires public subnets) or Internal (routes within VPC, private subnets).
   - **IP Address Type:** IPv4, Dualstack (IPv4 and IPv6), or Dualstack without public IPv4 for ALB/NLB. GLB is internal only, and CLB has similar options.

3. **Network Mapping:**
   - **VPC:** Select the Virtual Private Cloud. For Internet-facing, ensure an internet gateway is attached.
   - **Availability Zones:** Choose at least two Availability Zones, selecting subnets (public for Internet-facing, private for internal). Local Zones or Outpost subnets can be selected for ALB/NLB.
   - **Cross-Zone Load Balancing:** Enable to distribute traffic evenly across all targets, regardless of Availability Zone, available for ALB/NLB.

4. **Security Groups:**
   - Assign security groups to control inbound and outbound traffic. Must allow traffic on listener ports and health check ports. Create new or select existing, with rules for protocols like TCP/80, TCP/443 for HTTPS.

5. **Listeners:**
   - Configure listeners to check for connection requests. For ALB:
     - Default HTTP on port 80, select target group.
     - Optional HTTPS listener: Choose security policy (e.g., latest TLS), SSL/TLS certificate from ACM/IAM, enable mutual authentication if needed.
   - For NLB: TCP, UDP, TCP_UDP, or TLS, specify port, select target group.
   - For GLB: Typically GENEVE on port 6081.
   - For CLB: HTTP, HTTPS, TCP, or SSL, specify load balancer and instance ports.

6. **Target Groups:**
   - Create or select existing target groups. Options include:
     - **Target Type:** Instances, IP addresses, or Lambda for ALB/NLB; IP or instances for GLB; instances for CLB.
     - **Name:** Unique identifier.
     - **Protocol and Port:** E.g., HTTP/80 for ALB, TCP/80 for NLB.
     - **VPC:** For instance or IP targets.
     - **Health Checks:** Protocol, port, path (for HTTP/HTTPS), healthy threshold (consecutive successful checks), unhealthy threshold, timeout, interval, success codes.
   - Register targets: Select instances, enter IPs, or choose Lambda functions, specify ports.

7. **Advanced Settings:**
   - **Deregistration Delay:** Time to wait before deregistering targets, allowing existing connections to complete (connection draining).
   - **Slow Start Duration:** Gradually increase traffic to new targets, reducing load impact.
   - **Stickiness:** Enable for ALB/NLB, route requests from same client to same target, useful for session persistence.
   - **Access Logs:** Enable to capture detailed request information, stored in S3, for analysis and auditing.
   - **AWS WAF Integration:** For ALB, associate Web Application Firewall for protection against exploits.
   - **AWS Global Accelerator:** Improve performance by routing traffic through AWS global network, available for ALB/NLB.

8. **Tags:**
   - Add key-value pairs for organization, cost allocation, and management, with rules: max 32 characters, alphanumeric and special characters (+ - = . _ : / @).

**Type-Specific Notes:**
- **ALB:** Supports routing rules (path, host, HTTP headers), WebSocket, HTTP/2, ideal for modern applications.
- **NLB:** Static IPs per Availability Zone, source IP preservation, high throughput, less routing flexibility.
- **GLB:** Focused on virtual appliances, GENEVE protocol, internal scheme only.
- **CLB:** Legacy, direct instance registration, basic health checks, recommended for migration to ALB/NLB.

**Practical Example for ALB Creation:**
- Open EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
- Choose Create Load Balancer, select Application Load Balancer.
- Name: "WebAppLB", Scheme: Internet-facing, IP type: IPv4.
- VPC: Select default, Availability Zones: us-west-2a and us-west-2b, subnets accordingly.
- Security groups: Allow HTTP/80, HTTPS/443.
- Listeners: HTTP:80, select target group "WebServers".
- Target group: Create new, Target type: Instances, Protocol: HTTP, Port: 80, Health check: HTTP/80, Path "/health", Healthy threshold 3, etc.
- Review, Create.

This ensures a load balancer is set up to distribute web traffic, with health checks and security configured.

**Best Practices:**
- Use at least two Availability Zones for high availability.
- Enable cross-zone load balancing for even distribution.
- Configure health checks to detect and route around unhealthy targets.
- Enable access logs for monitoring and troubleshooting.
- For HTTPS, use ACM for certificate management, ensuring latest security policies.



