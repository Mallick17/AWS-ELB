# Target Groups

### What Are Target Groups?
Target groups are logical groupings of targets that an Application Load Balancer (ALB) or Network Load Balancer (NLB) can route requests to. They define where traffic should be sent based on listener rules, ensuring efficient distribution and health monitoring. Targets can include EC2 instances, IP addresses, or Lambda functions, registered with the target group for traffic routing.
- Target groups are sets of registered targets, such as EC2 instances, IP addresses, Lambda functions, or other load balancers, to which an ELB routes incoming traffic. They help distribute traffic efficiently, ensuring no single target is overwhelmed, which is crucial for maintaining application availability.

### **Key Features:**
- **Target Registration:** You register targets with a target group, specifying their type (instances, IPs, or Lambda). For example, for ALB, you might register EC2 instances running a web server, while for NLB, you could register IP addresses for high-performance applications.
- **Health Checks:** Configurable health checks determine target health, ensuring only healthy targets receive traffic. Parameters include protocol (HTTP, HTTPS, TCP), port, path (for HTTP/HTTPS), healthy threshold, unhealthy threshold, timeout, and interval. For instance, an HTTP health check might use a path like "/health" with a 200 status code indicating health.
- **Protocol and Port:** Define the protocol (e.g., HTTP, HTTPS for ALB; TCP, UDP for NLB) and port for communication with targets, aligning with listener configurations.
- **Multiple Associations:** A single load balancer can be associated with multiple target groups, each handling different traffic types. For example, one target group for "/api/*" and another for "/web/*" in an ALB setup.
- **Stickiness:** For ALB, you can enable stickiness at the target group level, ensuring requests from the same client are routed to the same target, useful for session persistence.
  
### How Do They Work with Different ELB Types?
- **Application Load Balancer (ALB)**: Ideal for web apps, supports HTTP, HTTPS, and gRPC, with ports 1-65535. It can route to instances, IPs, or Lambda, and includes features like weighted routing (e.g., 80% to one group, 20% to another).
- **Network Load Balancer (NLB)**: Designed for high-performance apps, supports TCP, TLS, UDP, and TCP_UDP, also on ports 1-65535. It handles instance, IP, and ALB targets, with client IP preservation options.
- **Gateway Load Balancer (GWLB)**: Used for network traffic, like firewalls, with GENEVE protocol on port 6081, supporting instance and IP targets, and features like stickiness based on IP and protocol.

### Why Are They Important?
Target groups enable dynamic scaling by integrating with Auto Scaling, support health checks to route only to healthy targets, and can be configured for DNS and routing failover, ensuring reliability during traffic spikes or failures.

**Use Cases:**
- **Microservices Architecture:** Route traffic to different microservices based on URL paths, enhancing modularity. For example, route "/api/users" to one target group and "/api/products" to another.
- **High Availability:** Distribute load across multiple Availability Zones, ensuring failover if one zone fails, with health checks detecting and rerouting from unhealthy targets.
- **Lambda Integration:** For serverless applications, route traffic to Lambda functions, simplifying backend management.
---

## Introduction to Target Groups
Target groups are collections of registered targets, such as Amazon EC2 instances, IP addresses, AWS Lambda functions, or other load balancers, to which an ELB routes incoming traffic based on configured listener rules. They are a fundamental component of ELB, enabling efficient traffic distribution, high availability, and fault tolerance by ensuring traffic is directed to healthy and appropriate resources. Each target group is associated with a specific protocol, port, and health check settings, and can be used with one load balancer at a time.

### Target Groups Across ELB Types
ELB offers three main types—Application Load Balancer (ALB), Network Load Balancer (NLB), and Gateway Load Balancer (GWLB)—each with distinct target group configurations suited to different use cases. Below, we detail each type, including protocols, target types, and key attributes, based on AWS documentation.

## Application Load Balancer (ALB) Target Groups
ALB operates at Layer 7, making it ideal for HTTP/HTTPS traffic and web applications. Target groups for ALB support the following:

