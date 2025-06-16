# UNIX Network Commands

## Overview

UNIX and Linux systems provide a comprehensive set of command-line tools for network configuration, monitoring, and troubleshooting. This guide covers both modern and traditional networking commands available on most UNIX-like systems.

## Modern Linux Commands

### **ip Command**
The modern Linux tool for network configuration and monitoring, replacing older tools like `ifconfig` and `route`.

#### **Interface Management**
```bash
# Show all network interfaces
ip link show
ip l                              # Short form

# Show specific interface
ip link show eth0

# Enable/disable interface
sudo ip link set eth0 up
sudo ip link set eth0 down

# Change interface MAC address
sudo ip link set eth0 down
sudo ip link set eth0 address 00:11:22:33:44:55
sudo ip link set eth0 up

# Set interface MTU
sudo ip link set eth0 mtu 1500

# Show interface statistics
ip -s link show eth0
```

#### **IP Address Management**
```bash
# Show all IP addresses
ip addr show
ip a                              # Short form

# Show addresses for specific interface
ip addr show eth0
ip a s eth0                       # Short form

# Add IP address to interface
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr add 2001:db8::100/64 dev eth0  # IPv6

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Show only IPv4 addresses
ip -4 addr show

# Show only IPv6 addresses
ip -6 addr show

# Flush all addresses from interface
sudo ip addr flush dev eth0
```

#### **Routing Management**
```bash
# Show routing table
ip route show
ip r                              # Short form

# Show IPv6 routing table
ip -6 route show

# Add default gateway
sudo ip route add default via 192.168.1.1

# Add specific route
sudo ip route add 10.0.0.0/24 via 192.168.1.1 dev eth0

# Add route with metric
sudo ip route add 10.0.0.0/24 via 192.168.1.1 metric 100

# Delete route
sudo ip route del 10.0.0.0/24

# Show route for specific destination
ip route get 8.8.8.8

# Show routing cache
ip route show cache
```

#### **Neighbor Table (ARP)**
```bash
# Show ARP table
ip neighbor show
ip neigh show                     # Alternative
ip n                              # Short form

# Show neighbors for specific interface
ip neigh show dev eth0

# Add static ARP entry
sudo ip neigh add 192.168.1.100 lladdr 00:11:22:33:44:55 dev eth0

# Change ARP entry
sudo ip neigh change 192.168.1.100 lladdr 00:11:22:33:44:56 dev eth0

# Delete ARP entry
sudo ip neigh del 192.168.1.100 dev eth0

# Flush ARP table
sudo ip neigh flush all
```

### **ss Command**
Modern replacement for `netstat` for socket statistics.

#### **Basic Usage**
```bash
# Show all sockets
ss -a

# Show listening sockets only
ss -l

# Show TCP sockets
ss -t

# Show UDP sockets
ss -u

# Show UNIX domain sockets
ss -x

# Show processes using sockets
ss -p

# Show socket statistics
ss -s
```

#### **Advanced Filtering**
```bash
# Show TCP listening ports with process info
ss -tlnp

# Show established TCP connections
ss -t state established

# Show connections to specific port
ss -tn '( dport = :80 or sport = :80 )'

# Show connections from specific IP
ss -tn src 192.168.1.100

# Show socket memory usage
ss -m

# Show detailed socket information
ss -e

# Show sockets with timers
ss -o
```

#### **State Filtering**
```bash
# Connection states
ss -t state established          # Established connections
ss -t state listening           # Listening sockets
ss -t state time-wait           # TIME_WAIT connections
ss -t state syn-sent            # SYN_SENT connections
ss -t state syn-recv            # SYN_RECV connections

# Multiple states
ss -t state connected           # All connected states
ss -t state synchronizing       # All synchronizing states
```

## Traditional UNIX Commands

### **ifconfig Command**
Traditional interface configuration tool (deprecated on modern Linux).

