# Network Troubleshooting

## Overview

**Network Troubleshooting** is the systematic process of identifying, diagnosing, and resolving network connectivity issues, performance problems, and configuration errors. Effective troubleshooting requires understanding network protocols, tools, and methodologies.

## Troubleshooting Methodology

### **Structured Approach**
1. **Define the Problem**: Clearly identify symptoms and scope
2. **Gather Information**: Collect relevant data and documentation
3. **Consider Possibilities**: List potential causes
4. **Create Action Plan**: Prioritize steps based on likelihood
5. **Implement and Test**: Execute solutions systematically
6. **Document Results**: Record findings and solutions

### **OSI Layer Troubleshooting**
Work through network layers systematically:

```
Layer 7 (Application)  ← Start here for app-specific issues
Layer 6 (Presentation)
Layer 5 (Session)
Layer 4 (Transport)    ← TCP/UDP connectivity issues
Layer 3 (Network)      ← IP routing and addressing
Layer 2 (Data Link)    ← Switching and VLAN issues
Layer 1 (Physical)     ← Cable, interface problems
```

### **Bottom-Up vs Top-Down**
- **Bottom-Up**: Start with physical layer, work up
- **Top-Down**: Start with application, work down
- **Divide and Conquer**: Test middle layers first
- **Follow the Path**: Trace data flow end-to-end

## Layer 1 (Physical) Troubleshooting

### **Common Physical Issues**
- **Cable Problems**: Damaged, loose, or incorrect cables
- **Interface Failures**: Hardware port failures
- **Power Issues**: Insufficient power, power supply failures
- **Environmental**: Temperature, humidity, interference
- **Connector Issues**: Dirty, corroded, or damaged connectors

### **Physical Layer Testing**
```bash
# Check interface status (Linux)
ip link show
ethtool eth0

# Interface statistics
ip -s link show eth0
cat /proc/net/dev | grep eth0

# Check for errors
ethtool -S eth0 | grep -E "(error|drop|fail)"

# Test cable (if supported)
ethtool --test eth0

# Interface configuration details
ethtool eth0
```

```cmd
REM Windows interface checking
ipconfig /all
netsh interface show interface

REM Interface statistics
netstat -e

REM Device Manager for hardware issues
devmgmt.msc
```

### **Physical Troubleshooting Tools**
- **Cable Tester**: Verify cable continuity and wiring
- **Tone Generator**: Trace cables in walls/conduits
- **Optical Power Meter**: Test fiber optic signal strength
- **Network Interface Tester**: Hardware-level interface testing
- **Protocol Analyzer**: Capture and analyze traffic

## Layer 2 (Data Link) Troubleshooting

### **Common Data Link Issues**
- **VLAN Misconfigurations**: Wrong VLAN assignments
- **Spanning Tree Problems**: Loops, blocked ports
- **MAC Address Conflicts**: Duplicate MAC addresses
- **Switch Port Issues**: Port security, speed/duplex mismatches
- **Frame Errors**: CRC errors, collisions, runts

### **Switching Troubleshooting**
```bash
# Check MAC address table (Linux bridge)
brctl showmacs br0

# VLAN information
bridge vlan show

# Check for switching loops
ping -f -c 100 192.168.1.1 | grep "packet loss"
```

```cisco
! Cisco switch troubleshooting
show mac address-table
show spanning-tree
show interfaces status
show interfaces fastethernet 0/1
show vlan brief

! Check for errors
show interfaces | include error
show interfaces counters errors

! Port security
show port-security interface fastethernet 0/1
```

### **Common L2 Problems and Solutions**

#### **VLAN Issues**
```cisco
! Check VLAN configuration
show vlan brief
show interfaces trunk

! Verify VLAN assignment
show interfaces fastethernet 0/1 switchport

! Fix VLAN assignment
interface fastethernet 0/1
 switchport access vlan 10
```

#### **Spanning Tree Issues**
```cisco
! Check spanning tree status
show spanning-tree

! Check for blocked ports
show spanning-tree interface fastethernet 0/1

! Root bridge verification
show spanning-tree root

! Fix spanning tree issues
spanning-tree vlan 1 priority 4096  ! Make root bridge
```

## Layer 3 (Network) Troubleshooting

