# Basic TCP/IP Commands

## Overview

**TCP/IP commands** are essential tools for network configuration, troubleshooting, and monitoring. These commands help diagnose connectivity issues, verify network configurations, and analyze network traffic across different operating systems.

## Universal Network Commands

### **ping**
Tests connectivity between two network hosts by sending ICMP echo requests.

#### **Basic Usage**
```bash
# Basic connectivity test
ping google.com
ping 8.8.8.8

# Send specific number of packets
ping -c 4 google.com          # Linux/macOS
ping -n 4 google.com          # Windows

# Continuous ping (stop with Ctrl+C)
ping -t google.com            # Windows
ping google.com               # Linux/macOS (default continuous)
```

#### **Advanced ping Options**
```bash
# Set packet size
ping -s 1000 google.com       # Linux/macOS
ping -l 1000 google.com       # Windows

# Set interval between packets
ping -i 2 google.com          # Linux/macOS (2 seconds)

# Flood ping (requires root/admin)
ping -f google.com            # Linux/macOS

# IPv6 ping
ping6 google.com              # Linux/macOS
ping -6 google.com            # Windows
```

#### **Interpreting ping Results**
```bash
PING google.com (172.217.12.142): 56 data bytes
64 bytes from 172.217.12.142: icmp_seq=0 ttl=54 time=15.2 ms
64 bytes from 172.217.12.142: icmp_seq=1 ttl=54 time=14.8 ms

--- google.com ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 14.8/15.0/15.2/0.2 ms
```

**Key Metrics:**
- **TTL (Time To Live)**: Hop count remaining
- **Time**: Round-trip time in milliseconds
- **Packet Loss**: Percentage of lost packets

### **traceroute / tracert**
Shows the path packets take to reach a destination.

#### **Basic Usage**
```bash
# Linux/macOS
traceroute google.com

# Windows
tracert google.com

# Specific number of hops
traceroute -m 15 google.com    # Linux/macOS
tracert -h 15 google.com       # Windows
```

#### **Advanced Options**
```bash
# Use ICMP instead of UDP (Linux/macOS)
traceroute -I google.com

# Use TCP instead of UDP
traceroute -T -p 80 google.com

# IPv6 traceroute
traceroute6 google.com         # Linux/macOS
tracert -6 google.com          # Windows
```

#### **Reading traceroute Output**
```bash
traceroute to google.com (172.217.12.142), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  1.234 ms  1.456 ms  1.789 ms
 2  10.0.0.1 (10.0.0.1)  5.123 ms  4.567 ms  4.890 ms
 3  * * *
 4  172.217.12.142 (172.217.12.142)  15.123 ms  14.567 ms  15.890 ms
```

**Understanding Output:**
- **Hop Number**: Router sequence number
- **IP Address**: Router's IP address
- **Response Times**: Three measurements per hop
- **Asterisks (*)**: Timeouts or blocked responses

### **nslookup**
Queries DNS servers to obtain domain name or IP address mapping.

#### **Basic DNS Lookup**
```bash
# Forward lookup (domain to IP)
nslookup google.com

# Reverse lookup (IP to domain)
nslookup 8.8.8.8

# Query specific DNS server
nslookup google.com 8.8.8.8
```

#### **Advanced nslookup**
```bash
# Interactive mode
nslookup
> set type=MX
> google.com
> exit

# Query specific record types
nslookup -type=MX google.com      # Mail exchange records
nslookup -type=NS google.com      # Name server records
nslookup -type=TXT google.com     # Text records
nslookup -type=AAAA google.com    # IPv6 addresses
```

### **dig** (Linux/macOS)
More powerful DNS lookup tool with detailed output.

#### **Basic dig Usage**
```bash
# Simple lookup
dig google.com

# Query specific record type
dig google.com A                  # IPv4 addresses
dig google.com AAAA               # IPv6 addresses
dig google.com MX                 # Mail exchange records
dig google.com NS                 # Name servers
dig google.com TXT                # Text records

# Query specific DNS server
dig @8.8.8.8 google.com
```

#### **Advanced dig Options**
```bash
# Short answer only
dig +short google.com

# Reverse lookup
dig -x 8.8.8.8

# Trace DNS resolution path
dig +trace google.com

# No recursion (query authoritative servers only)
dig +norecurse google.com
```

## UNIX/Linux Network Commands

