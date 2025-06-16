# Windows Network Commands

## Overview

Windows provides a comprehensive set of command-line tools for network configuration, monitoring, and troubleshooting. This guide covers both traditional Windows commands and modern PowerShell cmdlets for network management.

## Basic Network Commands

### **ipconfig**
Primary Windows tool for IP configuration and information.

#### **Basic Usage**
```cmd
REM Show basic IP configuration
ipconfig

REM Show detailed configuration for all adapters
ipconfig /all

REM Show configuration for specific adapter
ipconfig /all | findstr "Ethernet adapter"
```

#### **DHCP Management**
```cmd
REM Release IP address
ipconfig /release

REM Release specific adapter
ipconfig /release "Ethernet"

REM Renew IP address
ipconfig /renew

REM Renew specific adapter
ipconfig /renew "Ethernet"

REM Show DHCP class ID
ipconfig /showclassid "Ethernet"

REM Set DHCP class ID
ipconfig /setclassid "Ethernet" "MyClassID"
```

#### **DNS Management**
```cmd
REM Display DNS resolver cache
ipconfig /displaydns

REM Flush DNS resolver cache
ipconfig /flushdns

REM Register DNS names
ipconfig /registerdns
```

### **ping**
Test network connectivity.

#### **Basic Connectivity Testing**
```cmd
REM Basic ping
ping google.com
ping 8.8.8.8

REM Ping specific number of times
ping -n 4 google.com

REM Continuous ping (Ctrl+C to stop)
ping -t google.com

REM Ping with specific packet size
ping -l 1000 google.com

REM Ping with timeout
ping -w 5000 google.com
```

#### **Advanced Ping Options**
```cmd
REM IPv6 ping
ping -6 google.com

REM Don't fragment packets
ping -f google.com

REM Record route (up to 9 hops)
ping -r 9 google.com

REM Timestamp for hop count
ping -s 4 google.com

REM Source routing
ping -j 192.168.1.1 google.com
```

### **tracert**
Trace route to destination.

```cmd
REM Basic traceroute
tracert google.com

REM Traceroute with timeout
tracert -w 5000 google.com

REM Maximum hops
tracert -h 15 google.com

REM Don't resolve hostnames
tracert -d google.com

REM IPv6 traceroute
tracert -6 google.com
```

### **pathping**
Combines ping and tracert functionality.

```cmd
REM Basic pathping (analyzes path for 300 seconds)
pathping google.com

REM Specify analysis period
pathping -p 60 google.com

REM Specify number of queries per hop
pathping -q 10 google.com

REM Maximum hops
pathping -h 15 google.com

REM Don't perform ping analysis
pathping -n google.com
```

## Network Connection Management

### **netstat**
Display network connections and statistics.

#### **Connection Information**
```cmd
REM Show all connections and listening ports
netstat -a

REM Show ethernet statistics
netstat -e

REM Show listening ports only
netstat -an | findstr LISTENING

REM Show established connections
netstat -an | findstr ESTABLISHED

REM Show connections with process IDs
netstat -o

REM Show connections with process names
netstat -b

REM Refresh every 5 seconds
netstat -an 5
```

#### **Protocol Statistics**
```cmd
REM Show statistics for all protocols
netstat -s

REM Show TCP statistics only
netstat -s -p tcp

REM Show UDP statistics only
netstat -s -p udp

REM Show IPv4 statistics
netstat -s -f

REM Show IPv6 statistics
netstat -s -6
```

#### **Routing Information**
```cmd
REM Show routing table
netstat -r

REM Show IPv4 routing table
netstat -r -f

REM Show IPv6 routing table
netstat -r -6
```

### **nbtstat**
NetBIOS over TCP/IP statistics.