### **Common Network Layer Issues**
- **IP Address Conflicts**: Duplicate IP addresses
- **Subnet Mask Errors**: Incorrect subnet boundaries
- **Routing Problems**: Missing or incorrect routes
- **DHCP Issues**: IP assignment failures
- **DNS Problems**: Name resolution failures

### **IP Configuration Testing**
```bash
# Check IP configuration
ip addr show
ip route show

# Test IP connectivity
ping 127.0.0.1        # Loopback test
ping 192.168.1.1      # Default gateway
ping 8.8.8.8          # External connectivity
ping google.com       # DNS resolution + connectivity

# Detailed connectivity test
traceroute google.com
mtr google.com

# Check ARP table
ip neigh show
arp -a
```

```cmd
REM Windows IP troubleshooting
ipconfig /all
route print

REM Test connectivity
ping 127.0.0.1
ping 192.168.1.1
ping 8.8.8.8
ping google.com

REM Trace route
tracert google.com
pathping google.com

REM ARP table
arp -a
```

### **Common L3 Problems and Solutions**

#### **IP Address Conflicts**
```bash
# Detect IP conflicts
ping -c 1 192.168.1.100
arping -c 3 192.168.1.100

# Check for duplicate IPs
arp -a | grep "192.168.1.100"

# Fix conflicts
sudo ip addr del 192.168.1.100/24 dev eth0
sudo ip addr add 192.168.1.101/24 dev eth0
```

#### **Routing Issues**
```bash
# Check routing table
ip route show
route -n

# Test specific route
ip route get 8.8.8.8

# Add missing route
sudo ip route add 10.0.0.0/24 via 192.168.1.1

# Fix default gateway
sudo ip route del default
sudo ip route add default via 192.168.1.1
```

#### **DHCP Problems**
```bash
# Release and renew IP
sudo dhclient -r eth0    # Release
sudo dhclient eth0       # Renew

# Manual DHCP request
sudo dhclient -v eth0

# Check DHCP lease
cat /var/lib/dhcp/dhclient.leases
```

```cmd
REM Windows DHCP troubleshooting
ipconfig /release
ipconfig /renew

REM DHCP lease information
ipconfig /all | findstr "DHCP\|Lease"
```

## Layer 4 (Transport) Troubleshooting

### **Common Transport Issues**
- **Port Connectivity**: Blocked or filtered ports
- **Firewall Rules**: Blocking legitimate traffic
- **NAT Issues**: Port translation problems
- **Load Balancer Issues**: Backend server failures
- **Session Limits**: Connection exhaustion

### **TCP/UDP Testing**
```bash
# Check listening ports
ss -tuln
netstat -tuln

# Test port connectivity
telnet google.com 80
nc -zv google.com 80

# UDP port testing
nc -u -zv 8.8.8.8 53

# Show active connections
ss -tun
netstat -tun

# Connection statistics
ss -s
netstat -s
```

### **Firewall Troubleshooting**
```bash
# Linux iptables
iptables -L -n -v
iptables -t nat -L -n -v

# Check for blocked connections
iptables -L INPUT | grep DROP
iptables -L OUTPUT | grep DROP

# Temporarily allow traffic (testing)
iptables -I INPUT -p tcp --dport 80 -j ACCEPT

# UFW (Ubuntu)
ufw status verbose
ufw show listening
```

```cmd
REM Windows firewall
netsh advfirewall show allprofiles
netsh advfirewall firewall show rule all

REM Check blocked connections
netsh advfirewall monitor show currentprofile
```

### **NAT Troubleshooting**
```bash
# Check NAT translation table
cat /proc/net/nf_conntrack
conntrack -L

# NAT statistics
iptables -t nat -L -n -v

# Check for NAT exhaustion
echo "NAT entries: $(cat /proc/net/nf_conntrack | wc -l)"
echo "NAT limit: $(cat /proc/sys/net/netfilter/nf_conntrack_max)"
```

## Application Layer Troubleshooting

### **Common Application Issues**
- **DNS Resolution**: Name resolution failures
- **Certificate Issues**: SSL/TLS certificate problems
- **Service Failures**: Application not responding
- **Performance Issues**: Slow response times
- **Configuration Errors**: Wrong application settings

