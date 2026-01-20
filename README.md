# VLAN-Segmentation-Zone-Firewall-Lab
Emulates a router-on-stick topology using VyOS for routing/firewall and Open vSwitch for VLANs, ARP, RSTP, and traffic flow across multiple departmental VLANs (IT, Help Desk, Sales, HR). Includes Alpine VM for secure management, subnetting, DHCP, and zone-based firewall rules.

## Project Overview
This project emulates an enterprise-style network topology built in GNS3, using open-source networking and security technologies.
The lab focuses on Layer 2 segmentation, Layer 3 routing, redundancy, and security enforcement through zone-based firewalling.

The topology uses:

## VyOS as a router-on-a-stick solution, providing:
- Inter-VLAN routing
- DHCP services
- Stateful firewalling
- Zone-based firewall policies

## Open vSwitch (OVS) as Layer 2 switches to handle:
- VLANs
- ARP
- RSTP
- Traffic forwarding

## VPCs to simulate multiple enterprise departments
## Alpine Linux as a dedicated management (MGMT) jump host, isolated from user VLANs

A dedicated Management VLAN and zone is implemented to securely manage infrastructure devices and reduce lateral movement risks.

## Objectives

- Design a segmented enterprise network using VLANs
- Implement RSTP with manual root bridge selection
- Configure router-on-a-stick inter-VLAN routing
- Provide DHCP services per VLAN
- Analyze normal network behavior using Wireshark
- Implement both:
    + Traditional firewall rules
    + Zone-based firewall policies
- Isolate infrastructure management using a MGMT zone and jump host
- Apply stateful packet filtering and SSH hardening

## Network Segmentation & Subnetting
Subnetting was planned from the ground up to reflect realistic enterprise needs, allocating IP space based on departmental size and purpose.
| VLAN    | Department | Subnet          | Usable IPs |
| ------- | ---------- | --------------- | ---------- |
| VLAN 10 | IT         | 10.0.10.0/27    | 30         |
| VLAN 11 | Help Desk  | 10.0.11.0/27    | 30         |
| VLAN 20 | Sales      | 192.168.20.0/25 | 126        |
| VLAN 30 | HR         | 192.168.30.0/25 | 126        |
| VLAN 99 | Management | 10.0.99.0/28    | 14         |

## Layer 2 Design (Open vSwitch)
- Each Open vSwitch instance was configured with:
    + Proper VLAN tagging per interface
    + Access and trunk ports depending on traffic direction
    + RSTP enabled and tuned manually
- RSTP priorities were statically assigned to ensure deterministic behavior:
    + Root Bridge: priority 4096
    + Secondary Root: priority 8192
    + Tertiary Switch: priority 12288
- This ensures:
    + The switch connected to the router always acts as the root bridge
    + Redundant paths remain available
    + Backup links transition correctly to Alternate state when required
 
## Layer 3 Design (VyOS)
VyOS was configured as a router-on-a-stick using subinterfaces on eth1, one per VLAN.

- Key components:
    + Inter-VLAN routing
    + DHCP server configured per VLAN
    + Default gateways assigned via subinterfaces
 
- Each VLAN was tested by:
    + Requesting DHCP leases from multiple VPCs
    + Verifying gateway reachability
    + Testing inter-VLAN communication where permitted
 
## Traffic Analysis
- During normal operation, traffic was captured and analyzed using Wireshark to observe:
    + RSTP convergence behavior
    + ARP resolution
    + DHCP Discover / Offer / Request / ACK
    + ICMP flows between VLANs

This validated that the network was operating in a healthy and expected state before introducing security controls.

## Firewalling & Zone-Based Security
The project first implemented classic firewall rules to validate filtering logic, then migrated entirely to zone-based firewalling for better scalability and clarity.

- Key characteristics:
    + Stateful packet inspection using global ESTABLISHED / RELATED policies
    + Explicit zone-to-zone traffic control
    + Default deny behavior
    + Dedicated protection of the router control plane (LOCAL zone)
 
## Management Plane Isolation
A dedicated Management VLAN (VLAN 99) and ZONE_MGMT were implemented.

- An Alpine Linux VM acts as a management jump host, used exclusively to:
    + SSH into the VyOS router
    + Manage Open vSwitch devices
    + Security enhancements include:
    + SSH key-based authentication
    + Management access restricted to the MGMT zone
    + User VLANs denied access to infrastructure control planes

This design reduces the impact of potential compromises in user VLANs and limits lateral movement.

## SSH Hardening Notes
Password authentication was disabled in favor of SSH key pairs

Although the default SSH port (22) is used, changing it can add an additional layer of security by modifying:
  + VyOS SSH service configuration
  + sshd_config on the management VM

Something to have in mind is that to test ICMP packets on each VLAN from each VPC to the defaul gateway, a general ICMP rules was added at the router but this was knowing this is for a lab only. In real life that general rule should be avoided or tailored to specific devices such as the management console ot the IT department.

## What This Lab Demonstrates
- Enterprise-style VLAN and subnet planning
- Redundant Layer 2 design with RSTP
- Secure inter-VLAN routing
- Zone-based firewall architecture
- Control-plane and management-plane protection
- Realistic troubleshooting and validation methodology

## Lab Topology
*Img 1: Topology*<br>
<img width="1393" height="753" alt="final-topology" src="https://github.com/user-attachments/assets/2144bf59-a65a-4392-ae2f-70c6233d5d5a" />

