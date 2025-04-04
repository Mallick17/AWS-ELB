# **Application Load Balancer (ALB)**
An Application Load Balancer (ALB) is a Layer 7 (application layer) load balancer provided by AWS Elastic Load Balancing (ELB) service. It distributes incoming HTTP/HTTPS traffic (or other application-layer protocols like WebSocket) across multiple targets, such as EC2 instances, containers, or Lambda functions, based on advanced routing rules. ALBs are designed for modern applications with microservices architectures, offering features like content-based routing, WebSocket support, and integration with other AWS services.

### **Why Use an ALB?**
1. **High Availability**: Distributes traffic across multiple targets in different Availability Zones (AZs) to ensure your application remains available even if one AZ fails.
2. **Scalability**: Automatically scales to handle varying levels of traffic, ensuring performance during traffic spikes.
3. **Advanced Routing**: Supports path-based, host-based, and other routing rules to direct traffic to specific target groups based on the request (e.g., `/api` to one group, `/web` to another).
4. **Security**: Integrates with AWS services like Web Application Firewall (WAF) for protection and supports SSL termination/encryption.
5. **Monitoring and Observability**: Provides metrics and logs for monitoring application health and performance.

### **How Does an ALB Work?**
1. **Listeners**: ALBs use listeners to check for incoming connection requests. A listener is configured with a protocol (e.g., HTTP, HTTPS) and a port (e.g., 80, 443). In the screenshot, the ALB has a listener on HTTP:80.
2. **Rules**: Each listener has rules that define how to route traffic. Rules can be based on the URL path, hostname, HTTP headers, etc. In the screenshot, the listener has a default rule to forward traffic to the `mallow-php-app` target group.
3. **Target Groups**: These are logical groupings of targets (e.g., EC2 instances) that handle the traffic. Each target group can have its own health checks to ensure only healthy targets receive traffic. In the screenshot, the target group is `mallow-php-app`, but it currently has 0 targets registered.
4. **Targets**: The actual resources (e.g., EC2 instances, Lambda functions) that process the requests. Targets are registered with a target group.
5. **Health Checks**: ALBs periodically check the health of targets in a target group. If a target fails the health check, the ALB stops sending traffic to it until it becomes healthy again.

---

## **Components of the ALB**

### **1. Details Tab**
- **Load Balancer Type**: Application (Layer 7, HTTP/HTTPS traffic).
- **Scheme**: Internet-facing (accessible from the public internet).
- **Status**: Active (the ALB is operational).
- **Hosted Zone**: ZP97RAFLXTNZK (the Route 53 hosted zone for DNS).
- **VPC**: vpc-088e7a7bf4b68741b (the Virtual Private Cloud where the ALB resides).
- **Availability Zones and Subnets**:
  - `ap-south-1a (aps1-az1)`: subnet-070aec83e8b9d487b
  - `ap-south-1b (aps1-az3)`: subnet-0c4bf7a2535498ca02
  These subnets are in different AZs for high availability.
- **Load Balancer ARN**: A unique identifier for the ALB.
- **DNS Name**: mallow-vpc-lb-652465165.ap-south-1.elb.amazonaws.com (the public DNS name for accessing the ALB).
- **Load Balancer IP Address Type**: IPv4 (supports IPv4 traffic; IPv6 is disabled).
- **Date Created**: April 2, 2025, 11:12 UTC.

### **2. Listeners and Rules Tab**
A listener checks for connection requests on its configured protocol and port. Traffic received by the listener is routed according to the default action and any additional rules.
- **Listener**: HTTP:80 (listens for HTTP traffic on port 80).
- **Rules**: 1 rule, which forwards traffic to the `mallow-php-app` target group with 100% weight (all traffic goes to this group).
- **Security Policy**: Not applicable (since this is HTTP, not HTTPS; HTTPS would require an SSL/TLS certificate and a security policy).
- **Default SSL/TLS Certificate**: Not applicable (no HTTPS listener).
- **TLS**: Not applicable (no HTTPS listener).
- **Test Store**: Not applicable (used for mutual TLS, not configured here).