```cmd
REM Show local NetBIOS name table
nbtstat -n

REM Show NetBIOS name cache
nbtstat -c

REM Show NetBIOS sessions table
nbtstat -s

REM Show NetBIOS sessions by IP address
nbtstat -S

REM Reload NetBIOS name cache
nbtstat -R

REM Release and refresh NetBIOS names
nbtstat -RR

REM Show remote machine's NetBIOS name table
nbtstat -a computername
nbtstat -A ipaddress
```

## Advanced Network Commands

### **netsh**
Network Shell - comprehensive network configuration tool.

#### **Interface Configuration**
```cmd
REM Show all network interfaces
netsh interface show interface

REM Show IP configuration
netsh interface ip show config

REM Set static IP address
netsh interface ip set address "Local Area Connection" static 192.168.1.100 255.255.255.0 192.168.1.1

REM Set to DHCP
netsh interface ip set address "Local Area Connection" dhcp

REM Set DNS servers
netsh interface ip set dns "Local Area Connection" static 8.8.8.8
netsh interface ip add dns "Local Area Connection" 8.8.4.4 index=2

REM Enable/disable interface
netsh interface set interface "Local Area Connection" enabled
netsh interface set interface "Local Area Connection" disabled
```

#### **Wireless Configuration**
```cmd
REM Show wireless profiles
netsh wlan show profiles

REM Show wireless interface info
netsh wlan show interfaces

REM Connect to wireless network
netsh wlan connect name="NetworkName"

REM Disconnect from wireless
netsh wlan disconnect

REM Show available wireless networks
netsh wlan show availablenetworks

REM Export wireless profile
netsh wlan export profile name="NetworkName" folder=C:\Temp

REM Import wireless profile
netsh wlan add profile filename="C:\Temp\NetworkName.xml"
```

#### **Firewall Configuration**
```cmd
REM Show firewall status
netsh advfirewall show allprofiles

REM Enable/disable firewall
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles state off

REM Show firewall rules
netsh advfirewall firewall show rule all

REM Add firewall rule
netsh advfirewall firewall add rule name="Allow HTTP" dir=in action=allow protocol=TCP localport=80

REM Delete firewall rule
netsh advfirewall firewall delete rule name="Allow HTTP"

REM Reset firewall to default
netsh advfirewall reset
```

### **route**
Routing table management.

```cmd
REM Display routing table
route print

REM Add static route
route add 10.0.0.0 mask 255.255.255.0 192.168.1.1

REM Add persistent route
route -p add 10.0.0.0 mask 255.255.255.0 192.168.1.1

REM Delete route
route delete 10.0.0.0

REM Change existing route
route change 10.0.0.0 mask 255.255.255.0 192.168.1.2

REM Add default route
route add 0.0.0.0 mask 0.0.0.0 192.168.1.1

REM Show IPv6 routing table
route print -6
```

### **arp**
ARP table management.

```cmd
REM Display ARP table
arp -a

REM Display ARP entry for specific IP
arp -a 192.168.1.1

REM Add static ARP entry
arp -s 192.168.1.100 00-11-22-33-44-55

REM Delete ARP entry
arp -d 192.168.1.100

REM Delete all ARP entries
arp -d *
```

## DNS Commands

### **nslookup**
DNS lookup utility.

#### **Basic DNS Queries**
```cmd
REM Forward lookup
nslookup google.com

REM Reverse lookup
nslookup 8.8.8.8

REM Query specific DNS server
nslookup google.com 8.8.8.8

REM Interactive mode
nslookup
> set type=MX
> google.com
> exit
```

#### **Record Type Queries**
```cmd
REM Mail exchange records
nslookup -type=MX google.com

REM Name server records
nslookup -type=NS google.com

REM Text records
nslookup -type=TXT google.com

REM IPv6 addresses
nslookup -type=AAAA google.com

REM Start of authority
nslookup -type=SOA google.com
```

### **PowerShell DNS Cmdlets**
```powershell
# Resolve DNS name
Resolve-DnsName google.com

# Query specific record type
Resolve-DnsName google.com -Type MX
Resolve-DnsName google.com -Type AAAA

# Query specific DNS server
Resolve-DnsName google.com -Server 8.8.8.8

# Clear DNS cache
Clear-DnsClientCache

# Show DNS cache
Get-DnsClientCache
```

