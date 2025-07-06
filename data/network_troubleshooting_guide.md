# Network Troubleshooting Guide

## Introduction

Network troubleshooting is the process of identifying, diagnosing, and resolving network connectivity issues. This guide covers common network problems and their solutions using systematic approaches.

## Basic Network Troubleshooting Steps

### 1. Physical Layer Check
Always start with the physical layer when troubleshooting network issues:
- Check cable connections
- Verify power to network devices
- Look for damaged cables or loose connections
- Check LED indicators on network equipment

### 2. IP Configuration Verification
Verify the IP configuration on the affected device:
```bash
# Windows
ipconfig /all

# Linux/Mac
ifconfig
ip addr show
```

Common IP configuration issues:
- Incorrect IP address assignment
- Wrong subnet mask
- Missing or incorrect default gateway
- DNS server configuration problems

### 3. Connectivity Testing
Use ping to test connectivity at different levels:
```bash
# Test local loopback
ping 127.0.0.1

# Test default gateway
ping [gateway_ip]

# Test external connectivity
ping 8.8.8.8

# Test DNS resolution
ping google.com
```

## Common Network Problems and Solutions

### Problem: No Internet Connectivity

**Symptoms:**
- Cannot access websites
- Email not working
- Applications cannot connect to internet services

**Troubleshooting Steps:**
1. Check physical connections
2. Verify IP configuration using `ipconfig` or `ifconfig`
3. Test connectivity to default gateway
4. Test DNS resolution
5. Check firewall settings

**Common Solutions:**
- Restart network adapter
- Renew IP address: `ipconfig /renew` (Windows) or `dhclient` (Linux)
- Flush DNS cache: `ipconfig /flushdns` (Windows) or `sudo dscacheutil -flushcache` (Mac)
- Reset network settings

### Problem: Slow Network Performance

**Symptoms:**
- Web pages load slowly
- File transfers take longer than expected
- Video streaming buffers frequently

**Troubleshooting Steps:**
1. Test network speed using speed test tools
2. Check for network congestion
3. Monitor bandwidth usage
4. Verify Quality of Service (QoS) settings
5. Check for malware or unwanted applications

**Solutions:**
- Upgrade network hardware
- Implement QoS policies
- Optimize network configuration
- Remove bandwidth-consuming applications

### Problem: Intermittent Connectivity

**Symptoms:**
- Connection drops randomly
- Inconsistent network performance
- Applications timeout sporadically

**Troubleshooting Steps:**
1. Check cable integrity
2. Monitor network logs
3. Test with different devices
4. Check for interference (wireless networks)
5. Verify power supply stability

**Solutions:**
- Replace faulty cables
- Update network drivers
- Adjust wireless channel settings
- Implement redundant connections

## Advanced Troubleshooting Techniques

### Network Monitoring Tools

**Wireshark**
- Packet capture and analysis
- Protocol analysis
- Network performance monitoring

**Netstat**
```bash
# Display active connections
netstat -an

# Show routing table
netstat -rn

# Display network statistics
netstat -s
```

**Traceroute/Tracert**
```bash
# Windows
tracert google.com

# Linux/Mac
traceroute google.com
```

### DNS Troubleshooting

**Common DNS Issues:**
- DNS server not responding
- Incorrect DNS configuration
- DNS cache corruption

**DNS Testing Commands:**
```bash
# Windows
nslookup google.com
nslookup google.com 8.8.8.8

# Linux/Mac
dig google.com
dig @8.8.8.8 google.com
```

**Solutions:**
- Change DNS servers (8.8.8.8, 1.1.1.1)
- Clear DNS cache
- Check DNS server configuration

### DHCP Troubleshooting

**Common DHCP Issues:**
- DHCP server not responding
- IP address conflicts
- Incorrect DHCP scope configuration

**DHCP Commands:**
```bash
# Windows
ipconfig /release
ipconfig /renew

# Linux
sudo dhclient -r
sudo dhclient
```

## Wireless Network Troubleshooting

### Common Wireless Issues

**Signal Strength Problems:**
- Weak signal strength
- Interference from other devices
- Physical obstacles blocking signal

**Solutions:**
- Reposition wireless access points
- Change wireless channels
- Upgrade to newer wireless standards
- Use wireless repeaters or mesh networks