### **DNS Troubleshooting**
```bash
# Test DNS resolution
nslookup google.com
dig google.com

# Test different DNS servers
nslookup google.com 8.8.8.8
dig @1.1.1.1 google.com

# Check DNS configuration
cat /etc/resolv.conf

# Test reverse DNS
nslookup 8.8.8.8
dig -x 8.8.8.8

# DNS cache issues
sudo systemctl restart systemd-resolved
```

```cmd
REM Windows DNS troubleshooting
nslookup google.com
nslookup google.com 8.8.8.8

REM Flush DNS cache
ipconfig /flushdns
ipconfig /displaydns

REM Register DNS
ipconfig /registerdns
```

### **HTTP/HTTPS Troubleshooting**
```bash
# Test HTTP connectivity
curl -I http://google.com
wget --spider http://google.com

# Test HTTPS with details
curl -vI https://google.com
openssl s_client -connect google.com:443

# Check SSL certificate
echo | openssl s_client -connect google.com:443 | openssl x509 -text

# HTTP headers analysis
curl -v http://example.com 2>&1 | grep ">"
```

### **Application Performance**
```bash
# Measure response times
time curl -s http://example.com > /dev/null

# Monitor application ports
ss -tlnp | grep :80
netstat -tlnp | grep :80

# Check application logs
tail -f /var/log/apache2/error.log
journalctl -f -u nginx.service

# Process monitoring
top -p $(pgrep nginx)
htop -p $(pgrep apache2)
```

## Network Tools and Utilities

### **Basic Connectivity Tools**

#### **ping**
```bash
# Basic connectivity test
ping google.com

# Continuous ping
ping -c 10 google.com      # 10 packets
ping -i 2 google.com       # 2-second intervals
ping -s 1000 google.com    # Large packet size

# Flood ping (root required)
ping -f google.com

# IPv6 ping
ping6 google.com
```

#### **traceroute/tracert**
```bash
# Trace network path
traceroute google.com

# Use ICMP instead of UDP
traceroute -I google.com

# Use TCP
traceroute -T -p 80 google.com

# Show numeric IPs only
traceroute -n google.com

# IPv6 traceroute
traceroute6 google.com
```

#### **mtr (My Traceroute)**
```bash
# Interactive traceroute
mtr google.com

# Report mode
mtr -r -c 10 google.com

# JSON output
mtr -j google.com

# Force IPv6
mtr -6 google.com
```

### **Advanced Network Tools**

#### **nmap**
```bash
# Basic host discovery
nmap 192.168.1.0/24

# Port scanning
nmap -p 80,443 google.com
nmap -p 1-1000 192.168.1.100

# Service detection
nmap -sV 192.168.1.100

# OS detection
nmap -O 192.168.1.100

# Stealth scan
nmap -sS 192.168.1.100
```

#### **tcpdump/Wireshark**
```bash
# Capture all traffic
tcpdump -i eth0

# Capture specific protocol
tcpdump -i eth0 tcp
tcpdump -i eth0 udp port 53

# Capture with ASCII output
tcpdump -i eth0 -A

# Capture to file
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Filter by host
tcpdump -i eth0 host 192.168.1.100

# Complex filters
tcpdump -i eth0 'tcp port 80 and host 192.168.1.100'
```

#### **netstat/ss**
```bash
# Show all connections
netstat -an
ss -an

# Show listening ports
netstat -ln
ss -ln

# Show processes
netstat -lnp
ss -lnp

# Show TCP connections
netstat -tn
ss -tn

# Show statistics
netstat -s
ss -s
```

### **Specialized Tools**

#### **iftop**
```bash
# Real-time bandwidth usage
iftop -i eth0

# Show port numbers
iftop -i eth0 -P

# Show bandwidth in bits
iftop -i eth0 -B

# Filter by network
iftop -i eth0 -f "net 192.168.1.0/24"
```

#### **nethogs**
```bash
# Network usage by process
nethogs

# Monitor specific interface
nethogs eth0

# Update delay
nethogs -d 1 eth0
```

#### **iperf3**
```bash
# Server mode
iperf3 -s

# Client mode
iperf3 -c server_ip

# UDP test
iperf3 -c server_ip -u

# Specific duration
iperf3 -c server_ip -t 30

# Multiple streams
iperf3 -c server_ip -P 4
```