#### **Basic Usage**
```bash
# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0

# Show only active interfaces
ifconfig -a

# Configure IP address
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# Add secondary IP (alias)
sudo ifconfig eth0:1 192.168.1.101 netmask 255.255.255.0

# Enable/disable interface
sudo ifconfig eth0 up
sudo ifconfig eth0 down

# Set interface flags
sudo ifconfig eth0 promisc       # Promiscuous mode
sudo ifconfig eth0 -promisc     # Disable promiscuous mode
```

#### **Advanced Configuration**
```bash
# Set MTU
sudo ifconfig eth0 mtu 1500

# Set MAC address
sudo ifconfig eth0 hw ether 00:11:22:33:44:55

# Configure broadcast address
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0 broadcast 192.168.1.255

# Point-to-point configuration
sudo ifconfig ppp0 10.0.0.1 pointopoint 10.0.0.2
```

### **netstat Command**
Traditional network statistics tool.

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

# Show numeric addresses (no DNS resolution)
netstat -n

# Common combinations
netstat -tuln                    # TCP/UDP listening ports, numeric
netstat -tupln                   # Include process information
netstat -rn                      # Routing table, numeric
```

#### **Network Statistics**
```bash
# Show interface statistics
netstat -i

# Show per-protocol statistics
netstat -s

# Show multicast group membership
netstat -g

# Show network interface table
netstat -ie
```

### **route Command**
Traditional routing table management.

#### **Basic Usage**
```bash
# Show routing table
route
route -n                         # Numeric format

# Add default gateway
sudo route add default gw 192.168.1.1

# Add network route
sudo route add -net 10.0.0.0/24 gw 192.168.1.1

# Add host route
sudo route add -host 10.0.0.1 gw 192.168.1.1

# Delete route
sudo route del -net 10.0.0.0/24
sudo route del default

# Add route with metric
sudo route add -net 10.0.0.0/24 gw 192.168.1.1 metric 100
```

### **arp Command**
ARP table management (traditional tool).

#### **Basic Usage**
```bash
# Show ARP table
arp -a

# Show ARP table in numeric format
arp -n

# Show ARP entry for specific host
arp 192.168.1.1

# Add static ARP entry
sudo arp -s 192.168.1.100 00:11:22:33:44:55

# Delete ARP entry
sudo arp -d 192.168.1.100

# Show ARP table for specific interface
arp -i eth0 -a
```

## Network Monitoring Commands

### **tcpdump**
Packet capture and analysis tool.

#### **Basic Packet Capture**
```bash
# Capture packets on interface
sudo tcpdump -i eth0

# Capture specific number of packets
sudo tcpdump -i eth0 -c 10

# Capture to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Verbose output
sudo tcpdump -i eth0 -v
sudo tcpdump -i eth0 -vv          # More verbose
sudo tcpdump -i eth0 -vvv         # Most verbose
```

#### **Filtering Traffic**
```bash
# Filter by protocol
sudo tcpdump -i eth0 tcp
sudo tcpdump -i eth0 udp
sudo tcpdump -i eth0 icmp

# Filter by port
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 port 22

# Filter by host
sudo tcpdump -i eth0 host 192.168.1.100
sudo tcpdump -i eth0 src 192.168.1.100
sudo tcpdump -i eth0 dst 192.168.1.100

# Complex filters
sudo tcpdump -i eth0 "tcp port 80 and host 192.168.1.100"
sudo tcpdump -i eth0 "not port 22"
```

### **iftop**
Real-time bandwidth usage by connection.

```bash
# Monitor interface bandwidth
sudo iftop -i eth0

# Show port numbers
sudo iftop -i eth0 -P

# Show bandwidth in bits instead of bytes
sudo iftop -i eth0 -B

# Filter by network
sudo iftop -i eth0 -f "net 192.168.1.0/24"
```

### **nethogs**
Network usage by process.

```bash
# Monitor network usage by process
sudo nethogs

# Monitor specific interface
sudo nethogs eth0

# Monitor multiple interfaces
sudo nethogs eth0 eth1

# Set update delay
sudo nethogs -d 1 eth0
```

### **iotop**
I/O usage monitoring (includes network I/O).

```bash
# Monitor I/O usage
sudo iotop

# Show only processes doing I/O
sudo iotop -o