- **Protocols and Ports**: HTTP, HTTPS, and gRPC, with ports ranging from 1 to 65535. HTTPS uses security policies like `ELBSecurityPolicy-TLS13-1-0-2021-06` for TLS 1.3, otherwise `ELBSecurityPolicy-2016-08`, with no certificate validation (supports self-signed/expired certificates within VPC).
- **Target Types**: 
  - `instance`: Routes to EC2 instances by instance ID, using the primary private IP.
  - `ip`: Routes to IP addresses from VPC subnets, supporting RFC 1918 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) and RFC 6598 (100.64.0.0/10) blocks, no public IPs. Supports peered VPC instances and on-premises resources via Direct Connect/VPN.
  - `lambda`: Registers a single Lambda function, invoked when the load balancer receives a request; see [Use Lambda functions as targets of an Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/lambda-functions.html).
- **IP Address Type**: Supports IPv4 (default) and IPv6, requiring a `dualstack` load balancer for IPv6. All IPs in a group must match type; IPv6 supports `ip` and `instance` targets.
- **Protocol Version**: Supports HTTP/1.1, HTTP/2, and gRPC, with specific combinations for success or error, as shown in the table below:

| Request Protocol | Target Group Protocol | Result                     |
|------------------|-----------------------|----------------------------|
| HTTP/1.1         | HTTP/1.1             | Success                    |
| HTTP/2           | HTTP/1.1             | Success                    |
| gRPC             | HTTP/1.1             | Error                      |
| HTTP/1.1         | HTTP/2               | Error                      |
| HTTP/2           | HTTP/2               | Success                    |
| gRPC             | HTTP/2               | Success if targets support gRPC |
| HTTP/1.1         | gRPC                 | Error                      |
| HTTP/2           | gRPC                 | Success if POST request    |
| gRPC             | gRPC                 | Success                    |

- **Registered Targets**: Load balancer distributes traffic to healthy targets; can register/deregister dynamically. Deregistration delay defaults to 300 seconds (range 0-3600), entering `draining` state. Integrates with Auto Scaling; see [Attaching a load balancer to your Auto Scaling group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html). Cannot register another ALB's IP in the same VPC; peered VPC instances by IP, not instance ID.
- **Target Group Attributes**: Editable, with defaults and ranges as follows:

| Attribute                                      | Description/Values                                      | Default       |
|------------------------------------------------|---------------------------------------------------------|---------------|
| deregistration_delay.timeout_seconds           | Wait time before deregistering, 0-3600 seconds         | 300 seconds   |
| load_balancing.algorithm.type                  | round_robin, least_outstanding_requests, weighted_random| round_robin   |
| load_balancing.algorithm.anomaly_mitigation    | on/off, for weighted_random only                       | off           |
| load_balancing.cross_zone.enabled              | true, false, use_load_balancer_configuration           | use_load_balancer_configuration |
| slow_start.duration_seconds                    | Linear traffic increase, 30-900 seconds                | 0 (disabled)  |
| stickiness.enabled                             | true/false                                             | false         |
| stickiness.app_cookie.cookie_name              | Name, no AWSALB/AWSALBAPP/AWSALBTG prefixes            | -             |
| stickiness.app_cookie.duration_seconds         | 1-604800 seconds                                       | 86400 seconds |
| stickiness.lb_cookie.duration_seconds          | 1-604800 seconds                                       | 86400 seconds |
| stickiness.type                                | lb_cookie, app_cookie                                  | -             |
| target_group_health.dns_failover.minimum_healthy_targets.count | off, 1-max targets          | 1             |
| target_group_health.dns_failover.minimum_healthy_targets.percentage | off, 1-100              | off           |
| target_group_health.unhealthy_state_routing.minimum_healthy_targets.count | 1-max targets    | 1             |
| target_group_health.unhealthy_state_routing.minimum_healthy_targets.percentage | off, 1-100  | off           |
| lambda.multi_value_headers.enabled             | true/false, for Lambda type                            | false         |