## Troubleshooting Scenarios

### **Scenario 1: No Internet Connectivity**

#### **Symptoms**
- Cannot reach external websites
- Can access local resources
- DNS resolution may or may not work

#### **Troubleshooting Steps**
```bash
# Step 1: Check local interface
ip addr show
# Look for: IP address assigned, interface UP

# Step 2: Test loopback
ping 127.0.0.1
# Should work if TCP/IP stack is functional

# Step 3: Test local network
ping 192.168.1.1  # Default gateway
# Tests local network connectivity

# Step 4: Check routing
ip route show
# Verify default route exists

# Step 5: Test external IP
ping 8.8.8.8
# Tests internet connectivity without DNS

# Step 6: Test DNS
nslookup google.com
# If step 5 works but this fails, DNS issue

# Step 7: Test external hostname
ping google.com
# Combines DNS + connectivity
```

#### **Common Causes and Solutions**
```bash
# No IP address (DHCP issue)
sudo dhclient eth0

# Wrong default gateway
sudo ip route del default
sudo ip route add default via 192.168.1.1

# DNS issues
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Firewall blocking
sudo iptables -L
sudo ufw status
```

### **Scenario 2: Slow Network Performance**

#### **Symptoms**
- High latency
- Low throughput
- Timeouts
- Intermittent connectivity

#### **Performance Testing**
```bash
# Bandwidth test
iperf3 -c speedtest.server.com

# Latency test
ping -c 100 google.com | tail -1

# Packet loss test
ping -c 1000 google.com | grep "packet loss"

# Interface utilization
iftop -i eth0

# Check for errors
ethtool -S eth0 | grep error
```

#### **Common Performance Issues**
```bash
# Duplex mismatch
ethtool eth0 | grep Duplex
sudo ethtool -s eth0 speed 1000 duplex full autoneg on

# High CPU usage
top | grep "Cpu(s)"
iostat 1

# Network congestion
ss -i  # Check congestion window
```

### **Scenario 3: Intermittent Connectivity**

#### **Symptoms**
- Connection works sometimes
- Frequent timeouts
- Connections drop unexpectedly

#### **Diagnostic Approach**
```bash
# Continuous monitoring
ping -i 1 google.com | ts  # timestamp each ping

# Monitor interface
watch 'ip link show eth0'

# Monitor routing
watch 'ip route show'

# Check for flapping
journalctl -f | grep -i "link\|interface\|network"

# Monitor ARP table
watch 'ip neigh show'
```

### **Scenario 4: DNS Resolution Issues**

#### **Symptoms**
- IP addresses work, hostnames don't
- Some domains resolve, others don't
- Slow DNS lookups

#### **DNS Troubleshooting**
```bash
# Check DNS configuration
cat /etc/resolv.conf

# Test different DNS servers
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com
dig @192.168.1.1 google.com

# Check for DNS filtering
dig google.com +trace

# DNS cache issues
sudo systemctl restart systemd-resolved

# DNS server connectivity
telnet 8.8.8.8 53
```

### **Scenario 5: Application Not Accessible**

#### **Symptoms**
- Service unreachable from remote hosts
- Works locally but not remotely
- Connection refused or timeout

#### **Service Troubleshooting**
```bash
# Check if service is running
systemctl status nginx
ps aux | grep nginx

# Check listening ports
ss -tlnp | grep :80
netstat -tlnp | grep :80

# Test local connectivity
curl -I http://localhost
telnet localhost 80

# Check firewall
iptables -L | grep 80
ufw status | grep 80

# Check binding
ss -tlnp | grep nginx
```

## Network Monitoring and Logging

### **System Logs**
```bash
# Network-related system logs
journalctl -u NetworkManager
journalctl -u systemd-networkd

# Kernel network messages
dmesg | grep -i network
dmesg | grep -i eth0

# Syslog network entries
grep -i network /var/log/syslog
grep -i dhcp /var/log/syslog
```

### **Interface Monitoring**
```bash
# Real-time interface statistics
watch 'cat /proc/net/dev'

# Interface errors
watch 'ip -s link show eth0'

# Traffic monitoring
vnstat -i eth0
vnstat -i eth0 -l  # Live mode
```

