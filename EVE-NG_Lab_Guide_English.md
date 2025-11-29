# EVE-NG Lab Design Guide - Multi-Segment Network

## Overview

This guide provides comprehensive documentation for designing and implementing a small enterprise multi-segment network using the **EVE-NG** platform. The project focuses on applying fundamental concepts in building enterprise networks, such as hierarchical architecture, advanced routing protocols, network segmentation using VLANs, and ensuring network continuity through resilience techniques.

---

## Learning Objectives

This project aims to achieve the following learning objectives:

1. **Understand hierarchical network architecture** (Core, Distribution, Access)
2. **Implement OSPF routing protocol** in a multi-area environment
3. **Create VLANs** to isolate traffic between users and servers
4. **Test network resilience** through redundant links using EtherChannel
5. **Build a lab environment** ready for expansion and integration of future projects such as Firewalls or Network Access Control (NAC) systems

---

## Network Topology Diagram

<img width="1919" height="947" alt="EVE-NG_Multi-Segment_Network_Lab" src="https://github.com/user-attachments/assets/e70c7948-74a4-47e9-b318-164d388ca53e" />

The diagram above shows the complete network structure with all devices, connections, and IP addresses.

---

## Devices and Components Table

| Device | Type | Primary Function | Notes |
|---|---|---|---|
| **R1** | Router | Core Router (Main Branch) | Operates in OSPF Area 0 and connects to other branches |
| **R2** | Router | Branch Router (Secondary Branch) | Connects to main network via WAN and operates in OSPF Area 1 |
| **SW-Core1** | L3 Switch | Distribution/Core Switch | Performs Inter-VLAN Routing and serves as aggregation point for Access Switches |
| **SW-Access1** | L2/L3 Switch | Access Switch | Connects end devices to the network and manages their VLANs |
| **Server1** | Server | DHCP/DNS Server | Provides DHCP and Domain Name System (DNS) services |
| **Server2** | Server | Web/Backup Server | Web server and backup server |
| **PC1** | End Device | User Devices | In VLAN 30 (Sales) for connectivity and isolation testing |
| **PC2** | End Device | User Devices | In VLAN 40 (HR) for connectivity and isolation testing |
| **PC3** | End Device | User Devices | Additional device for network testing |

---

## IP Addressing Plan

### VLANs & Subnets Table

| VLAN ID | Network Name | Description | Network Address | Subnet Mask | Default Gateway | DHCP Range |
|---|---|---|---|---|---|---|
| **10** | Management | Device management network | 192.168.10.0 | 255.255.255.0 | 192.168.10.1 | - |
| **20** | Servers | Server network | 192.168.20.0 | 255.255.255.0 | 192.168.20.1 | - |
| **30** | Users-Sales | Sales department user network | 192.168.30.0 | 255.255.255.0 | 192.168.30.1 | 192.168.30.100-200 |
| **40** | Users-HR | HR department user network | 192.168.40.0 | 255.255.255.0 | 192.168.40.1 | 192.168.40.100-200 |
| **N/A** | WAN Link | Link between R1 and R2 | 10.0.0.0 | 255.255.255.252 | - | - |

### Device IP Addresses Table

| Device | Interface | IP Address | Subnet Mask | Description |
|---|---|---|---|---|
| **R1** | GigabitEthernet0/0 | 192.168.10.254 | 255.255.255.0 | Connection to SW-Core1 |
| **R1** | Serial0/0 | 10.0.0.1 | 255.255.255.252 | WAN Link to R2 |
| **R2** | GigabitEthernet0/0 | 192.168.50.254 | 255.255.255.0 | Connection to Branch Network |
| **R2** | Serial0/0 | 10.0.0.2 | 255.255.255.252 | WAN Link to R1 |
| **SW-Core1** | VLAN 10 (SVI) | 192.168.10.1 | 255.255.255.0 | Management network gateway |
| **SW-Core1** | VLAN 20 (SVI) | 192.168.20.1 | 255.255.255.0 | Server network gateway |
| **SW-Core1** | VLAN 30 (SVI) | 192.168.30.1 | 255.255.255.0 | Sales network gateway |
| **SW-Core1** | VLAN 40 (SVI) | 192.168.40.1 | 255.255.255.0 | HR network gateway |
| **SW-Core1** | GigabitEthernet0/0 | 192.168.10.2 | 255.255.255.0 | Connection to R1 |
| **Server1** | eth0 | 192.168.20.10 | 255.255.255.0 | DHCP/DNS Server |
| **Server2** | eth0 | 192.168.20.20 | 255.255.255.0 | Web and Backup Server |
| **PC1** | eth0 | DHCP | - | Sales department device (VLAN 30) |
| **PC2** | eth0 | DHCP | - | HR department device (VLAN 40) |
| **PC3** | eth0 | DHCP or Static | - | Additional testing device |

