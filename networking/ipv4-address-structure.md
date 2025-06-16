# IPv4 Address Structure

## Overview

**IPv4 (Internet Protocol version 4)** is the fourth version of the Internet Protocol and the most widely used protocol for addressing devices on networks. It uses 32-bit addresses, allowing for approximately 4.3 billion unique addresses.

## IPv4 Address Format

### **Binary Representation**
IPv4 addresses are 32-bit numbers typically represented in dotted decimal notation:
- **32 bits** divided into **4 octets** (8 bits each)
- Each octet ranges from **0 to 255** in decimal
- Example: `192.168.1.1`

### **Dotted Decimal Notation**
```
192.168.1.1
│   │   │ │
│   │   │ └── 4th octet (1)
│   │   └──── 3rd octet (1) 
│   └──────── 2nd octet (168)
└────────────── 1st octet (192)
```

### **Binary to Decimal Conversion**
```
192.168.1.1 in binary:
11000000.10101000.00000001.00000001

Breaking down 192:
128 + 64 + 0 + 0 + 0 + 0 + 0 + 0 = 192
 │   │   │   │   │   │   │   │
 2⁷  2⁶  2⁵  2⁴  2³  2²  2¹  2⁰
```

## IPv4 Address Classes

IPv4 addresses are traditionally divided into classes based on the first octet:

### **Class A Networks**
- **Range**: 1.0.0.0 to 126.255.255.255
- **First Bit**: 0
- **Network Bits**: 8 (first octet)
- **Host Bits**: 24 (last three octets)
- **Subnet Mask**: 255.0.0.0 (/8)
- **Networks**: 126 networks
- **Hosts per Network**: 16,777,214

**Example:**
```
10.0.0.0/8
Network: 10.0.0.0
Broadcast: 10.255.255.255
Usable IPs: 10.0.0.1 to 10.255.255.254
```

### **Class B Networks**
- **Range**: 128.0.0.0 to 191.255.255.255
- **First Two Bits**: 10
- **Network Bits**: 16 (first two octets)
- **Host Bits**: 16 (last two octets)
- **Subnet Mask**: 255.255.0.0 (/16)
- **Networks**: 16,384 networks
- **Hosts per Network**: 65,534

**Example:**
```
172.16.0.0/16
Network: 172.16.0.0
Broadcast: 172.16.255.255
Usable IPs: 172.16.0.1 to 172.16.255.254
```

### **Class C Networks**
- **Range**: 192.0.0.0 to 223.255.255.255
- **First Three Bits**: 110
- **Network Bits**: 24 (first three octets)
- **Host Bits**: 8 (last octet)
- **Subnet Mask**: 255.255.255.0 (/24)
- **Networks**: 2,097,152 networks
- **Hosts per Network**: 254

**Example:**
```
192.168.1.0/24
Network: 192.168.1.0
Broadcast: 192.168.1.255
Usable IPs: 192.168.1.1 to 192.168.1.254
```

### **Class D (Multicast)**
- **Range**: 224.0.0.0 to 239.255.255.255
- **First Four Bits**: 1110
- **Purpose**: Multicast addressing
- **No Host/Network Division**: Used for group communication

### **Class E (Experimental)**
- **Range**: 240.0.0.0 to 255.255.255.255
- **First Four Bits**: 1111
- **Purpose**: Experimental and research use
- **Not for General Use**: Reserved for future use

## Special IPv4 Address Ranges

### **Private Address Ranges (RFC 1918)**
These addresses are not routed on the public internet:

```
Class A: 10.0.0.0/8
- Range: 10.0.0.0 to 10.255.255.255
- Hosts: 16,777,216

Class B: 172.16.0.0/12
- Range: 172.16.0.0 to 172.31.255.255
- Hosts: 1,048,576

Class C: 192.168.0.0/16
- Range: 192.168.0.0 to 192.168.255.255
- Hosts: 65,536
```