### **ip** (Modern Linux)
Primary tool for network configuration and monitoring on modern Linux systems.

#### **Interface Management**
```bash
# Show all network interfaces
ip addr show
ip a                              # Short form

# Show specific interface
ip addr show eth0

# Add IP address to interface
sudo ip addr add 192.168.1.100/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Enable/disable interface
sudo ip link set eth0 up
sudo ip link set eth0 down
```

#### **Routing Management**
```bash
# Show routing table
ip route show
ip r                              # Short form

# Add default gateway
sudo ip route add default via 192.168.1.1

# Add specific route
sudo ip route add 10.0.0.0/24 via 192.168.1.1

# Delete route
sudo ip route del 10.0.0.0/24

# Show route for specific destination
ip route get 8.8.8.8
```

#### **Neighbor Table (ARP)**
```bash
# Show ARP table
ip neigh show
ip n                              # Short form

# Add static ARP entry
sudo ip neigh add 192.168.1.100 lladdr 00:11:22:33:44:55 dev eth0

# Delete ARP entry
sudo ip neigh del 192.168.1.100 dev eth0
```

### **ss** (Socket Statistics)
Modern replacement for netstat showing socket connections.

#### **Basic ss Usage**
```bash
# Show all connections
ss -a

# Show listening ports only
ss -l

# Show TCP connections
ss -t

# Show UDP connections
ss -u

# Show with process information
ss -p

# Show with numeric addresses (no name resolution)
ss -n
```

#### **Advanced ss Examples**
```bash
# TCP listening ports with processes
ss -tlnp

# Connections to specific port
ss -t state established '( dport = :80 or sport = :80 )'

# Connections from specific IP
ss src 192.168.1.100

# Statistics
ss -s
```

### **ifconfig** (Traditional Unix)
Traditional interface configuration tool (deprecated on many modern systems).

#### **Basic ifconfig Usage**
```bash
# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0

# Configure interface
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# Enable/disable interface
sudo ifconfig eth0 up
sudo ifconfig eth0 down

# Add alias IP
sudo ifconfig eth0:1 192.168.1.101
```

### **route** (Traditional Unix)
Traditional routing table management (deprecated on many modern systems).

#### **Basic route Usage**
```bash
# Show routing table
route -n                          # Numeric format

# Add default gateway
sudo route add default gw 192.168.1.1

# Add specific route
sudo route add -net 10.0.0.0/24 gw 192.168.1.1

# Delete route
sudo route del -net 10.0.0.0/24
```

### **netstat** (Traditional Unix)
Shows network connections, routing tables, and network statistics.

#### **Connection Information**
```bash
# Show all connections
netstat -a

# Show listening ports
netstat -l

# Show TCP connections
netstat -t

# Show UDP connections
netstat -u

# Show with process information
netstat -p

# Numeric output (no name resolution)
netstat -n
```

#### **Routing and Interface Information**
```bash
# Show routing table
netstat -r

# Show interface statistics
netstat -i

# Show network statistics
netstat -s
```

## Windows Network Commands

### **ipconfig**
Primary Windows tool for IP configuration.

#### **Basic ipconfig Usage**
```cmd
REM Show basic configuration
ipconfig

REM Show detailed configuration
ipconfig /all

REM Release IP address
ipconfig /release

REM Renew IP address
ipconfig /renew

REM Flush DNS cache
ipconfig /flushdns

REM Display DNS cache
ipconfig /displaydns
```

#### **Advanced ipconfig Options**
```cmd
REM Release/renew specific adapter
ipconfig /release "Ethernet"
ipconfig /renew "Ethernet"

REM Register DNS name
ipconfig /registerdns

REM Show DHCP class information
ipconfig /showclassid "Ethernet"
```

### **Windows ping**
```cmd
REM Basic ping
ping google.com

REM Continuous ping
ping -t google.com

REM Specific number of packets
ping -n 10 google.com

REM Specific packet size
ping -l 1000 google.com

REM IPv6 ping
ping -6 google.com
```

### **netstat** (Windows)
```cmd
REM Show all connections
netstat -a

REM Show with process IDs
netstat -o

REM Show listening ports
netstat -an | findstr LISTENING

REM Show routing table
netstat -r

REM Show interface statistics
netstat -e
```