### **3. Network Mapping Tab**
Targets in the listed zones and subnets are available for traffic from the load balancer using the IP addresses shown.
- **VPC**: vpc-088e7a7bf4b68741b (same as in Details).
- **Load Balancer IP Address Type**: IPv4.
- **IP Pools**: None (not using specific IP address pools).
- **Availability Zones and Subnets**:

Including two or more Availability Zones, and corresponding subnets, increases the fault tolerance of your applications.
  - `ap-south-1a`: subnet-070aec83e8b9d487b (CIDR: 10.0.1.0/24)
  - `ap-south-1b`: subnet-0c4bf7a2535498ca02 (CIDR: 10.0.2.0/24)
  - IPv6 Address: Not applicable (IPv6 is disabled).

### **4. Resource Map Tab**
- **Overview**: Shows the ALB (`mallow-vpc-lb`), its listener (HTTP:80), rule (forward to target group), and target group (`mallow-php-app`).
- **Target Group**: `mallow-php-app` (0 targets registered, 0 healthy, 0 unhealthy, 0 draining, 0 unused).

### **5. Security Tab**
- **Security Groups**:
  - `sg-0f4890a56207b1d1` (default): Default VPC security group.
  - `sg-0df4758e208f0b8e1` (php-lb-sg): Security group for the load balancer.
- **Security Groups Role**: Security Groups control the traffic to the ALB (inbound rules) and from the ALB to the targets (outbound rules). As discussed earlier, Security Groups are stateful, meaning they automatically allow return traffic for allowed connections.

### **6. Monitoring Tab**
- **Metrics**:
  - **Target Response Time**: No data (no targets registered).
  - **Requests**: No data.
  - **Rule Evaluations**: No data.
  - **Target 5XXs, 4XXs, ELB 5XXs, 4XXs**: No data (no traffic or targets).
  - **Target Connection Errors**: No data.
  - **Sum Rejected Connection Count**: No data.
  - **Target TLS Negotiation Errors**: No data.
  - **Client TLS Negotiation Errors**: No data.
  - **Target 3XXs, 2XXs**: No data.
  - **Active Connections**: 4 active connections (some traffic is hitting the ALB, but no targets are processing it).

### **7. Integrations Tab**
- **Amazon Application Recovery Controller (ARC)**: Not integrated (optimizes availability).
- **Amazon CloudFront + AWS WAF**: Not integrated (optimizes performance, availability, and security).
- **AWS Global Accelerator**: Not integrated (optimizes performance and availability).
- **AWS Config**: Not integrated (optimizes monitoring and compliance).
- **AWS WAF**: Not integrated (optimizes security).

### **8. Attributes Tab**
- **TLS Version and Cipher Headers**: Off (not applicable for HTTP).
- **WAF Fail Open**: Off (not applicable since WAF is not integrated).
- **HTTP/2**: On (supports HTTP/2 for better performance).
- **Connection Idle Timeout**: 60 seconds (closes idle connections after 60 seconds).
- **HTTP Client Keepalive Duration**: 3600 seconds (keeps connections alive for 1 hour).
- **Packet Handling**:
  - **Desync Mitigation Mode**: Defensive (protects against HTTP desync attacks).
  - **Drop Invalid Header Fields**: Off (does not drop requests with invalid headers).
  - **X-Forwarded-For Header**: Append (adds the client’s IP to the X-Forwarded-For header).
  - **Client Port Preservation**: Off (does not preserve the client’s source port).
- **Availability Zone Routing Configuration**:
  - **Cross-Zone Load Balancing**: On (distributes traffic evenly across all AZs).
  - **ARC Zonal Shift Integration**: Disabled (not using ARC for zonal shifts).
- **Protection**:
  - **Deletion Protection**: Off (ALB can be deleted without restriction).
- **Monitoring**:
  - **Access Logs**: Off (not logging requests to S3).
  - **Connection Logs**: Off (not logging connection details).