### **Special Purpose Addresses**
```
Loopback: 127.0.0.0/8
- 127.0.0.1 (localhost)
- Used for local testing

Link-Local: 169.254.0.0/16
- Automatic assignment when DHCP fails
- Used for local communication only

Multicast: 224.0.0.0/4
- Group communication
- 224.0.0.1 (All Hosts), 224.0.0.2 (All Routers)

Broadcast: 255.255.255.255
- Limited broadcast address
- Sent to all hosts on local network
```

## Subnet Masks

### **Purpose of Subnet Masks**
Subnet masks determine which portion of an IP address represents:
- **Network portion**: Identifies the network
- **Host portion**: Identifies the device within the network

### **Subnet Mask Representation**
```
Decimal: 255.255.255.0
Binary:  11111111.11111111.11111111.00000000
CIDR:    /24

Network bits: 1s (24 bits)
Host bits: 0s (8 bits)
```

### **CIDR Notation**
**CIDR (Classless Inter-Domain Routing)** uses a slash followed by the number of network bits:

```
Common CIDR Notations:
/8  = 255.0.0.0     (Class A default)
/16 = 255.255.0.0   (Class B default)
/24 = 255.255.255.0 (Class C default)
/25 = 255.255.255.128
/26 = 255.255.255.192
/27 = 255.255.255.224
/28 = 255.255.255.240
/29 = 255.255.255.248
/30 = 255.255.255.252
```

### **Subnetting Examples**

#### **Subnetting a Class C Network**
Original: 192.168.1.0/24 (254 hosts)

**Split into 2 subnets (/25):**
```
Subnet 1: 192.168.1.0/25
- Range: 192.168.1.0 to 192.168.1.127
- Usable: 192.168.1.1 to 192.168.1.126 (126 hosts)

Subnet 2: 192.168.1.128/25
- Range: 192.168.1.128 to 192.168.1.255
- Usable: 192.168.1.129 to 192.168.1.254 (126 hosts)
```

#### **Subnetting into 4 subnets (/26):**
```
Subnet 1: 192.168.1.0/26    (192.168.1.1-62)
Subnet 2: 192.168.1.64/26   (192.168.1.65-126)
Subnet 3: 192.168.1.128/26  (192.168.1.129-190)
Subnet 4: 192.168.1.192/26  (192.168.1.193-254)
Each subnet: 62 usable hosts
```

## IP Address Ranges and Calculations

### **Calculating Network Information**
Given IP: 192.168.10.100/26

**Step 1: Convert subnet mask**
```
/26 = 255.255.255.192
Binary: 11111111.11111111.11111111.11000000
```

**Step 2: Perform AND operation**
```
IP:   192.168.10.100 = 11000000.10101000.00001010.01100100
Mask: 255.255.255.192 = 11111111.11111111.11111111.11000000
                      = 11000000.10101000.00001010.01000000
Network: 192.168.10.64
```

**Step 3: Calculate ranges**
```
Network: 192.168.10.64
Broadcast: 192.168.10.127
First Usable: 192.168.10.65
Last Usable: 192.168.10.126
Total Hosts: 62 usable (64 total - 2)
```

### **Common Subnet Calculations**

| CIDR | Subnet Mask | Network Bits | Host Bits | Subnets | Hosts/Subnet |
|------|-------------|--------------|-----------|---------|--------------|
| /25  | 255.255.255.128 | 25 | 7 | 2 | 126 |
| /26  | 255.255.255.192 | 26 | 6 | 4 | 62 |
| /27  | 255.255.255.224 | 27 | 5 | 8 | 30 |
| /28  | 255.255.255.240 | 28 | 4 | 16 | 14 |
| /29  | 255.255.255.248 | 29 | 3 | 32 | 6 |
| /30  | 255.255.255.252 | 30 | 2 | 64 | 2 |

## Default Gateway

### **What is a Default Gateway?**
The **default gateway** is the router that forwards traffic from a local network to other networks or the internet when the destination is not on the local network.