---

## Complete Device Configurations

### 1. R1 Configuration – Core Router

```
enable
configure terminal
!
hostname R1
!
! === Interface Configuration ===
interface GigabitEthernet0/0
 description Link to SW-Core1
 ip address 192.168.10.254 255.255.255.0
 no shutdown
!
interface Serial0/0
 description WAN Link to R2
 ip address 10.0.0.1 255.255.255.252
 clock rate 128000
 no shutdown
!
! === OSPF Configuration ===
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
!
end
write memory
```

**Command Explanation:**
- `hostname R1`: Set device hostname
- `ip address 192.168.10.254 255.255.255.0`: Assign IP address to interface
- `clock rate 128000`: Set clock rate for Serial interface (required on DCE side)
- `router ospf 1`: Enable OSPF protocol with process ID 1
- `router-id 1.1.1.1`: Assign unique router ID for OSPF
- `network 192.168.10.0 0.0.0.255 area 0`: Advertise network in OSPF Area 0

---

### 2. R2 Configuration – Branch Router

```
enable
configure terminal
!
hostname R2
!
! === Interface Configuration ===
interface GigabitEthernet0/0
 description Link to Branch Network
 ip address 192.168.50.254 255.255.255.0
 no shutdown
!
interface Serial0/0
 description WAN Link to R1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
!
! === OSPF Configuration ===
router ospf 1
 router-id 2.2.2.2
 network 192.168.50.0 0.0.0.255 area 1
 network 10.0.0.0 0.0.0.3 area 0
!
end
write memory
```

**Command Explanation:**
- `network 192.168.50.0 0.0.0.255 area 1`: Advertise branch network in OSPF Area 1
- `network 10.0.0.0 0.0.0.3 area 0`: Advertise WAN link in OSPF Area 0

---

### 3. SW-Core1 Configuration – Distribution/Core Switch

```
enable
configure terminal
!
hostname SW-Core1
!
! === Enable Routing ===
ip routing
!
! === Create VLANs ===
vlan 10
 name Management
vlan 20
 name Servers
vlan 30
 name Users-Sales
vlan 40
 name Users-HR
!
! === Configure SVIs for Inter-VLAN Routing ===
interface Vlan10
 description Management VLAN
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
interface Vlan20
 description Servers VLAN
 ip address 192.168.20.1 255.255.255.0
 no shutdown
!
interface Vlan30
 description Users-Sales VLAN
 ip address 192.168.30.1 255.255.255.0
 no shutdown
!
interface Vlan40
 description Users-HR VLAN
 ip address 192.168.40.1 255.255.255.0
 no shutdown
!
! === Connection to R1 ===
interface GigabitEthernet0/0
 description Link to R1
 no switchport
 ip address 192.168.10.2 255.255.255.0
 no shutdown
!
! === EtherChannel Configuration ===
interface Port-channel1
 description EtherChannel to SW-Access1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
!
interface range GigabitEthernet0/1 - 2
 description Member of Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
 channel-group 1 mode desirable
 no shutdown
!
! === OSPF Configuration ===
router ospf 10
 router-id 3.3.3.3
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
!
end
write memory
```

**Command Explanation:**
- `ip routing`: Enable routing on L3 switch
- `interface Vlan10`: Create Switch Virtual Interface (SVI) for VLAN 10
- `no switchport`: Convert port from switch mode to router mode
- `switchport trunk encapsulation dot1q`: Specify trunk encapsulation type
- `channel-group 1 mode desirable`: Add interface to Port-channel using PAgP

