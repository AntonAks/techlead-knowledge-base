# Network Routing

## Overview

**Network Routing** is the process of selecting paths in a network along which to send data packets from source to destination. Routers use routing tables and protocols to determine the best path for forwarding packets between networks.

## Core Concepts

### **Key Components**
- **Router**: Device that forwards packets between networks
- **Routing Table**: Database of network destinations and paths
- **Next Hop**: Immediate destination for packet forwarding
- **Metric**: Cost value used to determine best path
- **Administrative Distance**: Trustworthiness of routing source

### **Routing vs Switching**
| Aspect | Routing | Switching |
|--------|---------|-----------|
| **Layer** | Layer 3 (Network) | Layer 2 (Data Link) |
| **Addressing** | IP addresses | MAC addresses |
| **Scope** | Between networks | Within network |

### **Packet Forwarding Process**
1. Router receives packet on interface
2. Examine destination IP address
3. Consult routing table for best match (longest prefix)
4. Determine next hop interface
5. Forward packet to next hop

## Routing Tables

### **Structure**
Each entry contains:
- **Destination Network**: Target network address
- **Subnet Mask/Prefix**: Network boundary definition
- **Next Hop**: IP address of next router
- **Interface**: Outgoing interface for packet
- **Metric**: Cost to reach destination

### **Example Routing Tables**

**Linux:**
```bash
# Display routing table
ip route show

# Example output:
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 scope link
10.0.0.0/8 via 192.168.1.1 dev eth0
```

**Windows:**
```cmd
route print

Network Destination    Netmask          Gateway       Interface
0.0.0.0                0.0.0.0          192.168.1.1   192.168.1.100
192.168.1.0            255.255.255.0    On-link       192.168.1.100
```

**Cisco:**
```cisco
show ip route

S*   0.0.0.0/0 [1/0] via 203.0.113.1
C    192.168.1.0/24 is directly connected, FastEthernet0/0
O    10.0.0.0/8 [110/20] via 192.168.1.2
```

### **Longest Prefix Match**
Router selects most specific route:
```
Destination: 192.168.1.100

0.0.0.0/0          -> Default route (0 bits)
192.168.0.0/16     -> Gateway B (16 bits)
192.168.1.0/24     -> Gateway C (24 bits) ← Selected
```

## Static Routing

### **Configuration Examples**

**Linux:**
```bash
# Add static route
sudo ip route add 10.0.0.0/24 via 192.168.1.1

# Add default route
sudo ip route add default via 192.168.1.1

# Delete route
sudo ip route del 10.0.0.0/24
```

**Windows:**
```cmd
# Add static route
route add 10.0.0.0 mask 255.255.255.0 192.168.1.1

# Persistent route
route -p add 10.0.0.0 mask 255.255.255.0 192.168.1.1
```

**Cisco:**
```cisco
# Basic static route
ip route 10.0.0.0 255.255.255.0 192.168.1.1

# Default route
ip route 0.0.0.0 0.0.0.0 203.0.113.1

# Floating static (backup)
ip route 10.0.0.0 255.255.255.0 192.168.1.2 150
```

### **Advantages/Disadvantages**
**Pros:** Predictable, no overhead, secure, simple
**Cons:** Manual configuration, no fault tolerance, poor scalability

## Dynamic Routing Protocols

### **Protocol Types**
- **Interior Gateway Protocols (IGP)**: Within autonomous systems
- **Exterior Gateway Protocols (EGP)**: Between autonomous systems

### **Common Protocols**

#### **RIP (Routing Information Protocol)**
```cisco
router rip
 version 2
 network 192.168.1.0
 no auto-summary
```
- **Metric**: Hop count (max 15)
- **Updates**: Every 30 seconds
- **Convergence**: Slow

#### **OSPF (Open Shortest Path First)**
```cisco
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.255.255.255 area 1
```
- **Metric**: Cost (bandwidth-based)
- **Algorithm**: Dijkstra shortest path
- **Convergence**: Fast

