# **Application Load Balancer (ALB)**
An Application Load Balancer (ALB) is a Layer 7 (application layer) load balancer provided by AWS Elastic Load Balancing (ELB) service. It distributes incoming HTTP/HTTPS traffic (or other Layer 7 protocols like WebSocket) across multiple targets, such as EC2 instances, containers, or Lambda functions, based on advanced routing rules. Unlike a Network Load Balancer (Layer 4) or Classic Load Balancer, an ALB is designed to handle HTTP/HTTPS traffic with features like content-based routing, path-based routing, and support for modern application architectures (e.g., microservices).

### **Why Use an ALB?**
1. **Scalability**: ALBs automatically scale to handle varying levels of traffic, ensuring your application remains responsive during traffic spikes.
2. **High Availability**: By distributing traffic across multiple Availability Zones (AZs), ALBs improve fault tolerance. If one AZ fails, traffic is routed to healthy targets in other AZs.
3. **Advanced Routing**: ALBs support routing based on URL paths, hostnames, HTTP headers, or query strings, making them ideal for microservices or multi-tenant applications.
4. **Security**: ALBs integrate with AWS Web Application Firewall (WAF) for protection against common web exploits and support SSL/TLS termination for secure communication.
5. **Monitoring and Observability**: ALBs provide detailed metrics and logs (e.g., via CloudWatch) to monitor application health and performance.

### **How Does an ALB Work?**
1. **Traffic Entry**: Clients (e.g., users’ browsers) send requests to the ALB via its DNS name (e.g., `mallow-vpc-lb-652465165.ap-south-1.elb.amazonaws.com`).
2. **Listener**: The ALB uses listeners to receive incoming traffic on specific ports and protocols (e.g., HTTP on port 80, as seen in the "Listeners and rules" tab).
3. **Rules**: Each listener has rules that determine how to route traffic. For example, a rule might forward traffic to a specific target group based on the URL path (e.g., `/api` to one target group, `/web` to another).
4. **Target Groups**: These are collections of targets (e.g., EC2 instances) that handle the traffic. The ALB forwards requests to healthy targets within the group based on the routing rules.
5. **Health Checks**: The ALB continuously monitors the health of targets using health checks. Unhealthy targets are excluded from receiving traffic until they recover.
6. **Response**: After the target processes the request, the response is sent back through the ALB to the client.

---

### **Components of the ALB**
#### **1. Details Tab**
This tab provides an overview of the ALB’s configuration:
- **Load Balancer Type**: Application (indicating it’s an ALB, operating at Layer 7).
- **Scheme**: Internet-facing (the ALB is accessible from the public internet).
- **Hosted Zone**: ZP97RAFLXTNZK (the Route 53 hosted zone for DNS).
- **VPC**: `vpc-088e7a7bf4b68741b` (the Virtual Private Cloud where the ALB resides).
- **Availability Zones**:
  - `subnet-070aec83e8b9d487b` in `ap-south-1a` (aps1-az1).
  - `subnet-0c4bf7a2535498ca02` in `ap-south-1b` (aps1-az3).
  - This ensures the ALB can distribute traffic across two AZs for high availability.
- **Load Balancer IP Address Type**: IPv4 (the ALB uses IPv4 addresses; IPv6 is not enabled).
- **Date Created**: April 2, 2025, 11:12 UTC.
- **Load Balancer ARN**: A unique identifier for the ALB (`arn:aws:elasticloadbalancing:ap-south-1:3539713104321:loadbalancer/app/mallow-vpc-lb/a223c390bd3ed397`).
- **DNS Name**: `mallow-vpc-lb-652465165.ap-south-1.elb.amazonaws.com` (the public DNS name clients use to access the ALB).

**Purpose**: This tab gives a high-level summary of the ALB’s configuration, including its network placement and accessibility.

---

#### **2. Listeners and Rules Tab**
- **Listener**: HTTP:80 (the ALB listens for HTTP traffic on port 80).
- **Default Action**: Forward to target group `mallow-php-app` (100% of traffic is sent to this target group by default).
- **Rules**: 1 rule (the default rule forwards traffic to the target group).
- **Security Policy, SSL/TLS Certificate, TLS**: Not applicable (since this listener is HTTP, not HTTPS; HTTPS would require a certificate and security policy).