## PowerShell Network Cmdlets

### **Network Adapter Management**
```powershell
# Show network adapters
Get-NetAdapter

# Show specific adapter
Get-NetAdapter -Name "Ethernet"

# Enable/disable adapter
Enable-NetAdapter -Name "Ethernet"
Disable-NetAdapter -Name "Ethernet"

# Restart adapter
Restart-NetAdapter -Name "Ethernet"

# Show adapter statistics
Get-NetAdapterStatistics -Name "Ethernet"
```

### **IP Configuration**
```powershell
# Show IP configuration
Get-NetIPConfiguration

# Show IP addresses
Get-NetIPAddress

# Set static IP address
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.100 -PrefixLength 24 -DefaultGateway 192.168.1.1

# Remove IP address
Remove-NetIPAddress -IPAddress 192.168.1.100

# Set DNS servers
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 8.8.8.8,8.8.4.4

# Enable DHCP
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Enabled
```

### **Routing Management**
```powershell
# Show routing table
Get-NetRoute

# Add static route
New-NetRoute -DestinationPrefix 10.0.0.0/24 -NextHop 192.168.1.1 -InterfaceAlias "Ethernet"

# Remove route
Remove-NetRoute -DestinationPrefix 10.0.0.0/24

# Show default gateway
Get-NetRoute -DestinationPrefix 0.0.0.0/0
```

### **Network Connections**
```powershell
# Show TCP connections
Get-NetTCPConnection

# Show listening ports
Get-NetTCPConnection -State Listen

# Show UDP endpoints
Get-NetUDPEndpoint

# Show connections by process
Get-NetTCPConnection | Select-Object LocalAddress,LocalPort,RemoteAddress,RemotePort,State,@{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}}
```

## Network Testing and Troubleshooting

### **telnet**
Test TCP port connectivity.

```cmd
REM Install telnet client (if not already installed)
dism /online /Enable-Feature /FeatureName:TelnetClient

REM Test port connectivity
telnet google.com 80
telnet 192.168.1.1 23

REM Connect to specific port
telnet smtp.gmail.com 587
```

### **Test-NetConnection (PowerShell)**
Modern network connectivity testing.

```powershell
# Basic connectivity test
Test-NetConnection google.com

# Test specific port
Test-NetConnection google.com -Port 80
Test-NetConnection google.com -Port 443

# Test with detailed output
Test-NetConnection google.com -Port 80 -InformationLevel Detailed

# Traceroute functionality
Test-NetConnection google.com -TraceRoute

# Test multiple ports
80,443,993 | ForEach-Object { Test-NetConnection gmail.com -Port $_ }
```

### **Network Diagnostics**
```cmd
REM Windows Network Diagnostics
msdt.exe -id NetworkDiagnosticsDA

REM Network troubleshooter
msdt.exe -id NetworkDiagnosticsNetworkAdapter

REM Internet connection troubleshooter
msdt.exe -id NetworkDiagnosticsWeb
```

## Wireless Network Management

### **netsh wlan Commands**
```cmd
REM Show wireless adapter info
netsh wlan show interfaces

REM Show wireless profiles
netsh wlan show profiles

REM Show profile details
netsh wlan show profile name="ProfileName" key=clear

REM Scan for networks
netsh wlan show availablenetworks

REM Connect to network
netsh wlan connect name="NetworkName"

REM Disconnect
netsh wlan disconnect

REM Delete profile
netsh wlan delete profile name="NetworkName"

REM Show wireless settings
netsh wlan show settings
```

### **PowerShell Wireless Management**
```powershell
# Show wireless profiles
Get-NetConnectionProfile

# Show available networks
netsh wlan show availablenetworks

# Connect to wireless network (requires profile)
Connect-NetConnectionProfile -Name "ProfileName"
```