#### **EIGRP (Enhanced Interior Gateway Routing Protocol)**
```cisco
router eigrp 100
 network 192.168.1.0 0.0.0.255
 no auto-summary
```
- **Metric**: Composite (bandwidth, delay)
- **Algorithm**: DUAL
- **Features**: Unequal cost load balancing

#### **BGP (Border Gateway Protocol)**
```cisco
router bgp 65001
 neighbor 203.0.113.1 remote-as 65002
 network 192.168.1.0 mask 255.255.255.0
```
- **Usage**: Internet routing
- **Metric**: Path attributes
- **Policy**: Extensive control

## Administrative Distance

Cisco route preference (lower is better):
```
Connected:     0
Static:        1
eBGP:         20
EIGRP:        90
OSPF:        110
RIP:         120
iBGP:        200
```

## Default Gateways

### **Host Configuration**
```bash
# Linux
ip route add default via 192.168.1.1

# Windows
route add 0.0.0.0 mask 0.0.0.0 192.168.1.1
```

### **Router Default Routes**
```cisco
# Static default
ip route 0.0.0.0 0.0.0.0 203.0.113.1

# OSPF default origination
router ospf 1
 default-information originate
```

## Advanced Concepts

### **Load Balancing**
```bash
# Linux ECMP
ip route add 10.0.0.0/24 \
  nexthop via 192.168.1.1 weight 1 \
  nexthop via 192.168.1.2 weight 1
```

### **Route Redistribution**
```cisco
# OSPF redistributing static
router ospf 1
 redistribute static metric 20 subnets
```

### **Policy-Based Routing**
```cisco
# Route based on source
access-list 100 permit ip 192.168.1.0 0.0.0.255 any
route-map PBR permit 10
 match ip address 100
 set ip next-hop 203.0.113.2
interface FastEthernet0/0
 ip policy route-map PBR
```

## Troubleshooting

### **Common Issues**
- **Missing Routes**: Check routing table and protocol config
- **Routing Loops**: Use traceroute to identify
- **Suboptimal Paths**: Verify metrics and path selection
- **Convergence Issues**: Check protocol timers and neighbors

### **Troubleshooting Commands**

**Linux:**
```bash
ip route show                    # View routing table
ip route get 8.8.8.8            # Check specific route
traceroute google.com            # Trace path
ping 192.168.1.1                # Test connectivity
```

**Windows:**
```cmd
route print                      # View routing table
tracert google.com              # Trace path
pathping google.com             # Combined ping/trace
```

**Cisco:**
```cisco
show ip route                    # Routing table
show ip route 8.8.8.8           # Specific route
show ip protocols                # Routing protocols
show ip ospf neighbor            # OSPF neighbors
show ip eigrp neighbors          # EIGRP neighbors
```

### **Systematic Approach**
1. Check interface status
2. Verify IP configuration
3. Test local connectivity (gateway)
4. Check routing table
5. Test external connectivity
6. Verify routing protocol operation

## Best Practices

### **Design Principles**
- **Hierarchical Design**: Core/distribution/access layers
- **Redundancy**: Multiple paths for fault tolerance
- **Summarization**: Route aggregation at boundaries
- **Documentation**: Maintain network diagrams

### **Protocol Selection**
- **Small networks**: Static routes or RIP
- **Medium networks**: OSPF or EIGRP
- **Large networks**: OSPF with areas
- **Internet connectivity**: BGP

### **Security**
- **Authentication**: Use routing protocol authentication
- **Filtering**: Implement route filtering
- **Monitoring**: Track routing table changes
- **Access Control**: Limit routing protocol access

### **Performance**
- **Optimize metrics**: Tune for desired paths
- **Fast convergence**: Adjust protocol timers
- **Load balancing**: Use multiple equal-cost paths
- **Route summarization**: Reduce routing table size

## Summary

Network routing enables communication between different networks through path selection and packet forwarding. Understanding routing concepts, protocols, and troubleshooting is essential for network design and operation.

**Key Points:**
- **Static routing**: Manual, predictable, suitable for small networks
- **Dynamic routing**: Automatic, scalable, adapts to changes
- **Protocol selection**: Choose based on network size and requirements
- **Troubleshooting**: Use systematic approach and appropriate tools