**Authentication Issues:**
- Incorrect wireless password
- Security protocol mismatch
- MAC address filtering

**Solutions:**
- Verify wireless credentials
- Check security settings
- Update wireless drivers
- Reset wireless configuration

## Network Security Troubleshooting

### Firewall Issues

**Common Problems:**
- Blocked ports preventing communication
- Overly restrictive firewall rules
- Firewall software conflicts

**Solutions:**
- Review firewall logs
- Create specific allow rules
- Temporarily disable firewall for testing
- Update firewall software

### VPN Troubleshooting

**Common VPN Issues:**
- Cannot establish VPN connection
- VPN connection drops frequently
- Slow performance over VPN

**Solutions:**
- Check VPN server status
- Verify VPN credentials
- Test different VPN protocols
- Check firewall settings for VPN traffic

## Corporate Network Troubleshooting

### Desktop User Troubleshooting (Corporate Environment)

#### Domain Connectivity Issues

**Symptoms:**
- Cannot log into domain
- Group Policy not applying
- Cannot access shared resources
- "Trust relationship failed" errors

**Desktop Troubleshooting Commands:**
```cmd
# Check domain controller connectivity
ping domain-controller.company.com
nslookup domain-controller.company.com

# Test domain trust relationship
nltest /test:domain-controller.company.com

# Check computer account status
nltest /sc_query:DOMAIN

# Reset computer account (requires admin rights)
netdom resetpwd /s:domain-controller.company.com /ud:DOMAIN\admin /pd:*

# Force Group Policy update
gpupdate /force

# Check applied Group Policies
gpresult /r
gpresult /h gpresult.html
```

#### Corporate WiFi Authentication Issues

**802.1X Authentication Problems:**
```cmd
# Check wireless profiles
netsh wlan show profiles

# Show specific profile details
netsh wlan show profile name="Corporate-WiFi" key=clear

# Delete and recreate wireless profile
netsh wlan delete profile name="Corporate-WiFi"

# Export wireless profile
netsh wlan export profile name="Corporate-WiFi" folder=C:\temp

# Check certificate store for authentication
certlm.msc
```

#### Network Drive Mapping Issues

**Troubleshooting Shared Resources:**
```cmd
# List current network drives
net use

# Map network drive with credentials
net use Z: \\server\share /user:DOMAIN\username password

# Delete network drive mapping
net use Z: /delete

# Test SMB connectivity
telnet server-name 445

# Check SMB protocol version
Get-SmbConnection (PowerShell)
```

#### Corporate Proxy Issues

**Proxy Configuration Troubleshooting:**
```cmd
# Check proxy settings via registry
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"

# Test proxy connectivity
curl -x proxy-server:8080 http://www.google.com

# Bypass proxy for specific sites
netsh winhttp set proxy proxy-server:8080 bypass-list="*.company.com;localhost"

# Reset proxy settings
netsh winhttp reset proxy
```

### Wireless Controller Troubleshooting

#### Cisco Wireless LAN Controller (WLC) Commands

**Basic Status and Configuration:**
```cisco
# Show controller summary
show summary

# Show all access points
show ap summary

# Show specific AP details
show ap config general AP-NAME

# Show wireless client summary
show client summary

# Show specific client details
show client detail CLIENT-MAC

# Show WLAN configuration
show wlan summary
show wlan id WLAN-ID
```

**Access Point Troubleshooting:**
```cisco
# Reboot specific access point
config ap reset AP-NAME

# Show AP join statistics
show ap join stats summary
show ap join stats detailed AP-NAME

# Check AP image version
show ap image all

# Upgrade AP image
config ap image predownload primary all
config ap image swap AP-NAME

# Show AP crash info
show ap crash-file
```

**Client Connectivity Issues:**
```cisco
# Debug client association
debug client CLIENT-MAC

# Show client authentication details
show client detail CLIENT-MAC

# Show client roaming history
show client roam-history CLIENT-MAC

# Deauthenticate problematic client
config client deauthenticate CLIENT-MAC

# Show client statistics
show client stats CLIENT-MAC
```