---

### 4. SW-Access1 Configuration – Access Switch

```
enable
configure terminal
!
hostname SW-Access1
!
! === Create VLANs ===
vlan 20
 name Servers
vlan 30
 name Users-Sales
vlan 40
 name Users-HR
!
! === EtherChannel Configuration ===
interface Port-channel1
 description EtherChannel to SW-Core1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
!
interface range GigabitEthernet0/0 - 1
 description Member of Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
 channel-group 1 mode desirable
 no shutdown
!
! === Access Ports for End Devices ===
interface GigabitEthernet0/2
 description To PC1 (Sales)
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/3
 description To PC2 (HR)
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet1/0
 description To PC3
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
!
end
write memory
```

**Command Explanation:**
- `switchport mode access`: Set port as access port
- `switchport access vlan 30`: Assign port to VLAN 30
- `spanning-tree portfast`: Enable PortFast to speed up connection

---

### 5. Server1 Configuration – DHCP/DNS Server (Linux)

#### Network Interface Configuration:
```bash
sudo ip addr add 192.168.20.10/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 192.168.20.1
```

#### Install and Configure DHCP Server:
```bash
sudo apt-get update
sudo apt-get install -y isc-dhcp-server
```

#### `/etc/dhcp/dhcpd.conf` File Content:
```
# DHCP Configuration for VLAN 30 - Sales
subnet 192.168.30.0 netmask 255.255.255.0 {
  range 192.168.30.100 192.168.30.200;
  option routers 192.168.30.1;
  option domain-name-servers 192.168.20.10;
  option domain-name "lab.local";
  default-lease-time 600;
  max-lease-time 7200;
}

# DHCP Configuration for VLAN 40 - HR
subnet 192.168.40.0 netmask 255.255.255.0 {
  range 192.168.40.100 192.168.40.200;
  option routers 192.168.40.1;
  option domain-name-servers 192.168.20.10;
  option domain-name "lab.local";
  default-lease-time 600;
  max-lease-time 7200;
}
```

#### Start DHCP Service:
```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

#### Install and Configure DNS Server (BIND9):
```bash
sudo apt-get install -y bind9 bind9utils bind9-doc
sudo systemctl start bind9
sudo systemctl enable bind9
```

---

### 6. Server2 Configuration – Web/Backup Server (Linux)

#### Network Interface Configuration:
```bash
sudo ip addr add 192.168.20.20/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 192.168.20.1
```

#### Install and Configure Apache Web Server:
```bash
sudo apt-get update
sudo apt-get install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2
```

#### Create Test Web Page:
```bash
echo "<h1>Welcome to Lab Network - Server2</h1>" | sudo tee /var/www/html/index.html
```

---

## Verification and Testing Steps

### 1. Basic Connectivity Testing (Ping Tests)

#### From R1:
```
R1# ping 10.0.0.2
R1# ping 192.168.10.1
R1# ping 192.168.20.10
R1# ping 192.168.50.254
```

#### From R2:
```
R2# ping 10.0.0.1
R2# ping 192.168.10.1
R2# ping 192.168.20.10
```

#### From PC1 (VLAN 30):
```bash
ping 192.168.40.1
ping 192.168.20.20
ping 192.168.30.1
ping 8.8.8.8
```

#### From PC2 (VLAN 40):
```bash
ping 192.168.30.1
ping 192.168.20.10
ping 192.168.40.1
```

---

### 2. OSPF Verification

#### On R1:
```
R1# show ip ospf neighbor
R1# show ip route ospf
R1# show ip ospf interface brief
R1# show ip ospf database
```

**Expected Result:**
- Neighbor relationship should be established with R2 and SW-Core1
- Routes from Area 1 (192.168.50.0/24) should appear in routing table

#### On R2:
```
R2# show ip ospf neighbor
R2# show ip route ospf
R2# show ip protocols
```

#### On SW-Core1:
```
SW-Core1# show ip ospf neighbor
SW-Core1# show ip route ospf
SW-Core1# show ip protocols
```

---

### 3. EtherChannel Verification

#### On SW-Core1:
```
SW-Core1# show etherchannel summary
SW-Core1# show etherchannel port-channel
SW-Core1# show interfaces port-channel 1
SW-Core1# show interfaces trunk
```

**Expected Result:**
```
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Gi0/1(P)    Gi0/2(P)
```

#### On SW-Access1:
```
SW-Access1# show etherchannel summary
SW-Access1# show interfaces port-channel 1
```

---

### 4. VLAN Verification

#### On SW-Core1:
```
SW-Core1# show vlan brief
SW-Core1# show interfaces trunk
SW-Core1# show ip interface brief
```

#### On SW-Access1:
```
SW-Access1# show vlan brief
SW-Access1# show interfaces trunk
SW-Access1# show interfaces status
```

---

### 5. DHCP Verification

#### On PC1, PC2, and PC3:

**On Linux:**
```bash
sudo dhclient -r eth0
sudo dhclient -v eth0
ip addr show eth0
ip route show
cat /etc/resolv.conf
```

**On Windows:**
```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