### **arp** (Windows)
```cmd
REM Show ARP table
arp -a

REM Add static ARP entry
arp -s 192.168.1.100 00-11-22-33-44-55

REM Delete ARP entry
arp -d 192.168.1.100
```

### **route** (Windows)
```cmd
REM Show routing table
route print

REM Add default gateway
route add 0.0.0.0 mask 0.0.0.0 192.168.1.1

REM Add specific route
route add 10.0.0.0 mask 255.255.255.0 192.168.1.1

REM Delete route
route delete 10.0.0.0

REM Make route persistent
route -p add 10.0.0.0 mask 255.255.255.0 192.168.1.1
```

## Troubleshooting Scenarios

### **No Internet Connectivity**
```bash
# Step 1: Check local interface
ip addr show                      # Linux
ipconfig                          # Windows

# Step 2: Test local connectivity
ping 127.0.0.1                    # Loopback test

# Step 3: Test gateway connectivity
ping 192.168.1.1                  # Default gateway

# Step 4: Test external connectivity
ping 8.8.8.8                      # Google DNS

# Step 5: Test DNS resolution
nslookup google.com

# Step 6: Trace route to identify where packets stop
traceroute google.com             # Linux/macOS
tracert google.com                # Windows
```

### **Slow Network Performance**
```bash
# Check interface statistics
ip -s link show eth0              # Linux
netstat -e                        # Windows

# Monitor active connections
ss -i                             # Linux
netstat -o                        # Windows

# Test with large packets
ping -s 1472 google.com           # Linux/macOS
ping -l 1472 google.com           # Windows

# Continuous monitoring
ping -i 0.2 google.com            # Linux/macOS
```

### **DNS Issues**
```bash
# Test different DNS servers
nslookup google.com 8.8.8.8
nslookup google.com 1.1.1.1

# Clear DNS cache
sudo systemctl flush-dns          # Linux (systemd)
ipconfig /flushdns                 # Windows

# Check DNS configuration
cat /etc/resolv.conf               # Linux
ipconfig /all                      # Windows
```

### **Port Connectivity Issues**
```bash
# Check if port is open locally
ss -tlnp | grep :80               # Linux
netstat -an | findstr :80         # Windows

# Test remote port connectivity
telnet google.com 80
nc -zv google.com 80              # Linux (netcat)

# Scan ports
nmap -p 80,443 google.com         # If nmap is installed
```

## Advanced Networking Commands

### **tcpdump** (Linux/macOS)
Packet capture and analysis tool.

```bash
# Capture packets on interface
sudo tcpdump -i eth0

# Capture specific protocol
sudo tcpdump -i eth0 tcp

# Capture specific port
sudo tcpdump -i eth0 port 80

# Capture and save to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Verbose output with timestamp
sudo tcpdump -i eth0 -v -t
```

### **iptables** (Linux Firewall)
```bash
# List current rules
sudo iptables -L

# Allow specific port
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Block specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Save rules (Ubuntu/Debian)
sudo iptables-save > /etc/iptables/rules.v4
```

### **Network Performance Tools**
```bash
# Bandwidth testing with iperf3
iperf3 -s                         # Server mode
iperf3 -c server_ip               # Client mode

# Monitor network usage
iftop                             # Real-time bandwidth usage
nethogs                           # Process-specific network usage
nload                             # Network load monitoring

# Network latency testing
mtr google.com                    # Continuous traceroute
```

## Command Comparison: Linux vs Windows

| Function | Linux/Unix | Windows | Description |
|----------|------------|---------|-------------|
| **IP Configuration** | `ip addr` / `ifconfig` | `ipconfig` | Show/configure IP settings |
| **Routing Table** | `ip route` / `route` | `route print` | Display routing information |
| **Network Connections** | `ss` / `netstat` | `netstat` | Show active connections |
| **DNS Lookup** | `dig` / `nslookup` | `nslookup` | Query DNS records |
| **Connectivity Test** | `ping` | `ping` | Test network connectivity |
| **Path Tracing** | `traceroute` | `tracert` | Trace network path |
| **ARP Table** | `ip neigh` / `arp` | `arp` | Show/manage ARP entries |

## Network Troubleshooting Methodology

### **Layer-by-Layer Approach**
Following the OSI model for systematic troubleshooting:

#### **Physical Layer (Layer 1)**
```bash
# Check cable connections, link status
ip link show                      # Linux
ipconfig                          # Windows (look for "Media disconnected")

# Check interface statistics for errors
ip -s link show eth0              # Linux
netstat -e                        # Windows
```

#### **Data Link Layer (Layer 2)**
```bash
# Check ARP table
ip neigh show                     # Linux
arp -a                            # Windows

# Check for MAC address conflicts
# Monitor switch port status if accessible
```

#### **Network Layer (Layer 3)**
```bash
# Verify IP configuration
ip addr show                      # Linux
ipconfig /all                     # Windows

# Test local network connectivity
ping <gateway_ip>
ping <local_host_ip>

# Check routing table
ip route show                     # Linux
route print                       # Windows
```

#### **Transport Layer (Layer 4)**
```bash
# Check listening services
ss -tlnp                          # Linux
netstat -an                       # Windows

# Test specific ports
telnet <host> <port>
nc -zv <host> <port>              # Linux
```

### **Common Network Issues and Solutions**

#### **"Cannot reach external websites"**
```bash
# Troubleshooting steps:
1. ping 127.0.0.1                # Test loopback
2. ping <gateway_ip>              # Test gateway
3. ping 8.8.8.8                  # Test external IP
4. nslookup google.com            # Test DNS
5. ping google.com                # Test external hostname

# If step 3 works but 4/5 fail = DNS issue
# If step 2 fails = gateway/routing issue
# If step 1 fails = local network interface issue
```

#### **"Slow network performance"**
```bash
# Performance diagnostics:
ping -i 0.1 <host>                # Test latency consistency
mtr <host>                        # Continuous traceroute
iperf3 -c <server>                # Bandwidth test

# Check interface errors
ip -s link show                   # Linux
netstat -e                        # Windows

# Monitor network utilization
iftop                             # Linux
perfmon.exe                       # Windows (Performance Monitor)
```

#### **"Cannot connect to specific service"**
```bash
# Service-specific troubleshooting:
1. telnet <host> <port>           # Test port connectivity
2. ss -tlnp | grep <port>         # Check if service is listening
3. iptables -L                    # Check firewall rules (Linux)
4. netsh advfirewall show allprofiles # Check firewall (Windows)
```

## Monitoring and Logging

### **Continuous Monitoring Scripts**

#### **Linux Network Monitor Script**
```bash
#!/bin/bash
# network_monitor.sh

while true; do
    echo "=== $(date) ==="
    echo "Gateway connectivity:"
    ping -c 1 $(ip route | grep default | awk '{print $3}') 2>/dev/null && echo "OK" || echo "FAIL"
    
    echo "DNS resolution:"
    nslookup google.com >/dev/null 2>&1 && echo "OK" || echo "FAIL"
    
    echo "Active connections:"
    ss -tuln | wc -l
    
    echo "Interface statistics:"
    ip -s link show | grep -A1 "state UP"
    
    sleep 60
done
```

#### **Windows Network Monitor Batch**
```batch
@echo off
:loop
echo === %date% %time% ===
echo Gateway connectivity:
ping -n 1 %gateway% >nul && echo OK || echo FAIL

echo DNS resolution:
nslookup google.com >nul && echo OK || echo FAIL

echo Active connections:
netstat -an | find /c ":"

timeout /t 60 /nobreak >nul
goto loop
```

### **Log Analysis**
```bash
# Linux system logs
tail -f /var/log/syslog | grep -i network
journalctl -f -u NetworkManager    # systemd systems

# Windows event logs
eventvwr.msc                       # Event Viewer GUI
wevtutil qe System /f:text | findstr /i network
```

## Security Considerations

### **Network Security Commands**
```bash
# Check for suspicious connections
ss -tuln | grep ESTABLISHED        # Linux
netstat -an | findstr ESTABLISHED  # Windows

# Monitor failed connection attempts
grep "authentication failure" /var/log/auth.log  # Linux
eventvwr.msc                       # Windows (Security logs)

# Check firewall status
iptables -L -n                     # Linux
netsh advfirewall show allprofiles # Windows
```

### **Network Scanning Prevention**
```bash
# Limit ICMP responses (Linux)
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

# Disable ping response (Windows)
netsh advfirewall firewall add rule name="Block ICMP" protocol=icmpv4:8,any dir=in action=block
```

## Automation and Scripting