### **9. Capacity Tab**
- **Load Balancer Capacity Unit (LCU) Reservation**: 2 LCUs reserved until 00:00 UTC (ensures capacity during traffic spikes).
- **LCU Usage**: Graph shows peak LCU usage (red) vs. reserved LCU (green). Currently, usage is low.

### **10. Tags Tab**
- No tags are associated with the ALB.

---

### **How the ALB is Currently Used**
- The ALB (`mallow-vpc-lb`) is set up to handle HTTP traffic on port 80 and forward it to the `mallow-php-app` target group.
- It’s deployed in two Availability Zones (`ap-south-1a` and `ap-south-1b`) for high availability.
- Cross-zone load balancing is enabled, ensuring traffic is distributed evenly across AZs.
- Security Groups are applied to control traffic, but no targets are registered in the target group, so the ALB isn’t processing any traffic effectively (despite 4 active connections).
- No advanced integrations (e.g., WAF, CloudFront) or monitoring (e.g., access logs) are enabled.

---

## **What Happens if All Integrations and Configurations Are Enabled?**

### **Integrations**
1. **Amazon Application Recovery Controller (ARC)**:
   - **What it does**: ARC helps manage and recover applications during outages by enabling zonal shifts (rerouting traffic away from a failing AZ).
   - **If enabled**: The ALB would integrate with ARC, allowing automated zonal shifts if one AZ (`ap-south-1a` or `ap-south-1b`) fails. Traffic would be rerouted to the healthy AZ, improving availability.
   - **Impact**: Better resilience during AZ outages, but requires ARC setup and configuration.

2. **Amazon CloudFront + AWS WAF**:
   - **What it does**: CloudFront is a CDN that caches content globally, reducing latency. AWS WAF (Web Application Firewall) protects against common web attacks (e.g., SQL injection, XSS).
   - **If enabled**:
     - **CloudFront**: The ALB would sit behind CloudFront, which would cache static content and reduce latency for global users. Requests would hit CloudFront first, then the ALB for dynamic content.
     - **WAF**: WAF rules would filter malicious traffic before it reaches the ALB, blocking attacks like SQL injection or DDoS.
   - **Impact**: Improved performance (via CloudFront) and security (via WAF). WAF fail-open (currently off) could be enabled to allow traffic if WAF fails, but this might reduce security.

3. **AWS Global Accelerator**:
   - **What it does**: Global Accelerator routes traffic over the AWS global network, improving performance and availability by directing users to the nearest healthy endpoint.
   - **If enabled**: The ALB would be accessible via Global Accelerator, which would route traffic to the closest AZ (`ap-south-1a` or `ap-south-1b`) using anycast IP addresses. It would also provide failover if one AZ fails.
   - **Impact**: Faster and more reliable access for global users, with automatic failover.

4. **AWS Config**:
   - **What it does**: AWS Config tracks configuration changes and compliance for AWS resources.
   - **If enabled**: AWS Config would monitor the ALB’s configuration (e.g., listener rules, security groups) and alert on non-compliant changes (e.g., if deletion protection is disabled).
   - **Impact**: Better governance and auditing, but requires setting up Config rules.

5. **AWS WAF (Standalone)**:
   - **What it does**: Protects the ALB from web attacks (already covered with CloudFront + WAF, but can be standalone).
   - **If enabled**: WAF would filter traffic directly at the ALB level, blocking malicious requests.
   - **Impact**: Enhanced security, but requires WAF rule configuration.

### **Configurations**
1. **TLS Version and Cipher Headers**: Currently off (not applicable for HTTP).
   - **If enabled**: Requires an HTTPS listener (port 443) with an SSL/TLS certificate. This would enforce specific TLS versions (e.g., TLS 1.2, 1.3) and ciphers for secure connections.
   - **Impact**: Secures traffic with encryption, but requires a certificate (e.g., via AWS Certificate Manager) and an HTTPS listener.