**Purpose**: Listeners define how the ALB accepts incoming traffic, and rules determine how that traffic is routed. Here, all HTTP traffic on port 80 is forwarded to the `mallow-php-app` target group.

---

#### **3. Network Mapping Tab**
- **VPC**: `vpc-088e7a7bf4b68741b` (same as in the Details tab).
- **Load Balancer IP Address Type**: IPv4 (consistent with the Details tab).
- **Availability Zones and Subnets**:
  - `ap-south-1a` (aps1-az1): `subnet-070aec83e8b9d487b` with private IPv4 CIDR 10.0.1.0/24.
  - `ap-south-1b` (aps1-az3): `subnet-0c4bf7a2535498ca02` with private IPv4 CIDR 10.0.2.0/24.
- **IPv6 Address**: Not applicable (IPv6 is not enabled).

**Purpose**: This tab shows the network configuration, including the subnets where the ALB operates. The ALB needs at least two subnets in different AZs for high availability. The private IPv4 addresses indicate the ALB’s internal network placement within the VPC.

---

#### **4. Resource Map Tab**
- **Listeners**: HTTP:80.
- **Rules**: 1 rule (forward to target group).
- **Target Groups**: `mallow-php-app` (the target group receiving traffic).
- **Targets**: 0 targets (no instances are currently registered with the target group).

**Purpose**: This tab provides a visual representation of the ALB’s traffic flow: from the listener to the target group. The lack of targets suggests the ALB is either not fully configured or the instances are not registered yet.

---

#### **5. Security Tab**
- **Security Groups**:
  - `sg-0f4890a5c627b1d71` (default VPC security group).
  - `sg-0c4bf7a253e0f8be01` (named `php-lb-sg`, a custom security group for the load balancer).

**Purpose**: Security Groups (SGs) control the traffic allowed to and from the ALB. As discussed in your previous question, SGs are stateful, meaning they automatically allow return traffic for allowed inbound connections. For example, if `php-lb-sg` allows inbound HTTP on port 80, the response traffic is automatically permitted. You’d need to check the rules of these SGs to see the specific ports and sources allowed (e.g., 0.0.0.0/0 for public access on port 80).

**Note**: The ALB is in a VPC, and NACLs (stateless) would also apply at the subnet level (e.g., for `subnet-070aec83e8b9d487b`). However, NACLs are not shown in the ALB console—they’re managed separately at the VPC level. NACLs would require explicit inbound and outbound rules for traffic to flow, as they don’t track connection state.

---

#### **6. Monitoring Tab**
This tab shows CloudWatch metrics for the ALB:
- **Target Response Time**: No data (indicating no traffic or targets).
- **Requests**: No data.
- **Rule Evaluations**: No data.
- **Target 5XXs, 4XXs, ELB 5XXs, 4XXs**: No data (no errors recorded).
- **ELB 503s, 502s, 504s**: No data (no gateway errors).
- **Target Connection Errors**: No data.
- **Sum Rejected Connection Count**: No data.
- **Target TLS Negotiation Errors**: No data.
- **Client TLS Negotiation Errors**: No data.
- **Target 3XXs, 2XXs**: No data.
- **Active Connection Count**: 4 connections (some minimal activity, possibly health checks).

**Purpose**: This tab helps monitor the ALB’s performance and health. The lack of data suggests the ALB isn’t receiving significant traffic, likely because no targets are registered. The 4 active connections might be from health checks or test traffic.

---

#### **7. Integrations Tab**
- **Amazon Application Recovery Controller (ARC)**: Not integrated.
- **Amazon CloudFront + AWS WAF**: Not integrated.
- **AWS Global Accelerator**: Not integrated.
- **AWS Config**: Not integrated.
- **AWS WAF**: Not integrated.

**Purpose**: This tab shows potential integrations with other AWS services to enhance the ALB’s functionality (e.g., WAF for security, Global Accelerator for performance). None are enabled here, so the ALB operates without these additional features.

---