### **Automated Network Testing**
```bash
#!/bin/bash
# comprehensive_network_test.sh

HOSTS=("8.8.8.8" "1.1.1.1" "google.com" "github.com")
GATEWAY=$(ip route | grep default | awk '{print $3}')

echo "Network Connectivity Test - $(date)"
echo "========================================"

# Test gateway
echo -n "Gateway ($GATEWAY): "
ping -c 1 -W 2 $GATEWAY >/dev/null 2>&1 && echo "OK" || echo "FAIL"

# Test external hosts
for host in "${HOSTS[@]}"; do
    echo -n "$host: "
    ping -c 1 -W 2 $host >/dev/null 2>&1 && echo "OK" || echo "FAIL"
done

# DNS resolution test
echo -n "DNS Resolution: "
nslookup google.com >/dev/null 2>&1 && echo "OK" || echo "FAIL"

# Interface status
echo "Interface Status:"
ip link show | grep "state UP" | awk '{print $2}' | sed 's/:$//'

# Route table summary
echo "Default Route:"
ip route | grep default
```

### **Windows PowerShell Network Testing**
```powershell
# PowerShell network connectivity test
$hosts = @("8.8.8.8", "1.1.1.1", "google.com", "github.com")

Write-Host "Network Connectivity Test - $(Get-Date)"
Write-Host "=========================================="

# Test each host
foreach ($host in $hosts) {
    $result = Test-Connection -ComputerName $host -Count 1 -Quiet
    Write-Host "$host : $(if($result){'OK'}else{'FAIL'})"
}

# Get network adapter status
Get-NetAdapter | Where-Object {$_.Status -eq "Up"} | Select-Object Name, InterfaceDescription
```

## Best Practices

### **Regular Network Maintenance**
- **Monitor Interface Statistics**: Check for errors, dropped packets
- **Update ARP Tables**: Clear stale entries periodically
- **DNS Cache Management**: Flush DNS cache when troubleshooting
- **Log Analysis**: Review network logs for anomalies
- **Performance Baselines**: Establish normal performance metrics

### **Troubleshooting Guidelines**
- **Start Simple**: Begin with basic connectivity tests
- **Work Bottom-Up**: Follow OSI layers systematically
- **Document Changes**: Record configuration modifications
- **Test After Changes**: Verify network functionality after modifications
- **Have Rollback Plans**: Prepare to revert changes if needed

### **Security Best Practices**
- **Minimize Information Disclosure**: Limit responses to network probes
- **Monitor Connections**: Regularly check for unauthorized connections
- **Firewall Configuration**: Properly configure host and network firewalls
- **Access Control**: Implement appropriate network access controls
- **Logging**: Enable comprehensive network logging

## Command Reference Quick Sheet

### **Essential Commands**
```bash
# Connectivity
ping <host>                       # Test connectivity
traceroute/tracert <host>         # Trace network path

# Configuration
ip addr show / ipconfig           # Show IP configuration
ip route show / route print      # Show routing table

# Monitoring
ss -tlnp / netstat -an           # Show connections
ip -s link / netstat -e          # Interface statistics

# DNS
nslookup <host>                  # DNS lookup
dig <host>                       # Detailed DNS query (Linux)

# Troubleshooting
ping <gateway>                   # Test gateway connectivity
ping 8.8.8.8                    # Test external connectivity
nslookup google.com              # Test DNS resolution
```

## Summary

TCP/IP commands are fundamental tools for network administration and troubleshooting. Understanding these commands enables:

**Key Capabilities:**
- **Connectivity Testing**: Verify network reachability and performance
- **Configuration Management**: View and modify network settings
- **Problem Diagnosis**: Systematically identify network issues
- **Performance Monitoring**: Track network utilization and health

**Platform Considerations:**
- **Linux/Unix**: Rich set of tools with modern (`ip`, `ss`) and traditional (`ifconfig`, `netstat`) options
- **Windows**: Integrated tools focused on ease of use and GUI integration
- **Cross-Platform**: Core tools like `ping` and `traceroute` work similarly across platforms

**Best Practices:**
- **Systematic Approach**: Use layer-by-layer troubleshooting methodology
- **Documentation**: Record configurations and troubleshooting steps
- **Automation**: Script repetitive tasks and monitoring
- **Security**: Consider security implications of network commands and configurations

Mastering these commands provides the foundation for effective network administration, enabling quick problem resolution and proactive network management.