## Network Monitoring Tools

### **Performance Monitor (perfmon)**
```cmd
REM Open Performance Monitor
perfmon

REM Network-related counters to monitor:
REM - Network Interface\Bytes Total/sec
REM - Network Interface\Packets/sec
REM - Network Interface\Current Bandwidth
REM - TCP\Connections Established
REM - TCP\Segments/sec
```

### **Resource Monitor (resmon)**
```cmd
REM Open Resource Monitor
resmon

REM Network tab shows:
REM - Processes with network activity
REM - Network connections
REM - Listening ports
REM - TCP connections
```

### **Task Manager Network Monitoring**
```cmd
REM Open Task Manager
taskmgr

REM Performance tab -> Ethernet shows:
REM - Network utilization
REM - Link speed
REM - Send/Receive rates
```

## Network Configuration Files and Registry

### **Important Network Locations**
```cmd
REM Network adapter settings
control ncpa.cpl

REM Network and Sharing Center
control.exe /name Microsoft.NetworkAndSharingCenter

REM Advanced adapter settings
ncpa.cpl

REM Hosts file location
notepad C:\Windows\System32\drivers\etc\hosts

REM Network configuration backup
netsh dump > C:\network_config_backup.txt
```

### **Registry Locations**
```cmd
REM Network adapters
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Network

REM TCP/IP parameters
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters

REM Network interfaces
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces
```

## Troubleshooting Workflows

### **Basic Connectivity Troubleshooting**
```cmd
REM 1. Check adapter status
ipconfig /all

REM 2. Test loopback
ping 127.0.0.1

REM 3. Test default gateway
ping [gateway-ip]

REM 4. Test external connectivity
ping 8.8.8.8

REM 5. Test DNS resolution
nslookup google.com

REM 6. Trace route
tracert google.com

REM 7. Check for malware/firewall
netstat -an | findstr LISTENING
```

### **PowerShell Troubleshooting Script**
```powershell
# Network troubleshooting script
function Test-NetworkConnectivity {
    Write-Host "Testing Network Connectivity..." -ForegroundColor Yellow
    
    # Test adapter status
    $adapters = Get-NetAdapter | Where-Object {$_.Status -eq "Up"}
    Write-Host "Active Network Adapters:" -ForegroundColor Green
    $adapters | Format-Table Name, InterfaceDescription, LinkSpeed
    
    # Test IP configuration
    Write-Host "`nIP Configuration:" -ForegroundColor Green
    Get-NetIPConfiguration | Format-Table InterfaceAlias, IPv4Address, IPv4DefaultGateway
    
    # Test connectivity
    $tests = @(
        @{Name="Loopback"; Target="127.0.0.1"},
        @{Name="Gateway"; Target=(Get-NetRoute -DestinationPrefix "0.0.0.0/0").NextHop | Select-Object -First 1},
        @{Name="Google DNS"; Target="8.8.8.8"},
        @{Name="External Host"; Target="google.com"}
    )
    
    Write-Host "`nConnectivity Tests:" -ForegroundColor Green
    foreach ($test in $tests) {
        $result = Test-NetConnection $test.Target -WarningAction SilentlyContinue
        Write-Host "$($test.Name): $($result.PingSucceeded)" -ForegroundColor $(if($result.PingSucceeded){"Green"}else{"Red"})
    }
    
    # DNS test
    Write-Host "`nDNS Resolution Test:" -ForegroundColor Green
    try {
        $dns = Resolve-DnsName google.com -ErrorAction Stop
        Write-Host "DNS Resolution: Success" -ForegroundColor Green
    } catch {
        Write-Host "DNS Resolution: Failed" -ForegroundColor Red
    }
}

# Run the test
Test-NetworkConnectivity
```

### **Network Reset Commands**
```cmd
REM Reset network adapters
netsh int ip reset

REM Reset Winsock catalog
netsh winsock reset

REM Reset firewall
netsh advfirewall reset

REM Flush DNS
ipconfig /flushdns