#### **8. Attributes Tab**
- **TLS Version and Cipher Headers**: Off.
- **WAF Fail Open**: Off (if WAF were enabled, this would determine behavior on WAF failure).
- **HTTP/2**: On (the ALB supports HTTP/2 for better performance).
- **Connection Idle Timeout**: 60 seconds (if no data is sent for 60 seconds, the connection is closed).
- **HTTP Client Keepalive Duration**: 3600 seconds (1 hour; keeps connections alive for reuse).
- **Packet Handling**:
  - **Desync Mitigation Mode**: Defensive (protects against HTTP desync attacks).
  - **Drop Invalid Header Fields**: Off (invalid headers are not dropped).
  - **X-Forwarded-For Header**: Append (the ALB appends the client’s IP to the X-Forwarded-For header).
  - **Client Port Preservation**: Off (the client’s source port is not preserved).
- **Availability Zone Routing Configuration**:
  - **Cross-Zone Load Balancing**: On (traffic is distributed across all AZs, not just within the same AZ as the request).
  - **ARC Zonal Shift Integration**: Disabled.
- **Deletion Protection**: Off (the ALB can be deleted without extra confirmation).
- **Access Logs**: Off (access logs are not being stored in S3).
- **Connection Logs**: Off (detailed connection logs are not enabled).

**Purpose**: This tab configures the ALB’s behavior, such as how it handles connections, security features, and logging. For example, enabling HTTP/2 improves performance, while the connection idle timeout ensures resources aren’t wasted on inactive connections.

---

#### **9. Capacity Tab**
- **Load Balancer Capacity Unit (LCU) Reservation**: 2 LCUs reserved until 00:00 UTC (decreases remaining today).
- **Load Balancer Capacity Units (LCU) Peak vs. Reserved**: A graph showing peak LCU usage (red) vs. reserved LCUs (green). Currently, no significant usage is shown.

**Purpose**: LCUs measure the ALB’s resource consumption (e.g., connections, requests, bandwidth). Reserving LCUs ensures capacity during traffic spikes. The lack of usage aligns with the absence of targets and traffic.

---

#### **10. Tags Tab**
- **Tags**: 0 (no tags are associated with the ALB).

**Purpose**: Tags are key-value pairs used for resource management (e.g., cost allocation, organization). None are set here.

---

### **How the ALB is Used in This Setup**
- **Current Configuration**:
  - The ALB (`mallow-vpc-lb`) is internet-facing and listens for HTTP traffic on port 80.
  - It forwards all traffic to the `mallow-php-app` target group, but no targets (e.g., EC2 instances) are registered, so no traffic is being processed.
  - It operates across two AZs (`ap-south-1a` and `ap-south-1b`) for high availability.
  - Security Groups (`php-lb-sg` and the default SG) control inbound traffic, likely allowing HTTP on port 80.
  - Cross-zone load balancing is enabled, ensuring even traffic distribution across AZs.
  - HTTP/2 is enabled for better performance, but HTTPS is not configured (no SSL/TLS).

- **Intended Use**:
  - This ALB is likely meant to distribute traffic to a PHP application (`mallow-php-app`), possibly running on EC2 instances.
  - The lack of targets suggests the setup is incomplete—EC2 instances need to be registered with the target group, and health checks need to pass for the ALB to route traffic.
  - Once configured, the ALB would:
    1. Receive HTTP requests from clients.
    2. Apply Security Group rules to allow/deny traffic.
    3. Forward requests to healthy instances in the `mallow-php-app` target group.
    4. Return responses to clients, leveraging HTTP/2 for efficiency.

- **Potential Improvements**:
  - Enable HTTPS by adding a listener on port 443 with an SSL/TLS certificate (e.g., via AWS Certificate Manager).
  - Register EC2 instances with the target group and configure health checks.
  - Enable access logs for troubleshooting and auditing.
  - Integrate with AWS WAF for security if the application is exposed to the public internet.
  - Add tags for better resource management.

---

### **Summary**
The Application Load Balancer (`mallow-vpc-lb`) is a Layer 7 load balancer designed to distribute HTTP/HTTPS traffic across multiple targets. It’s used to improve scalability, availability, and performance for web applications. In this setup, it’s partially configured to handle HTTP traffic for a PHP application but lacks registered targets, so it’s not fully operational. The various tabs (Details, Listeners and Rules, Security, etc.) provide a comprehensive view of its configuration, allowing you to fine-tune its behavior for your application’s needs.

Let me know if you’d like to dive deeper into any specific component!