# Show accumulated I/O
sudo iotop -a
```

## Network Testing Commands

### **ping**
Test network connectivity.

```bash
# Basic ping
ping google.com
ping 8.8.8.8

# Ping specific number of times
ping -c 4 google.com

# Ping with specific interval
ping -i 2 google.com              # 2 second intervals

# Ping with specific packet size
ping -s 1000 google.com

# Flood ping (requires root)
sudo ping -f google.com

# IPv6 ping
ping6 google.com
```

### **traceroute**
Trace network path to destination.

```bash
# Basic traceroute
traceroute google.com

# Use ICMP instead of UDP
sudo traceroute -I google.com

# Use TCP
sudo traceroute -T -p 80 google.com

# Set maximum hops
traceroute -m 15 google.com

# IPv6 traceroute
traceroute6 google.com
```

### **mtr**
Combination of ping and traceroute.

```bash
# Basic MTR
mtr google.com

# Run specific number of tests
mtr -c 10 google.com

# Report mode (non-interactive)
mtr -r google.com

# Use TCP instead of ICMP
mtr -T -P 80 google.com

# JSON output
mtr -j google.com
```

### **nmap**
Network exploration and security scanner.

```bash
# Scan single host
nmap 192.168.1.100

# Scan network range
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 80,443 192.168.1.100

# TCP SYN scan
nmap -sS 192.168.1.100

# UDP scan
nmap -sU 192.168.1.100

# Service version detection
nmap -sV 192.168.1.100

# OS detection
nmap -O 192.168.1.100
```

## Network Configuration Files

### **Network Interface Configuration**

#### **Debian/Ubuntu (/etc/network/interfaces)**
```bash
# Static IP configuration
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

# DHCP configuration
auto eth0
iface eth0 inet dhcp

# IPv6 configuration
iface eth0 inet6 static
    address 2001:db8::100
    netmask 64
    gateway 2001:db8::1
```

#### **Red Hat/CentOS (/etc/sysconfig/network-scripts/ifcfg-eth0)**
```bash
# Static IP configuration
DEVICE=eth0
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4

# DHCP configuration
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
```

#### **Netplan (Ubuntu 18.04+) (/etc/netplan/01-network.yaml)**
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

### **DNS Configuration (/etc/resolv.conf)**
```bash
# DNS servers
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 2001:4860:4860::8888

# Search domains
search example.com local.domain

# Options
options timeout:1 attempts:3
```

### **Host Resolution (/etc/hosts)**
```bash
# IPv4 entries
127.0.0.1       localhost
192.168.1.10    server1.example.com server1
192.168.1.11    server2.example.com server2

# IPv6 entries
::1             localhost ip6-localhost ip6-loopback
2001:db8::10    server1.example.com server1
```

## Network Service Management

### **systemd (Modern Linux)**
```bash
# Network service management
sudo systemctl status networking
sudo systemctl restart networking
sudo systemctl enable networking

# NetworkManager
sudo systemctl status NetworkManager
sudo systemctl restart NetworkManager

# Apply netplan changes (Ubuntu)
sudo netplan apply
```

### **Traditional Init Systems**
```bash
# Network service management
sudo service networking restart
sudo service network restart       # Red Hat/CentOS

# Interface management
sudo ifup eth0
sudo ifdown eth0
```

## Wireless Network Commands

### **iwconfig**
Wireless interface configuration.

```bash
# Show wireless interfaces
iwconfig

# Configure wireless settings
sudo iwconfig wlan0 essid "MyNetwork"
sudo iwconfig wlan0 key "password"

# Set wireless mode
sudo iwconfig wlan0 mode managed
sudo iwconfig wlan0 mode monitor
```

### **iw**
Modern wireless configuration tool.

```bash
# Show wireless interfaces
iw dev

# Scan for networks
sudo iw dev wlan0 scan

# Connect to network
sudo iw dev wlan0 connect "MyNetwork"

# Show link information
iw dev wlan0 link

# Show station information
iw dev wlan0 station dump
```

### **wpa_supplicant**
WPA/WPA2 wireless authentication.

```bash
# Generate configuration
wpa_passphrase "MyNetwork" "password" > /etc/wpa_supplicant/wpa_supplicant.conf