2. **WAF Fail Open**: Currently off.
   - **If enabled**: If WAF fails to process a request, traffic would still reach the ALB (fail-open behavior).
   - **Impact**: Ensures availability during WAF failures, but might allow malicious traffic through.

3. **HTTP/2**: Already on.
   - **Impact**: Improves performance by allowing multiplexing (multiple requests over a single connection).

4. **Connection Idle Timeout**: 60 seconds.
   - **If adjusted**: Increasing this (e.g., to 300 seconds) would keep idle connections open longer, useful for applications with long-lived connections (e.g., WebSockets).
   - **Impact**: More resource usage on the ALB, but better for certain workloads.

5. **HTTP Client Keepalive Duration**: 3600 seconds.
   - **If adjusted**: Reducing this (e.g., to 300 seconds) would close keepalive connections sooner, freeing resources.
   - **Impact**: Balances resource usage vs. performance.

6. **Packet Handling**:
   - **Desync Mitigation Mode**: Already defensive (protects against HTTP desync attacks).
   - **Drop Invalid Header Fields**: Currently off.
     - **If enabled**: Requests with invalid headers would be dropped, improving security.
     - **Impact**: Might break some legitimate traffic if clients send malformed headers.
   - **X-Forwarded-For Header**: Already append (preserves client IP).
   - **Client Port Preservation**: Currently off.
     - **If enabled**: Preserves the client’s source port, useful for certain applications.
     - **Impact**: More resource usage on the ALB.

7. **Availability Zone Routing**:
   - **Cross-Zone Load Balancing**: Already on (distributes traffic across AZs).
   - **ARC Zonal Shift Integration**: Currently disabled.
     - **If enabled**: Works with ARC to shift traffic during AZ failures.
     - **Impact**: Improves availability.

8. **Protection**:
   - **Deletion Protection**: Currently off.
     - **If enabled**: Prevents accidental deletion of the ALB.
     - **Impact**: Adds safety, but might hinder automation scripts that delete resources.

9. **Monitoring**:
   - **Access Logs**: Currently off.
     - **If enabled**: Logs all requests to an S3 bucket, useful for auditing and debugging.
     - **Impact**: Increases storage costs and requires S3 bucket setup.
   - **Connection Logs**: Currently off.
     - **If enabled**: Logs connection details (e.g., source IP, port), useful for troubleshooting.
     - **Impact**: More detailed monitoring, but increases logging overhead.

---

### **Summary of Impact if All Integrations and Configurations Are Enabled**
- **Performance**: CloudFront and Global Accelerator would reduce latency and improve global access. HTTP/2 (already on) ensures efficient connections.
- **Availability**: ARC and Global Accelerator would improve failover and recovery during AZ outages. Cross-zone load balancing (already on) ensures even traffic distribution.
- **Security**: WAF (with CloudFront or standalone) would protect against attacks. Enabling TLS (with HTTPS) would encrypt traffic. Deletion protection would prevent accidental deletion.
- **Monitoring**: Access logs, connection logs, and AWS Config would provide detailed auditing and compliance tracking.
- **Trade-offs**: Increased complexity (more services to manage), higher costs (e.g., WAF, CloudFront, S3 logging), and potential for misconfiguration (e.g., overly strict WAF rules blocking legitimate traffic).

---

### **Current Issues and Recommendations**
- **No Targets Registered**: The `mallow-php-app` target group has 0 targets, so the ALB isn’t forwarding traffic to any instances. Register EC2 instances or other targets to make the ALB functional.
- **HTTP Only**: The ALB uses HTTP:80, which is insecure. Add an HTTPS:443 listener with an SSL/TLS certificate for encryption.
- **No WAF**: Enable WAF to protect against web attacks, especially since this is an internet-facing ALB.
- **No Access Logs**: Enable access logs for auditing and troubleshooting.
- **Integrations**: Consider enabling CloudFront + WAF and Global Accelerator for better performance and security.

Let me know if you’d like to dive deeper into any specific component or configuration!