### **How Default Gateway Works**
```
Host Decision Process:
1. Is destination IP on my local network?
   - YES: Send directly to destination
   - NO: Send to default gateway

Example:
Host: 192.168.1.100/24
Gateway: 192.168.1.1

Destination 192.168.1.50:
- Same network (/24) → Direct delivery

Destination 8.8.8.8:
- Different network → Send to 192.168.1.1
```

### **Default Gateway Configuration**
```bash
# Linux - View default gateway
ip route show default
# Output: default via 192.168.1.1 dev eth0

# Linux - Set default gateway
sudo ip route add default via 192.168.1.1

# Windows - View default gateway
ipconfig
# Look for "Default Gateway" field

# Windows - Set default gateway
route add 0.0.0.0 mask 0.0.0.0 192.168.1.1
```

### **Multiple Gateways**
In complex networks, you might have multiple gateways:
```
# Specific route for internal network
ip route add 10.0.0.0/8 via 192.168.1.10

# Default route for internet
ip route add default via 192.168.1.1

# Route priority (metric)
ip route add default via 192.168.1.1 metric 100
ip route add default via 192.168.1.2 metric 200
```

## Practical Examples

### **Home Network Example**
```
ISP Router/Modem: 192.168.1.1/24
Computer: 192.168.1.100/24
Smartphone: 192.168.1.101/24
Printer: 192.168.1.102/24

Network: 192.168.1.0/24
Gateway: 192.168.1.1
DNS: 8.8.8.8, 8.8.4.4
```

### **Office Network Example**
```
Main Office: 10.1.0.0/16
- Servers: 10.1.1.0/24
- Workstations: 10.1.2.0/24
- Printers: 10.1.3.0/24
- WiFi: 10.1.4.0/24

Gateway: 10.1.0.1
Internal DNS: 10.1.1.10
```

### **Network Troubleshooting with IP Commands**
```bash
# Check current IP configuration
ip addr show

# Check routing table
ip route show

# Test connectivity to gateway
ping 192.168.1.1

# Test connectivity to external host
ping 8.8.8.8

# Trace route to destination
traceroute 8.8.8.8
```

## IPv4 Address Exhaustion

### **The Problem**
- IPv4 provides ~4.3 billion addresses
- Not all addresses are usable (private, multicast, reserved)
- Global internet growth exceeded available addresses

### **Solutions**
- **NAT (Network Address Translation)**: Multiple private IPs share one public IP
- **CIDR**: More efficient address allocation
- **IPv6**: 128-bit addressing with virtually unlimited addresses
- **Private Networks**: Use RFC 1918 addresses internally

## Best Practices

### **IP Address Planning**
- **Use Private Ranges**: For internal networks
- **Plan for Growth**: Leave room for expansion
- **Document Networks**: Maintain IP address management (IPAM)
- **Avoid Overlapping**: Ensure unique subnets across VPNs

### **Subnetting Guidelines**
- **Start Large**: Begin with larger subnets, subdivide as needed
- **Consistent Sizing**: Use consistent subnet sizes when possible
- **Reserve Addresses**: Keep some subnets for future use
- **Security Segmentation**: Use subnets to isolate different functions

### **Gateway Configuration**
- **High Availability**: Configure backup gateways when possible
- **Security**: Secure gateway devices with proper access controls
- **Monitoring**: Monitor gateway performance and connectivity
- **Documentation**: Document gateway configurations and changes

## Summary

Understanding IPv4 address structure is fundamental to network configuration and troubleshooting. Key concepts include:

**Address Structure:**
- 32-bit addresses in dotted decimal notation
- Network and host portions determined by subnet mask
- Classful addressing (A, B, C) largely replaced by CIDR

**Practical Application:**
- Private address ranges for internal networks
- Subnetting for network segmentation
- Default gateways for routing between networks
- Address planning for scalable network design

**Modern Considerations:**
- IPv4 exhaustion driving IPv6 adoption
- NAT enabling private address reuse
- CIDR providing flexible address allocation
- Security through network segmentation

Mastering IPv4 addressing enables effective network design, troubleshooting, and optimization in both small and enterprise environments.