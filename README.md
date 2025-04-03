# Cross-Zone Load Balancing
This is where things get interesting—it’s about how traffic gets spread across targets in different AZs. Each load balancer node handles a portion of the incoming traffic (determined by DNS, via Amazon Route 53, which points clients to node IP addresses). The default routing method is *round robin*, meaning requests are handed out one-by-one in a circular order. Cross-zone load balancing decides whether a node can send traffic to targets in *all* AZs or just its own.

## Scenario: Two AZs, 10 Targets
Imagine you have two AZs:
- AZ A: 2 targets
- AZ B: 8 targets
- Total: 10 targets
- Two load balancer nodes (one in each AZ), each getting 50% of the client traffic.

### Cross-Zone Load Balancing Enabled
- Each node can send its traffic (50%) to *all 10 targets* across both AZs.
- The traffic is evenly split: each target gets 10% of the total traffic.
- Why? Because the nodes don’t care about AZ boundaries—they distribute traffic globally across all healthy targets using round robin.
- Result: 2 targets in AZ A and 8 in AZ B each handle 10% of the load, perfectly balanced.

### Cross-Zone Load Balancing Disabled
- Each node only sends its traffic (50%) to targets in *its own AZ*.
- AZ A node: 50% of traffic split across 2 targets → each gets 25%.
- AZ B node: 50% of traffic split across 8 targets → each gets 6.25% (50 ÷ 8).
- Result: Uneven distribution. Targets in AZ A handle more load per server (25%) than those in AZ B (6.25%), because each node is limited to its own zone.

## Load Balancer Types and Defaults
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