REM Release and renew IP
ipconfig /release
ipconfig /renew

REM Reset network stack (requires reboot)
netsh int ip reset resetlog.txt
```

## Event Log Monitoring

### **Network-Related Event Logs**
```cmd
REM View System log for network events
eventvwr.msc

REM PowerShell event log queries
Get-WinEvent -LogName System | Where-Object {$_.Id -eq 20} | Select-Object -First 10

REM Network adapter events
Get-WinEvent -LogName System | Where-Object {$_.ProviderName -like "*network*"}

REM DHCP client events
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin"
```

### **Network Event IDs**
```
Common Network Event IDs:
- 20: Network adapter enabled
- 21: Network adapter disabled  
- 27: Network adapter connection established
- 4202: DHCP lease obtained
- 4201: DHCP lease renewal
- 1014: DNS resolution failure
- 1129: Network connectivity lost
```

## Security Commands

### **Network Security Monitoring**
```cmd
REM Show active connections with processes
netstat -ano | findstr ESTABLISHED

REM Show listening ports
netstat -an | findstr LISTENING

REM Check for suspicious processes
tasklist /svc

REM Monitor network activity
netstat -an 2
```

### **PowerShell Security Checks**
```powershell
# Check for suspicious network connections
Get-NetTCPConnection | Where-Object {$_.State -eq "Established" -and $_.RemoteAddress -notlike "192.168.*" -and $_.RemoteAddress -notlike "10.*" -and $_.RemoteAddress -ne "127.0.0.1"} | Select-Object LocalAddress,LocalPort,RemoteAddress,RemotePort,@{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}}

# Check firewall status
Get-NetFirewallProfile

# Show firewall rules
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"} | Select-Object DisplayName,Direction,Action,Profile
```

## Performance Optimization

### **Network Performance Tuning**
```cmd
REM Check network adapter properties
netsh int tcp show global

REM Enable/disable TCP features
netsh int tcp set global autotuninglevel=normal
netsh int tcp set global chimney=enabled
netsh int tcp set global rss=enabled

REM Check receive window size
netsh int tcp show global | findstr "Receive Window"

REM Network adapter power management
powercfg -devicequery wake_armed
```

### **PowerShell Performance Monitoring**
```powershell
# Monitor network utilization
Get-Counter "\Network Interface(*)\Bytes Total/sec" -SampleInterval 1 -MaxSamples 10

# Monitor specific adapter
Get-Counter "\Network Interface(Ethernet)\Bytes Total/sec" -Continuous

# Network performance summary
Get-NetAdapterStatistics | Format-Table Name,ReceivedBytes,SentBytes,ReceivedUnicastPackets,SentUnicastPackets
```

## Summary

Windows provides comprehensive command-line tools for network management, from basic connectivity testing to advanced configuration and troubleshooting. Understanding both traditional commands and modern PowerShell cmdlets is essential for effective Windows network administration.

**Key Command Categories:**
- **Basic Tools**: ipconfig, ping, tracert for fundamental network tasks
- **Advanced Tools**: netsh, PowerShell cmdlets for complex configuration
- **Monitoring**: netstat, perfmon, Get-NetTCPConnection for network monitoring
- **Troubleshooting**: pathping, Test-NetConnection for problem diagnosis

**Best Practices:**
- **Use PowerShell**: Modern PowerShell cmdlets provide more functionality
- **Document Changes**: Record network configuration modifications
- **Test Thoroughly**: Validate network changes before deploying
- **Monitor Regularly**: Establish baseline network performance metrics

**Essential Skills:**
- **Basic Connectivity**: Master ping, tracert, ipconfig for troubleshooting
- **Configuration**: Understand netsh and PowerShell for network setup
- **Monitoring**: Use netstat and PowerShell for network monitoring
- **Security**: Implement and monitor network security settings

These Windows network commands are essential for administrators managing Windows-based networks and provide the foundation for effective network troubleshooting and management.