# Zonal Shift
### What is Zonal Shift?
Zonal shift is a feature in Amazon’s Application Recovery Controller (ARC) that lets you quickly redirect traffic away from an *Availability Zone* (AZ) that’s having issues—like hardware failures, network problems, or other impairments. Instead of letting your application struggle in a failing zone, you can shift traffic to healthy AZs within the same AWS Region with a single action. This is all about keeping your service running smoothly when things go wrong.

### How Zonal Shift Works
1. **Triggering the Shift**: You initiate a zonal shift for a specific load balancer resource tied to an AZ you want to avoid. For example, if AZ us-east-1a is impaired, you tell ARC to shift traffic away from it.
2. **Immediate Action**: ARC starts the zonal shift right away. The load balancer stops sending new traffic to targets in the affected AZ.
3. **Handling In-Progress Connections**: Existing connections in the impaired AZ aren’t cut off instantly. It takes a few minutes (typically) to let those in-progress requests complete or timeout gracefully. This avoids abruptly dropping users mid-action.
4. **Health Checks and IP Addresses**: Behind the scenes, ARC works with the load balancer’s health checks and zonal IP addresses. Each AZ has its own load balancer node with an IP address. When you shift traffic away, that node’s IP is effectively sidelined, and traffic reroutes to nodes in other AZs. For more technical details, the ARC Developer Guide explains how health checks and DNS updates play into this.

### Where Zonal Shift Fits with Load Balancers
Remember how a load balancer distributes traffic across nodes in enabled AZs? Zonal shift builds on that:
- Normally, a load balancer sends traffic to healthy targets across all enabled AZs (or just its own AZ if cross-zone load balancing is off).
- With zonal shift, you’re overriding that normal behavior for one AZ, telling the load balancer, “Don’t use this zone, even if it’s enabled.” Traffic then flows only to targets in the remaining healthy AZs.

### Key Details and Limitations
Before you use zonal shift, there are some important things to know:

1. **Supported Load Balancers**:
   - Works with *Network Load Balancers (NLBs)*, whether cross-zone load balancing is on or off. This flexibility is great because NLBs handle raw TCP/UDP traffic and are often used for high-performance setups.
   - *Doesn’t work* with *Application Load Balancers (ALBs)* when they’re used as endpoints in AWS Global Accelerator. Global Accelerator manages traffic differently, and zonal shift isn’t compatible there.

2. **Single AZ at a Time**:
   - You can only shift traffic away from *one AZ* per action. If multiple AZs are impaired, you’d need separate zonal shifts for each. This keeps things precise but means you can’t blanket-shift an entire region at once.

3. **Cross-Zone Load Balancing Impact**:
   - If your NLB has *cross-zone load balancing on*, each node already sends traffic to targets across all AZs. A zonal shift just excludes the impaired AZ, and the remaining nodes pick up the slack evenly across their targets.
   - If it’s *off*, each node only talks to targets in its own AZ. A zonal shift removes that AZ’s node entirely, and you lose its target capacity unless other AZs can handle the load. This is a big deal—check your capacity before shifting!

4. **AWS Proactive DNS Updates**:
   - When AWS detects major issues in an AZ, it might automatically pull that zone’s load balancer IP addresses out of DNS. This overlaps with zonal shift but happens proactively. Either way, traffic stops hitting the bad AZ.

5. **Nested Load Balancers**:
   - If an ALB is a target of an NLB (a common setup for layering HTTP handling behind raw traffic management), you *must* start the zonal shift from the NLB. Why? The NLB controls the traffic flow. If you shift the ALB instead, the NLB won’t know and will keep sending traffic to the ALB in the impaired AZ, defeating the purpose.

6. **Capacity Check**:
   - Before shifting, ensure the remaining AZs have enough healthy targets to handle the traffic. If you shift away from an AZ with most of your capacity and the others can’t cope, you’ll overload them.

### Example Scenario
Let’s say you have an NLB with:
- Three AZs: A (2 targets), B (3 targets), C (5 targets).
- Cross-zone load balancing is on, so each of the 10 targets gets ~10% of the traffic.
- AZ C starts failing (e.g., network outage).

You start a zonal shift for AZ C:
- The NLB stops sending traffic to AZ C’s 5 targets.
- Traffic redistributes across AZ A and B (now 5 targets total).
- Each target in A and B now handles ~20% of the traffic (100 ÷ 5).
- After a few minutes, all connections in AZ C wrap up, and the shift is fully in effect.

If cross-zone was *off*:
- AZ C’s node (handling 33% of traffic) drops out.
- AZ A and B nodes take over, but only for their own targets (2 in A, 3 in B), potentially unbalancing the load unless you’ve planned capacity.

### Why It Matters
Zonal shift is a lifeline for resilience. Instead of scrambling to manually reconfigure your load balancer or deploy new targets during an outage, you flip a switch in ARC, and traffic moves to safety. It’s fast, controlled, and leverages the load balancer’s existing smarts—like health checks and routing rules—to keep your app alive.