# Connect using wpa_supplicant
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf

# DHCP after connection
sudo dhclient wlan0
```

## Network Troubleshooting Workflow

### **Systematic Approach**
```bash
# 1. Check interface status
ip link show
ifconfig

# 2. Check IP configuration
ip addr show
route -n

# 3. Test local connectivity
ping 127.0.0.1                   # Loopback
ping 192.168.1.1                 # Gateway

# 4. Test external connectivity
ping 8.8.8.8                     # External IP
ping google.com                  # External hostname

# 5. Check DNS resolution
nslookup google.com
dig google.com

# 6. Trace network path
traceroute google.com
mtr google.com

# 7. Check listening services
ss -tlnp
netstat -tlnp
```

### **Common Issues and Solutions**

#### **Interface Not Up**
```bash
# Check interface status
ip link show eth0

# Bring interface up
sudo ip link set eth0 up
# or
sudo ifconfig eth0 up
```

#### **No IP Address**
```bash
# Check DHCP client
sudo dhclient eth0

# Manual IP assignment
sudo ip addr add 192.168.1.100/24 dev eth0
# or
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0
```

#### **No Default Gateway**
```bash
# Add default gateway
sudo ip route add default via 192.168.1.1
# or
sudo route add default gw 192.168.1.1
```

#### **DNS Resolution Issues**
```bash
# Check /etc/resolv.conf
cat /etc/resolv.conf

# Test specific DNS server
nslookup google.com 8.8.8.8

# Flush DNS cache (if applicable)
sudo systemctl restart systemd-resolved
```

## Performance Monitoring

### **Network Interface Statistics**
```bash
# Real-time interface statistics
watch 'cat /proc/net/dev'

# Interface statistics with ip
ip -s link show eth0

# Detailed interface information
ethtool eth0
```

### **Connection Monitoring**
```bash
# Monitor connections
watch 'ss -tuln'

# Connection count by state
ss -tan | awk 'NR > 1 {print $1}' | sort | uniq -c

# Monitor specific port
watch 'ss -tn sport = :80'
```

### **Bandwidth Monitoring**
```bash
# Real-time bandwidth
iftop -i eth0

# Historical data
vnstat -i eth0
vnstat -i eth0 -d                # Daily stats
vnstat -i eth0 -m                # Monthly stats

# Network load
nload eth0
```

## Security and Monitoring

### **Network Security Commands**
```bash
# Check for listening ports
ss -tlnp
netstat -tlnp

# Monitor network connections
ss -tn
netstat -tn

# Check for suspicious activities
who                              # Logged in users
w                                # User activities
last                             # Login history
```

### **Firewall Commands**
```bash
# iptables (traditional)
sudo iptables -L                 # List rules
sudo iptables -L -n              # Numeric output

# ufw (Ubuntu)
sudo ufw status
sudo ufw enable
sudo ufw allow 22

# firewalld (Red Hat/CentOS)
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-service=ssh
```

## Summary

UNIX network commands provide comprehensive tools for network configuration, monitoring, and troubleshooting. Understanding both modern and traditional commands is essential for effective network management.

**Key Command Categories:**
- **Configuration**: ip, ifconfig for interface and routing setup
- **Monitoring**: ss, netstat for connection and socket monitoring
- **Testing**: ping, traceroute, mtr for connectivity testing
- **Analysis**: tcpdump, iftop, nethogs for traffic analysis

**Best Practices:**
- **Use Modern Tools**: Prefer ip and ss over ifconfig and netstat
- **Systematic Troubleshooting**: Follow logical troubleshooting steps
- **Document Changes**: Record network configuration changes
- **Monitor Regularly**: Establish baseline network performance metrics

**Essential Skills:**
- **Interface Management**: Configure and monitor network interfaces
- **Routing**: Understand and configure network routing
- **Troubleshooting**: Systematically diagnose network issues
- **Monitoring**: Track network performance and security

These commands form the foundation for UNIX network administration and are essential for anyone managing Linux or UNIX-based systems in networked environments.