**Expected Result:**
- PC1 should receive IP address from range `192.168.30.100-200`
- PC2 should receive IP address from range `192.168.40.100-200`
- Default Gateway should be correctly assigned
- DNS server should be `192.168.20.10`

#### On Server1 (to verify DHCP operations):
```bash
sudo tail -f /var/log/syslog | grep dhcpd
sudo journalctl -u isc-dhcp-server -f
```

---

### 6. Web Server Verification

#### From PC1, PC2, or PC3:
```bash
curl http://192.168.20.20
# Or from browser
firefox http://192.168.20.20
```

**Expected Result:**
- Test web page should be displayed

---

## EVE-NG Implementation Notes

### Device Requirements in EVE-NG:

| Device Type | Recommended Image | Notes |
|---|---|---|
| **Routers (R1, R2)** | vIOS L3 / IOL | `vios-adventerprisek9-m` |
| **L3 Switch (SW-Core1)** | IOL L3 | `i86bi-linux-l3-adventerprisek9` |
| **L2 Switch (SW-Access1)** | IOL L2/L3 | `i86bi-linux-l2-adventerprisek9` |
| **Servers** | Linux | Ubuntu Server 20.04 or Debian |
| **PCs** | VPCS or Linux | TinyCore Linux or VPCS |

### Setup Steps in EVE-NG:

1. Create new Lab in EVE-NG
2. Add devices according to topology diagram
3. Connect devices using network cables
4. Start all devices
5. Apply configurations mentioned above to each device
6. Save configurations using `write memory`
7. Perform verification tests

---

## Troubleshooting

### Issue: OSPF Neighbor Not Showing

**Solution:**
```
! Verify Area ID match
show ip ospf interface

! Verify basic connectivity
ping [neighbor-ip]

! Verify OSPF configuration
show running-config | section ospf
```

### Issue: EtherChannel Not Working

**Solution:**
```
! Verify matching settings on both sides
show etherchannel summary

! Verify trunk mode
show interfaces trunk

! Reconfigure EtherChannel
no interface port-channel 1
interface range gi0/1-2
 no channel-group 1
 shutdown
 no shutdown
 channel-group 1 mode desirable
```

### Issue: DHCP Not Working

**Solution:**
```bash
# On Server1
sudo systemctl status isc-dhcp-server
sudo journalctl -u isc-dhcp-server -n 50

# Verify configuration
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

# On PC
sudo dhclient -v eth0
```

---

## Conclusion

This project provides a solid foundation for building a reliable and scalable enterprise network. By implementing OSPF, VLANs, and EtherChannel, we have created a lab environment that reflects best practices in network design. This lab can be used as a starting point for advanced future projects, such as:

- Adding Firewalls
- Implementing Intrusion Detection Systems (IDS/IPS)
- Creating VPN networks
- Implementing Quality of Service (QoS) techniques
- Adding additional routing protocols (BGP, EIGRP)

These files were prepared using artificial intelligence tools [Manus AI and Claude AI], with ideas and content developed and wording edited by [Bandar Amer].


