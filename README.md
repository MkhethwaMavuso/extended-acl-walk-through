# extended-acl-walk-through
This repository documents the architectural design, configuration, and verification of an enterprise network featuring Multi-Area OSPF Dynamic Routing, Router-on-a-Stick (ROAS) VLAN routing, and Extended Access Control Lists (ACLs) for fine-grained security enforcement.

📋 OverviewThis repository documents the architectural design, configuration, and verification of an enterprise network featuring Multi-Area OSPF Dynamic Routing, Router-on-a-Stick (ROAS) VLAN routing, and Extended Access Control Lists (ACLs) for fine-grained security enforcement.The topology simulates an end-to-end enterprise environment connected via a core transit backbone across WAN links, isolating corporate departments while providing controlled access to internal application servers and external cloud resources.📐 Network Topology & Diagram AnalysisTopology BreakdownClient Access Layer (Branch Office / Campus Left)Switch1 (Cisco 2960): Handles Layer 2 access for client subnets.VLAN 10 (IT Department):Subnet: 192.168.10.0/24Devices: PC0 (192.168.10.2), PC1 (192.168.10.3)VLAN 20 (HR Department):Subnet: 192.168.20.0/24Devices: PC2 (192.168.20.2)Inter-VLAN Routing: Configured on R1 via Router-on-a-Stick (R-O-A-S) utilizing 802.1Q sub-interfaces.Core Transit & ISP Boundary (Center)Router R1 & Router R2 (Cisco 2911): Inter-connected via serial/WAN transit link simulating an IP backbone domain.R1 & R2 as ABRs: Handle inter-area topology synthesis and summarize intra-area routes across Area boundaries.Server Farm & Data Center (Right)Switch0 (Cisco 2960): Acts as the broadcast multi-access switch interconnecting internal departmental servers.Internal Servers:IT-SRV: 10.0.0.3HR-SRV: 10.0.0.2External Server / Web Infrastructure:WEB-SRV (Internet Service): 8.8.8.8 (Public DNS / Cloud Web Server connected directly to R2 edge)🌐 Quick Overview & Network Specs🟢 Architecture & Topology: Multi-Area OSPF Enterprise Network featuring a central Backbone (Area 0), non-backbone operational subnets (Area 1 & Area 2), and an isolated ISP External Transit Domain.🔵 Core Routing Protocols: OSPFv2 dynamic routing with manually configured Router IDs, custom wildcard masks, and sub-second convergence parameters.🟣 Inter-Area Communication (ABRs): Router R1 and Router R2 act as Area Border Routers, synthesizing intra-area topology into Type 3 Summary LSAs (O IA in the routing table).🔴 External Redistribution (ASBR): Router R3 operates as the Autonomous System Boundary Router, taking static/ISP default routes and injecting them domain-wide via Type 5 External LSAs (O E2 in the routing table).🟡 Layer 2 Transit Infrastructure: Multi-access backbone interconnect via Switch0 (123.0.0.0/29), demonstrating Designated Router (DR) / Backup Designated Router (BDR) election handling.🟠 High-Availability & Redundancy: Secondary inter-router point-to-point transit link (192.168.12.0/30) between ABRs to prevent Area 0 split-brain scenarios and ensure path redundancy.💡 Key Technical Insights & What This Lab Demonstrates🔍 LSA Hierarchy Analysis: Proves full mastery of the OSPF Link-State Database (LSDB) by auditing Type 1 (Router), Type 2 (Network), Type 3 (Summary), and Type 5 (External) LSAs across area boundaries.⚡ Traffic Optimization & Isolation: Demonstrates how multi-area segmentation limits Link-State Advertisement (LSA) flooding, reduces CPU/RAM overhead on routers, and isolates localized link flaps.🛠️ Real-World Verification Skills: Accompanied by full command-line audit logs showcasing show ip ospf neighbor, show ip ospf database, show ip route ospf, and end-to-end traceroute ICMP verification.🛡️ Edge Security & Routing Hygiene: Outlines proper Extended ACL placement and redistribution policies at the ASBR edge to prevent routing loops and ensure controlled external network access.🔒 Extended ACL Security Policy MatrixExtended Access Control Lists (ACLs) are applied closest to the source to filter traffic based on Source IP, Destination IP, Protocol, and Port Number.Rule IDSource Subnet/HostDestination Host/ServiceProtocol / PortActionPurpose / ContextACL-101192.168.10.0/24 (IT)10.0.0.3 (IT-SRV)IP / AnyPERMITFull access for IT team to IT ServerACL-102192.168.10.0/24 (IT)10.0.0.2 (HR-SRV)TCP / 80, 443PERMITWeb-only access for IT to HR ServerACL-103192.168.20.0/24 (HR)10.0.0.2 (HR-SRV)IP / AnyPERMITFull access for HR team to HR ServerACL-104192.168.20.0/24 (HR)10.0.0.3 (IT-SRV)AnyDENYExplicit deny from HR to IT ServerACL-105Any Client8.8.8.8 (WEB-SRV)ICMP / HTTPPERMITOutbound access to external internet resources⚙️ Configuration SnippetsRouter R1: Sub-Interfaces (Router-on-a-Stick) & OSPFinterface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 1
 network 192.168.20.0 0.0.0.255 area 1
 network 192.168.12.0 0.0.0.3 area 0
Router R1: Extended Access Control List Exampleip access-list extended SECURE_DEPT_TRAFFIC
 permit ip 192.168.10.0 0.0.0.255 host 10.0.0.3
 permit tcp 192.168.10.0 0.0.0.255 host 10.0.0.2 eq www
 permit ip 192.168.20.0 0.0.0.255 host 10.0.0.2
 deny ip 192.168.20.0 0.0.0.255 host 10.0.0.3
 permit ip any any
!
interface GigabitEthernet0/0.20
 ip access-group SECURE_DEPT_TRAFFIC in
🧪 Verification CommandsTo verify routing and ACL operation on Cisco IOS:# Verify OSPF Neighbor Relationships & Adjacencies
R1# show ip ospf neighbor

# Inspect Link-State Database (Type 1, 2, 3, and 5 LSAs)
R1# show ip ospf database

# Audit Learned OSPF Routes (Inter-Area & External)
R1# show ip route ospf

# Verify Extended ACL Match Counters
R1# show access-lists

# End-to-End Connectivity Check
PC0> ping 10.0.0.3
PC0> traceroute 8.8.8.8