- **Routing Algorithms**: Options include round_robin (even distribution), least_outstanding_requests (routes to lowest in-progress requests, no slow start, handles HTTP/2 to HTTP/1.1, WebSocket support), and weighted_random (random distribution, supports anomaly mitigation, no slow start, no sticky sessions).
- **Health Checks**: Default healthy if ≥1 target healthy; configurable for DNS and routing failover, with thresholds impacting cross-zone behavior. See [CloudWatch metrics for target group health](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html#target-group-health-metric-table).

An unexpected detail is the support for gRPC in ALB target groups, which is less common for interns to encounter but crucial for modern microservices, enabling streaming and custom health checks like `/package.service/method`.

## Network Load Balancer (NLB) Target Groups
NLB operates at Layer 4, designed for high-performance and low-latency applications, such as gaming or financial systems. Target groups for NLB include:

- **Protocols and Ports**: TCP, TLS, UDP, TCP_UDP, with ports 1-65535. TLS establishes connections using target-installed certificates, no validation, secure within VPC.
- **Target Types**: 
  - `instance`: By instance ID.
  - `ip`: By IP address, from VPC subnets or CIDR blocks (10.0.0.0/8, 100.64.0.0/10, 172.16.0.0/12, 192.168.0.0/16), no public IPs. Supports AWS resources (e.g., databases) and on-premises via Direct Connect/VPN.
  - `alb`: Registers a single Application Load Balancer; see [Use Application Load Balancers as targets of a Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/application-load-balancer-target.html).
- **IP Address Type**: Supports IPv4 and IPv6, requiring `dualstack` for IPv6, with IPv4 unable to use UDP with `dualstack`.
- **Supported Listener and Target Group Combinations**:

| Listener Protocol | Target Group Protocol | Target Group Type | Health Check Protocol |
|-------------------|-----------------------|-------------------|-----------------------|
| TCP               | TCP, TCP_UDP         | instance, ip, alb | HTTP, HTTPS, TCP      |
| TCP               | TCP                  | alb               | HTTP, HTTPS           |
| TLS               | TCP, TLS             | instance, ip      | HTTP, HTTPS, TCP      |
| UDP               | UDP, TCP_UDP         | instance, ip      | HTTP, HTTPS, TCP      |
| TCP_UDP           | TCP_UDP              | instance, ip      | HTTP, HTTPS, TCP      |

- **Registered Targets**: Requires at least one target per enabled Availability Zone. Supports dynamic registration/deregistration, with deregistration entering `draining` state. Client IP preservation disabled supports ~55,000 connections/min per NLB IP and unique target; exceeding may cause port allocation errors, mitigated by adding targets. Integrates with Auto Scaling; see [Attaching a load balancer to your Auto Scaling group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html).
- **Target Group Attributes**: Editable for `instance`, `ip`, defaults for `alb`, with details as follows:

| Attribute                                      | Description/Range/Default |
|------------------------------------------------|---------------------------|
| `deregistration_delay.timeout_seconds`         | Wait time before changing state from `draining` to `unused`; 0-3600s, default 300s |
| `deregistration_delay.connection_termination.enabled` | Terminate connections at deregistration timeout; `true`/`false`, default `true` for UDP/TCP_UDP, else `false` |
| `load_balancing.cross_zone.enabled`            | Cross-zone load balancing; `true`, `false`, `use_load_balancer_configuration`, default latter |
| `preserve_client_ip.enabled`                   | Client IP preservation; `true`/`false`, default disabled for IP/TCP/TLS, else enabled (cannot disable for UDP/TCP_UDP) |
| `proxy_protocol_v2.enabled`                    | Proxy protocol v2; `true`/`false`, default `false` |
| `stickiness.enabled`                           | Sticky sessions; `true`/`false`, default `false` |
| `stickiness.type`                              | Stickiness type; only `source_ip` |
| `target_group_health.dns_failover.minimum_healthy_targets.count` | Min healthy targets for DNS failover; `off`, 1-max targets, default 1 |
| `target_group_health.dns_failover.minimum_healthy_targets.percentage` | Min % healthy targets for DNS failover; `off`, 1-100, default `off` |
| `target_group_health.unhealthy_state_routing.minimum_healthy_targets.count` | Min healthy targets for routing failover; 1-max targets, default 1 |
| `target_group_health.unhealthy_state_routing.minimum_healthy_targets.percentage` | Min % healthy targets for routing failover; `off`, 1-100, default `off` |
| `target_health_state.unhealthy.connection_termination.enabled` | Terminate connections to unhealthy targets; `true`/`false`, default `true` |
| `target_health_state.unhealthy.draining_interval_seconds` | Wait time before changing unhealthy target state; 0-360000s, default 0s (only if connection termination disabled) |

- **Health Checks**: Default healthy if ≥1 target healthy; configurable for DNS and routing failover, with thresholds impacting cross-zone behavior. DNS failover marks zone unhealthy if below threshold, TTL 60 seconds; routing failover sends to all targets if below threshold. See [Health checks for Network Load Balancer target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html).

An unexpected detail for NLB is the support for ~55,000 connections per minute per NLB IP and unique target, which is a specific performance metric interns might not anticipate but is critical for high-traffic scenarios.

## Gateway Load Balancer (GWLB) Target Groups
GWLB operates at Layer 3, designed for network traffic management, such as with firewall appliances. Target groups for GWLB include:

- **Protocols and Ports**: GENEVE protocol, port 6081.
- **Target Types**: 
  - `instance`: By instance ID.
  - `ip`: By IP address, from VPC subnets or CIDR blocks (10.0.0.0/8, 100.64.0.0/10, 172.16.0.0/12, 192.168.0.0/16), no public IPs.
- **Registered Targets**: Requires at least one target per enabled Availability Zone. Can register with multiple groups, add for increased demand, deregister for decreased demand or maintenance. Deregistered targets enter `draining` state until in-flight requests complete.
- **Target Group Attributes**: Editable, with details as follows:

| Attribute                                      | Description/Range/Default |
|------------------------------------------------|---------------------------|
| `deregistration_delay.timeout_seconds`         | Wait time before deregistering, 0-3600 seconds, default 300 seconds |
| `stickiness.enabled`                           | Sticky sessions; `true`/`false`, default `false` (uses 5_tuple when false) |
| `stickiness.type`                              | Stickiness type; `source_ip_dest_ip`, `source_ip_dest_ip_proto` |
| `target_failover.on_deregistration`            | Failover behavior; `rebalance` or `no_rebalance`, default `no_rebalance` (must match `target_failover.on_unhealthy`) |
| `target_failover.on_unhealthy`                 | Failover behavior; `rebalance` or `no_rebalance`, default `no_rebalance` (must match `target_failover.on_deregistration`) |

- **Health Checks**: Defined per target group, uses default settings unless overridden, monitors target health in enabled Availability Zones. See [Health checks for Gateway Load Balancer target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/health-checks.html).

An unexpected detail for GWLB is the use of GENEVE protocol, which is less common and specific to network traffic, potentially new for interns but important for network security setups.

### Practical Tips
For interviews, be ready to explain how target groups improve scalability (e.g., integrating with Auto Scaling), compare features across ALB, NLB, and GWLB (e.g., ALB for HTTP/2, NLB for UDP, GWLB for firewalls), and discuss configuration options like health checks and sticky sessions. Practice scenarios, such as setting up an ALB target group for a web app with weighted routing or an NLB target group for a gaming platform with client IP preservation.

#### Key Considerations
Target groups are critical for ensuring traffic is routed to healthy resources, supporting dynamic scaling, and enhancing reliability. However, costs can increase with high traffic, especially with multiple target groups, so monitoring with CloudWatch is recommended. Migration from legacy systems using Classic Load Balancer to modern target groups may be necessary, as detailed at [Migrate your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/migrate-classic-load-balancer.html).