### **Connection Monitoring**
```bash
# Monitor new connections
ss -o state established | wc -l

# Track connection states
watch 'ss -tan | awk "NR > 1 {print \$1}" | sort | uniq -c'

# Monitor specific application
watch 'ss -tlnp | grep nginx'
```

## Automated Troubleshooting Scripts

### **Network Health Check Script**
```bash
#!/bin/bash
# network_health_check.sh

echo "=== Network Health Check ==="
echo "Date: $(date)"
echo ""

# Check interfaces
echo "=== Interface Status ==="
ip link show | grep -E "^[0-9]|state"
echo ""

# Check IP configuration
echo "=== IP Configuration ==="
ip addr show | grep -E "inet |^[0-9]"
echo ""

# Check routing
echo "=== Routing Table ==="
ip route show
echo ""

# Connectivity tests
echo "=== Connectivity Tests ==="
HOSTS=("127.0.0.1" "$(ip route | grep default | awk '{print $3}')" "8.8.8.8" "google.com")
for host in "${HOSTS[@]}"; do
    if [ -n "$host" ]; then
        if ping -c 1 -W 2 "$host" &>/dev/null; then
            echo "✓ $host - OK"
        else
            echo "✗ $host - FAIL"
        fi
    fi
done
echo ""

# DNS test
echo "=== DNS Test ==="
if nslookup google.com &>/dev/null; then
    echo "✓ DNS Resolution - OK"
else
    echo "✗ DNS Resolution - FAIL"
fi
echo ""

# Service status
echo "=== Key Services ==="
services=("NetworkManager" "systemd-networkd" "systemd-resolved")
for service in "${services[@]}"; do
    if systemctl is-active "$service" &>/dev/null; then
        echo "✓ $service - Active"
    else
        echo "✗ $service - Inactive"
    fi
done
```

### **Port Connectivity Test Script**
```bash
#!/bin/bash
# port_test.sh

HOST=$1
PORTS=$2

if [ $# -ne 2 ]; then
    echo "Usage: $0 <host> <port1,port2,port3>"
    exit 1
fi

echo "Testing connectivity to $HOST"
echo "================================"

IFS=',' read -ra PORT_ARRAY <<< "$PORTS"
for port in "${PORT_ARRAY[@]}"; do
    if timeout 3 bash -c "</dev/tcp/$HOST/$port" 2>/dev/null; then
        echo "✓ Port $port - OPEN"
    else
        echo "✗ Port $port - CLOSED/FILTERED"
    fi
done
```

### **Network Performance Monitor**
```bash
#!/bin/bash
# performance_monitor.sh

INTERFACE="eth0"
INTERVAL=5
LOG_FILE="/var/log/network_performance.log"

while true; do
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
    
    # Get interface statistics
    RX_BYTES=$(cat /sys/class/net/$INTERFACE/statistics/rx_bytes)
    TX_BYTES=$(cat /sys/class/net/$INTERFACE/statistics/tx_bytes)
    RX_PACKETS=$(cat /sys/class/net/$INTERFACE/statistics/rx_packets)
    TX_PACKETS=$(cat /sys/class/net/$INTERFACE/statistics/tx_packets)
    RX_ERRORS=$(cat /sys/class/net/$INTERFACE/statistics/rx_errors)
    TX_ERRORS=$(cat /sys/class/net/$INTERFACE/statistics/tx_errors)
    
    # Log to file
    echo "$TIMESTAMP,$RX_BYTES,$TX_BYTES,$RX_PACKETS,$TX_PACKETS,$RX_ERRORS,$TX_ERRORS" >> $LOG_FILE
    
    sleep $INTERVAL
done
```

## Advanced Troubleshooting Techniques

### **Packet Capture Analysis**
```bash
# Capture specific traffic
tcpdump -i eth0 -w problem.pcap 'host 192.168.1.100 and port 80'

# Analyze captures
tcpdump -r problem.pcap -n
tcpdump -r problem.pcap -A  # ASCII output

# Filter captured data
tcpdump -r problem.pcap 'tcp[tcpflags] & tcp-syn != 0'  # SYN packets
tcpdump -r problem.pcap 'icmp'                           # ICMP only
```