**RADIUS Authentication Troubleshooting:**
```cisco
# Show RADIUS server status
show radius summary
show radius statistics

# Test RADIUS connectivity
test radius username PASSWORD server-ip SERVER-IP

# Debug RADIUS authentication
debug aaa radius enable
debug aaa events enable

# Show AAA statistics
show aaa auth
show aaa local statistics
```

#### Aruba Wireless Controller Commands

**Basic System Information:**
```aruba
# Show system information
show version
show inventory
show clock

# Show access point status
show ap active
show ap database

# Show user associations
show user
show user-table
```

**Access Point Management:**
```aruba
# Reboot access point
ap reboot name AP-NAME

# Show AP configuration
show ap details name AP-NAME

# Show AP radio status
show ap radio-summary

# Provision new AP
ap provision AP-MAC group GROUP-NAME
```

**Wireless Client Troubleshooting:**
```aruba
# Show client details
show user username USERNAME
show user ip-address IP-ADDRESS

# Show authentication logs
show log security 100

# Show DHCP bindings
show ip dhcp binding

# Clear user session
clear user username USERNAME
```

#### Ubiquiti UniFi Controller Commands

**SSH Commands for UniFi Devices:**
```bash
# Connect to UniFi device
ssh ubnt@AP-IP-ADDRESS

# Show system information
info
mca-cli

# Show wireless status
iwconfig
wlanconfig ath0 list

# Show connected clients
wlanconfig ath0 list sta

# Restart wireless interface
ifconfig ath0 down
ifconfig ath0 up

# Show system logs
tail -f /var/log/messages
```

### Corporate Network Monitoring

#### SNMP Monitoring Commands

**Basic SNMP Queries:**
```bash
# Test SNMP connectivity
snmpwalk -v2c -c public DEVICE-IP system

# Get interface statistics
snmpwalk -v2c -c public DEVICE-IP 1.3.6.1.2.1.2.2.1.10

# Monitor bandwidth utilization
snmpget -v2c -c public DEVICE-IP 1.3.6.1.2.1.2.2.1.10.1
```

#### Network Performance Monitoring

**Bandwidth Testing:**
```bash
# iPerf bandwidth testing
# Server side
iperf3 -s

# Client side
iperf3 -c SERVER-IP -t 60 -i 5

# UDP testing
iperf3 -c SERVER-IP -u -b 100M
```

**Network Latency Monitoring:**
```bash
# Continuous ping with timestamps
ping -t DESTINATION-IP | ForEach {"$(Get-Date -format 'yyyy-MM-dd HH:mm:ss'): $_"}

# MTR (My Traceroute) for path analysis
mtr --report --report-cycles 100 DESTINATION-IP
```

### Enterprise Security Troubleshooting

#### Certificate-Based Authentication

**Certificate Troubleshooting:**
```cmd
# Check certificate store
certlm.msc
certmgr.msc

# Export certificate
certutil -exportpfx -p password My CERT-THUMBPRINT cert.pfx

# Verify certificate chain
certutil -verify -urlfetch CERTIFICATE-FILE

# Check certificate revocation
certutil -url CERTIFICATE-FILE
```

#### Network Access Control (NAC)

**802.1X Troubleshooting:**
```cmd
# Check 802.1X status
netsh lan show profiles
netsh lan show settings

# Enable 802.1X logging
netsh wlan set tracing mode=yes tracefile=c:\temp\wireless.etl

# View 802.1X events
eventvwr.msc (Windows Logs > System)
```

## Best Practices for Network Troubleshooting

### Documentation
- Keep network diagrams updated
- Document all network changes
- Maintain inventory of network equipment
- Record troubleshooting procedures

### Monitoring
- Implement network monitoring tools
- Set up alerts for critical issues
- Regular performance baseline testing
- Monitor network traffic patterns

### Preventive Measures
- Regular firmware updates
- Scheduled maintenance windows
- Backup configurations
- Redundancy planning

## Conclusion

Effective network troubleshooting requires a systematic approach, starting with basic connectivity tests and progressing to more advanced diagnostic techniques. Always document your findings and solutions to build a knowledge base for future reference.

Remember to:
1. Start with the basics (physical layer)
2. Use appropriate tools for diagnosis
3. Test one change at a time
4. Document your process and solutions
5. Implement preventive measures

Regular maintenance and monitoring can prevent many network issues before they impact users.