### **Network Simulation**
```bash
# Simulate packet loss
tc qdisc add dev eth0 root netem loss 1%

# Simulate latency
tc qdisc add dev eth0 root netem delay 100ms

# Simulate bandwidth limit
tc qdisc add dev eth0 root handle 1: htb default 30
tc class add dev eth0 parent 1: classid 1:1 htb rate 1mbit

# Remove simulation
tc qdisc del dev eth0 root
```

### **Performance Baseline**
```bash
# Create baseline measurements
echo "=== Network Baseline $(date) ===" > baseline.txt
echo "Interface Statistics:" >> baseline.txt
ip -s link show >> baseline.txt

echo "Connectivity Tests:" >> baseline.txt
for host in google.com cloudflare.com; do
    echo "Ping $host:" >> baseline.txt
    ping -c 10 $host | tail -1 >> baseline.txt
done

echo "Bandwidth Test:" >> baseline.txt
iperf3 -c speedtest.net -t 10 >> baseline.txt
```

## Troubleshooting Best Practices

### **Documentation**
- **Problem Description**: Detailed symptom documentation
- **Network Topology**: Up-to-date network diagrams
- **Configuration Records**: Baseline configurations
- **Change Logs**: Record of recent changes
- **Solution Database**: Known issues and fixes

### **Testing Methodology**
- **Isolate Variables**: Change one thing at a time
- **Test Before/After**: Verify changes fix the problem
- **Rollback Plan**: Ability to undo changes
- **Impact Assessment**: Consider change effects
- **Verification**: Confirm problem resolution

### **Tool Organization**
```bash
# Create troubleshooting toolkit
mkdir -p ~/network-tools/scripts
mkdir -p ~/network-tools/configs
mkdir -p ~/network-tools/logs

# Essential tools checklist
tools=(
    "ping" "traceroute" "mtr" "nmap" "tcpdump"
    "ss" "netstat" "iftop" "nethogs" "iperf3"
    "dig" "nslookup" "curl" "wget" "telnet"
)

for tool in "${tools[@]}"; do
    if ! command -v $tool &> /dev/null; then
        echo "Missing: $tool"
    fi
done
```

### **Emergency Procedures**
```bash
# Network emergency response
echo "1. Document current state"
echo "2. Identify affected systems"
echo "3. Implement temporary fixes"
echo "4. Escalate if needed"
echo "5. Implement permanent solution"
echo "6. Document resolution"

# Quick restoration commands
# Restart networking
sudo systemctl restart NetworkManager

# Reset network interface
sudo ip link set eth0 down
sudo ip link set eth0 up

# Flush and renew DHCP
sudo dhclient -r eth0 && sudo dhclient eth0

# Reset routing table
sudo ip route flush table main
sudo dhclient eth0
```

## Summary

Network troubleshooting is a critical skill requiring systematic methodology, appropriate tools, and deep understanding of network protocols and technologies.

**Key Success Factors:**
- **Systematic Approach**: Use structured troubleshooting methodology
- **Layer Understanding**: Know OSI/TCP-IP model for organized diagnosis
- **Tool Proficiency**: Master essential networking tools and commands
- **Documentation**: Maintain accurate network documentation and logs

**Essential Tools:**
- **Basic Connectivity**: ping, traceroute, mtr for connectivity testing
- **Interface Analysis**: ip, ethtool, ss for interface and connection analysis
- **Traffic Analysis**: tcpdump, Wireshark for packet-level troubleshooting
- **Performance Testing**: iperf3, iftop, nethogs for performance analysis

**Troubleshooting Process:**
- **Problem Definition**: Clearly identify symptoms and scope
- **Information Gathering**: Collect relevant data and baseline information
- **Hypothesis Testing**: Test potential causes systematically
- **Solution Implementation**: Apply fixes with proper testing and rollback plans
- **Documentation**: Record findings and solutions for future reference

**Best Practices:**
- **Baseline Establishment**: Know normal network behavior and performance
- **Change Management**: Control and document network changes
- **Monitoring**: Implement proactive monitoring and alerting
- **Skill Development**: Continuously update troubleshooting skills and knowledge

Effective network troubleshooting combines technical knowledge, practical experience, and systematic methodology to quickly identify and resolve network issues while minimizing business impact.