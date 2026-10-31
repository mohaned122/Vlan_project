---
title: VLAN Revision and Cisco Packet Tracer Lab Guide
aliases:
  - VLAN Team Study Guide
  - VLAN Presentation Preparation
tags:
  - networking
  - cisco
  - vlan
  - packet-tracer
  - ccna
status: team-study
---

# VLAN Revision & Cisco Packet Tracer Study Guide

> [!abstract] Team goal
> Refresh VLAN concepts, configure VLANs and trunks on Cisco switches, enable inter-VLAN routing, troubleshoot common failures, and prepare a clear technical presentation.

## Table of contents

- [[#1. Learning objectives]]
- [[#2. Revision roadmap]]
- [[#3. VLAN fundamentals]]
- [[#4. VLAN architecture and terminology]]
- [[#5. Cisco IOS command reference]]
- [[#6. Packet Tracer labs]]
- [[#7. Troubleshooting]]
- [[#8. Advanced topics and security]]
- [[#9. Presentation plan]]
- [[#10. Questions for team rehearsal]]
- [[#11. YouTube and reference resources]]
- [[#12. Final checklist]]

---

## 1. Learning objectives

By the end of this guide, each team member should be able to:

- Explain what a VLAN is and why networks use VLANs.
- Distinguish a broadcast domain from a collision domain.
- Explain how a Layer 2 switch learns MAC addresses and forwards frames.
- Configure VLANs, access ports, and 802.1Q trunks using Cisco IOS.
- Explain VLAN tagging and the native VLAN.
- Configure router-on-a-stick and understand Layer 3 switch SVIs.
- Verify a configuration using `show` commands.
- Troubleshoot VLAN membership, trunk, and inter-VLAN routing failures.
- Demonstrate VLAN behavior in Cisco Packet Tracer.

## 2. Revision roadmap

| Session | Focus | Suggested time |
|---|---|---:|
| 1 | VLAN fundamentals and switching | 45 min |
| 2 | Access ports, trunks, 802.1Q, native VLAN | 60 min |
| 3 | Packet Tracer Labs 1 and 2 | 60 min |
| 4 | Inter-VLAN routing and Lab 3 | 60 min |
| 5 | Troubleshooting and advanced topics | 45 min |
| 6 | Team presentation rehearsal | 30 min |

## 3. VLAN fundamentals

### 3.1 What is a VLAN?

A **VLAN (Virtual Local Area Network)** is a logical Layer 2 network that groups devices into separate broadcast domains, even when they are physically connected to the same switch.

Example departments:

- HR — VLAN 10
- IT — VLAN 20
- Finance — VLAN 30

A single physical switch can carry these separate logical networks. VLANs are like logically dividing one switch into multiple independent Layer 2 domains.

### 3.2 Why use VLANs?

| Benefit | Explanation |
|---|---|
| Broadcast control | Broadcast traffic is contained within its VLAN. |
| Segmentation | Departments or device types can be logically separated. |
| Flexibility | Group users by role instead of physical location. |
| Administration | Apply policies and manage network changes more easily. |
| Security foundation | Segmentation supports security design, but is not complete security by itself. |

> [!warning] Important security qualification
> VLANs provide Layer 2 separation. A router or Layer 3 switch may still route between VLANs. Use ACLs or firewall policies when communication must be restricted.

### 3.3 OSI model relationship

VLANs operate primarily at **Layer 2 — Data Link**.

- Layer 1: cables, physical signals, ports.
- Layer 2: Ethernet frames, MAC addresses, switching, VLANs.
- Layer 3: IP addressing and routing between networks.
- Layers 4–7: transport and application services.

**Key distinction:** A Layer 2 switch forwards frames within a VLAN. A router or Layer 3 switch routes packets between IP networks/VLANs.

### 3.4 Broadcast domain vs collision domain

- **Broadcast domain:** devices that receive a Layer 2 broadcast.
- **Collision domain:** a shared Ethernet segment where frame collisions could occur in traditional shared Ethernet.
- **VLAN:** a logical boundary that creates a separate broadcast domain.

In modern full-duplex switched Ethernet, collisions are generally eliminated on individual links. Each switch port is its own collision domain, while VLANs define broadcast domains.

Example: a 24-port switch with three active VLANs has 24 individual switched-port collision domains and three broadcast domains, assuming the VLANs are the only Layer 2 broadcast domains present.

### 3.5 How a switch forwards frames

When a frame arrives:

1. The switch receives the Ethernet frame on an interface.
2. It determines the VLAN associated with the ingress port or tag.
3. It learns the source MAC address in its MAC address table, associated with VLAN and port.
4. It looks up the destination MAC address within that VLAN.
5. If known, it forwards the frame out the destination port.
6. If unknown, it floods within that VLAN (except the ingress port).
7. Broadcast frames are flooded within that VLAN only.

A VLAN 10 broadcast is not flooded into VLAN 20.

### 3.6 Why different VLANs cannot communicate directly

Two hosts in different VLANs are separated at Layer 2—even if they are plugged into the same physical switch. Typically, they also use different IP subnets.

To communicate, they need:
- Correct IP addresses and subnet masks.
- A default gateway for each host.
- A router or Layer 3 switch to route between the VLANs.
- Routing and security policies that permit the traffic.

---

## 4. VLAN architecture and terminology

### 4.1 VLAN IDs and ranges

The 802.1Q VLAN ID field is 12 bits:

`2^12 = 4096` possible values, 0–4095.

- VLAN ID 0 is reserved for priority tagging.
- VLAN ID 4095 is reserved.
- Usable VLAN IDs are 1–4094.

Traditional Cisco VLAN range details vary by platform/software. Commonly:
- VLAN 1 is the default VLAN.
- VLANs 2–1001 are normal-range VLANs.
- VLANs 1002–1005 are historically reserved for legacy technologies.
- VLANs 1006–4094 are extended-range VLANs.

Check the documentation for the specific switch model and IOS version used in the lab.

### 4.2 Default VLAN

On many Cisco switches, VLAN 1 is the default VLAN. Access ports initially belong to VLAN 1, and it is commonly the default native VLAN on trunks.

A good practice is to explicitly assign user access ports to their intended VLAN and explicitly configure trunk settings.

### 4.3 Access port

An access port belongs to one data VLAN. Typical endpoint connections include PCs, printers, and servers.

```cisco
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
```

Access ports normally send and receive untagged frames for their assigned VLAN.

### 4.4 Trunk port

A trunk carries traffic for multiple VLANs over one physical link. Commonly used between switches, between a switch and router, or between a switch and a Layer 3 device.

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
```

A trunk allows VLANs to span switches without needing a separate physical link for every VLAN.

### 4.5 IEEE 802.1Q VLAN tagging

IEEE 802.1Q identifies VLAN membership on trunk links. The tag is inserted into the Ethernet frame after the source MAC address and before the original EtherType/Length field.

The 4-byte tag contains:
- TPID — identifies the tag protocol.
- PCP — priority information.
- DEI — drop eligibility indicator.
- VID — 12-bit VLAN ID.

The tag lets the receiving switch identify the VLAN associated with a frame on a trunk.

### 4.6 Native VLAN

The native VLAN is the VLAN associated with untagged traffic on an 802.1Q trunk. On many Cisco switches, VLAN 1 is the default native VLAN.

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 99
```

The native VLAN should match on both ends of a trunk.

**Do not confuse:**
- Access VLAN: VLAN assigned to an access port.
- Native VLAN: VLAN associated with untagged traffic on a trunk.

### 4.7 Allowed VLAN list

Restrict which VLANs can cross a trunk:

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

Add one VLAN without replacing the existing list:

```cisco
switchport trunk allowed vlan add 40
```

The allowed list is a common troubleshooting point: a VLAN may exist and be assigned correctly but still be blocked from crossing a trunk.

### 4.8 MAC address table and VLAN context

A switch learns MAC addresses against VLAN and port context. A MAC entry learned in VLAN 10 is not treated as a normal destination entry for VLAN 20.

```cisco
show mac address-table
show mac address-table vlan 10
```

### 4.9 Inter-VLAN routing

Inter-VLAN routing is Layer 3 forwarding between VLANs. Two common approaches:

1. **Router-on-a-stick:** one router physical interface with multiple VLAN subinterfaces connected to a switch trunk.
2. **Layer 3 switch with SVIs:** a multilayer switch routes internally using Switch Virtual Interfaces.

#### Router-on-a-stick

Each subinterface is associated with a VLAN using 802.1Q encapsulation:

```cisco
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
```

#### Layer 3 switch SVI

```cisco
ip routing

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

The SVI provides a Layer 3 interface for the VLAN. The VLAN and at least one active Layer 2 port in that VLAN generally need to be operational for the SVI to come up, depending on platform.

---

## 5. Cisco IOS command reference

### VLAN management

```cisco
show vlan brief
show vlan id 10

configure terminal
vlan 10
 name HR
exit
```

Delete a VLAN (use carefully):

```cisco
no vlan 10
```

### Access ports

```cisco
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
```

Port range:

```cisco
interface range fastEthernet 0/1-10
 switchport mode access
 switchport access vlan 10
```

### Trunk configuration

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 switchport trunk native vlan 99
```

### Verification

```cisco
show interfaces trunk
show interfaces gigabitEthernet 0/1 switchport
show interfaces status
show mac address-table
show ip interface brief
show running-config
```

### Save configuration

```cisco
copy running-config startup-config
```

> [!note] Packet Tracer compatibility
> Available commands and interface names depend on the simulated device. If a command is rejected, use `?` and check the model/IOS capabilities. Some Packet Tracer switch models support a subset of real IOS features.

---

## 6. Packet Tracer labs

### Lab 1 — Basic VLAN segmentation

**Goal:** Create two broadcast domains on one switch and observe that hosts in different VLANs cannot communicate without Layer 3 routing.

#### Topology and addressing

Use one Cisco 2960 switch and four PCs.

| Device | Switch port | VLAN | IP address |
|---|---|---:|---|
| PC1 | Fa0/1 | 10 | 192.168.10.11/24 |
| PC2 | Fa0/2 | 10 | 192.168.10.12/24 |
| PC3 | Fa0/3 | 20 | 192.168.20.11/24 |
| PC4 | Fa0/4 | 20 | 192.168.20.12/24 |

Subnet mask: `255.255.255.0`. No default gateway is needed for this lab.

#### Steps

1. Place one 2960 switch and four PCs.
2. Connect each PC to the switch with copper straight-through cable.
3. Set each PC's IP address.
4. Create VLANs:

```cisco
enable
configure terminal
vlan 10
 name HR
exit
vlan 20
 name IT
exit
```

5. Assign ports:

```cisco
interface range fastEthernet 0/1-2
 switchport mode access
 switchport access vlan 10
exit

interface range fastEthernet 0/3-4
 switchport mode access
 switchport access vlan 20
exit
```

6. Verify:

```cisco
show vlan brief
```

Expected: Fa0/1 and Fa0/2 in VLAN 10; Fa0/3 and Fa0/4 in VLAN 20.

7. Test:
   - PC1 → `ping 192.168.10.12`: expected success.
   - PC1 → `ping 192.168.20.11`: expected failure.

8. Use Packet Tracer **Simulation** mode and send a Simple PDU. Observe ARP and ICMP. Confirm that VLAN 10 traffic does not enter VLAN 20.

**Explain:** same VLAN + same subnet works; different VLANs require a Layer 3 path.

### Lab 2 — VLANs across two switches using a trunk

**Goal:** Carry VLAN 10 and VLAN 20 over one switch-to-switch link.

#### Topology

```text
PC1 (VLAN 10) -- SW1 ===== trunk ===== SW2 -- PC3 (VLAN 10)
PC2 (VLAN 20) -- SW1                  SW2 -- PC4 (VLAN 20)
```

Addressing:

| Device | VLAN | IP address |
|---|---:|---|
| PC1 | 10 | 192.168.10.11/24 |
| PC2 | 20 | 192.168.20.11/24 |
| PC3 | 10 | 192.168.10.12/24 |
| PC4 | 20 | 192.168.20.12/24 |

Connect SW1 Fa0/24 to SW2 Fa0/24.

#### Steps

1. Create VLAN 10 and VLAN 20 on both switches.
2. Assign Fa0/1 to VLAN 10 and Fa0/2 to VLAN 20 on each switch.
3. Configure the trunk on both switches:

```cisco
interface fastEthernet 0/24
 switchport mode trunk
```

4. Verify:

```cisco
show interfaces trunk
show vlan brief
```

5. Test:
   - PC1 → PC3: success.
   - PC2 → PC4: success.
   - PC1 → PC2: failure without routing.

6. Restrict VLANs on both trunk ends:

```cisco
interface fastEthernet 0/24
 switchport trunk allowed vlan 10
```

Now VLAN 20 traffic cannot cross the trunk. Test PC2 → PC4; expected failure.

7. Restore both VLANs:

```cisco
switchport trunk allowed vlan 10,20
```

**Explain:** VLANs span switches through trunks; the trunk's allowed VLAN list controls which VLANs can cross.

### Lab 3 — Router-on-a-stick

**Goal:** Route between VLAN 10 and VLAN 20 using a router trunk and subinterfaces.

#### Topology

```text
PC1 (VLAN 10) ----\
                   SW1 ===== trunk ===== R1
PC2 (VLAN 20) ----/                 G0/0.10, G0/0.20
```

Addressing:

| Device | Address | Default gateway |
|---|---|---|
| PC1 | 192.168.10.11/24 | 192.168.10.1 |
| PC2 | 192.168.20.11/24 | 192.168.20.1 |
| R1 G0/0.10 | 192.168.10.1/24 | — |
| R1 G0/0.20 | 192.168.20.1/24 | — |

Connect SW1 Gi0/1 to R1 Gi0/0 (use the actual available interfaces on the chosen model).

#### Switch configuration

```cisco
enable
configure terminal
vlan 10
 name HR
vlan 20
 name IT

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20

interface gigabitEthernet 0/1
 switchport mode trunk
```

#### Router configuration

```cisco
enable
configure terminal

interface gigabitEthernet 0/0
 no shutdown

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

Set PC1 and PC2 IP addresses, masks, and gateways from the table.

#### Test and explain packet flow

From PC1:

```text
ping 192.168.20.11
```

Expected: success.

Packet path:
1. PC1 sees the destination is in another IP subnet.
2. PC1 sends the frame to its default gateway.
3. SW1 forwards it over the trunk tagged for VLAN 10.
4. R1 receives it on G0/0.10 and routes the IP packet.
5. R1 sends it out G0/0.20 tagged for VLAN 20.
6. SW1 forwards the frame to PC2's access port.
7. PC2 receives the packet.

**Key point:** The switch handles Layer 2 forwarding; the router performs Layer 3 routing.

### Optional Lab 4 — Layer 3 switch SVI

If the Packet Tracer model supports Layer 3 switching, use a multilayer switch.

```cisco
configure terminal
ip routing

vlan 10
vlan 20

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```

Assign access ports to VLAN 10 and VLAN 20. Set each PC's default gateway to its SVI IP. Verify with:

```cisco
show ip interface brief
show ip route
```

---

## 7. Troubleshooting guide

### Symptom: Same-VLAN PCs cannot communicate

Check:
- Correct IP subnet and mask?
- Correct access VLAN?
- Interfaces up?
- Cabling correct?

Commands:

```cisco
show vlan brief
show interfaces status
show mac address-table
```

### Symptom: Same VLAN across switches cannot communicate

Check:
- Trunk exists and is operational?
- VLAN exists on both switches?
- VLAN allowed on trunk?
- Native VLAN matches?

```cisco
show interfaces trunk
show vlan brief
show interfaces gigabitEthernet 0/1 switchport
```

### Symptom: Different VLANs cannot communicate

Check:
- Correct IP address, mask, and gateway?
- Router subinterfaces or SVIs configured?
- Correct 802.1Q VLAN ID?
- Router physical interface up?
- Switch-to-router link is trunk?
- Routing enabled on multilayer switch?

```cisco
show ip interface brief
show running-config
show interfaces trunk
show ip route
```

### Symptom: VLAN is excluded from a trunk

Example: only VLAN 10 is allowed.

```cisco
switchport trunk allowed vlan 10
```

Add VLAN 20:

```cisco
switchport trunk allowed vlan add 20
```

### Symptom: Native VLAN mismatch

If one end uses native VLAN 99 and the other uses VLAN 1, configure matching native VLANs on both sides.

### Troubleshooting method

1. Identify the failing source and destination.
2. Check IP addressing and subnet masks.
3. Check access VLAN membership.
4. Check trunk state and allowed VLAN list.
5. Check gateway/subinterface/SVI configuration.
6. Verify with show commands.
7. Test again with ping and Simulation mode.

---

## 8. Advanced topics and security

| Topic | What to know |
|---|---|
| VTP | Cisco protocol that can distribute VLAN configuration between switches. |
| DTP | Cisco protocol that can negotiate trunk formation. |
| STP | Prevents Layer 2 switching loops. |
| Voice VLAN | VLAN for IP phone voice traffic. |
| Private VLAN | Provides additional isolation within a VLAN. |
| VLAN hopping | Attack class that attempts to cross VLAN boundaries. |
| VLAN pruning | Limits unnecessary VLAN traffic across trunks. |
| SVI | Logical Layer 3 interface associated with a VLAN. |

Security practices:
- Explicitly configure access ports.
- Put unused ports in an unused VLAN and shut them down where appropriate.
- Restrict trunk allowed VLANs.
- Disable trunk negotiation where appropriate for the platform/design.
- Keep native VLAN configuration consistent; avoid using the default VLAN for user traffic where practical.
- Use ACLs/firewalls to control inter-VLAN communication.
- Apply port security and other access-layer controls when appropriate.

---

## 9. Presentation plan (10–15 minutes)

| Slide | Topic | Core message |
|---|---|---|
| 1 | Introduction | VLAN = Virtual Local Area Network |
| 2 | The problem | Flat networks have a large broadcast domain |
| 3 | VLAN concept | Logical segmentation on shared physical infrastructure |
| 4 | Benefits | Broadcast control, flexibility, security foundation |
| 5 | Access ports | One data VLAN per access port |
| 6 | Trunk ports | Multiple VLANs on one link |
| 7 | 802.1Q | Tags identify VLAN traffic on trunks |
| 8 | Inter-VLAN routing | Router or Layer 3 switch connects VLANs |
| 9 | Packet Tracer demo | Show isolation, then routing |
| 10 | Conclusion | VLANs are a core switching concept |

### Suggested opening

> Imagine a company with three departments: HR, IT, and Finance. Each department needs its own network, but we do not want a separate physical switch for every department. VLANs allow us to divide one physical switching infrastructure into multiple logical networks. This improves organization, controls broadcast traffic, and creates a foundation for security policies.

### Recommended live demonstration

1. Start with four PCs on one switch.
2. Create VLAN 10 and VLAN 20.
3. Show same-VLAN ping succeeds and cross-VLAN ping fails.
4. Add a trunk between switches (optional) to show VLAN extension.
5. Configure router-on-a-stick.
6. Show cross-VLAN ping succeeds.
7. Explain the packet path and the role of the default gateway.

---

## 10. Questions for team rehearsal

1. **Can two PCs in different VLANs communicate without a router?**  
   Not through ordinary Layer 2 switching. A Layer 3 device is required.

2. **Access port vs trunk port?**  
   Access port is assigned to one data VLAN; trunk carries multiple VLANs.

3. **What is 802.1Q?**  
   The VLAN tagging standard used to identify VLANs on trunk links.

4. **What is the native VLAN?**  
   The VLAN associated with untagged traffic on an 802.1Q trunk.

5. **Do VLANs guarantee complete security?**  
   No. Routing may connect them; ACLs/firewalls enforce restrictions.

6. **Why must a VLAN be allowed on a trunk?**  
   Otherwise its traffic is not carried across that trunk.

7. **Where does a broadcast go?**  
   Within its VLAN only.

8. **Can a VLAN span multiple switches?**  
   Yes, through trunk links.

9. **What is router-on-a-stick?**  
   One router physical interface with multiple VLAN-specific subinterfaces.

10. **What is an SVI?**  
    A logical Layer 3 interface associated with a VLAN, commonly used on multilayer switches.

---

## 11. YouTube and reference resources

### YouTube — recommended learning order

1. **Jeremy's IT Lab — VLANs Part 1 (Day 16)**  
   Conceptual foundations, broadcast domains, VLAN purpose, basic configuration.  
   [Watch on YouTube](https://www.youtube.com/watch?v=cjFzOnm6u1g)

2. **Jeremy's IT Lab — VLANs Part 2 (Day 17)**  
   Trunk ports, 802.1Q tags, native VLAN, allowed VLANs, router-on-a-stick.  
   [Watch on YouTube](https://www.youtube.com/watch?v=Jl9OOzNaBDU)

3. **Jeremy's IT Lab — VLANs Part 3 (Day 18)**  
   Native VLAN details, multilayer switches, SVIs, inter-VLAN routing.  
   [Watch on YouTube](https://www.youtube.com/watch?v=OkPB028l2eE)

4. **Jeremy's IT Lab — VLANs Part 1 Lab (Day 16)**  
   Basic VLAN configuration and access-port practice in Packet Tracer.  
   [Watch on YouTube](https://www.youtube.com/watch?v=-tq7f3xtyLQ)

5. **Jeremy's IT Lab — VLANs Part 2 Lab (Day 17)**  
   Practical access ports, trunk between switches, router-on-a-stick testing.  
   [Watch on YouTube](https://www.youtube.com/watch?v=iRkFE_lpYgc)

6. **Jeremy's IT Lab — VLANs Part 3 Lab (Day 18)**  
   Replacing router-on-a-stick with Layer 3 switch routing and SVIs.  
   [Watch on YouTube](https://www.youtube.com/watch?v=MQcCr3QW1vE)

7. **DLTEC do Brasil — VLAN configuration with Packet Tracer and Cisco IOS**  
   VLANs, management VLANs, VTP, and router-on-a-stick.  
   [Watch on YouTube](https://www.youtube.com/watch?v=uNoDj5eLy4I)

8. **Cisco Config — Router-on-a-stick walkthrough**  
   Switch/router configuration, VLANs, trunking, subinterfaces, and verification.  
   [Watch on YouTube](https://www.youtube.com/watch?v=u8Xya6wfmtE)

### Official and supporting references

- [Cisco Learning Network — VLAN fundamentals](https://learningnetwork.cisco.com/s/article/fundamentals-of-creating-vlans-part-1-the-concept)
- [Cisco — Configure VLAN trunks](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr2-fwd/vlan/vlan-configuration-guide/configure-vlan-trunks.html)
- [Cisco Networking Academy](https://www.netacad.com/)
- [Packet Tracer VLAN configuration activity (PDF)](https://www.netacad.com/content/srwe/1.0/courses/content/m3/en-US/assets/3.3.12-packet-tracer---vlan-configuration.pdf)

> [!tip] How to study with the videos
> Watch the concept video first, pause before the configuration, and reproduce each command in Packet Tracer. Then deliberately break one setting (wrong access VLAN, excluded VLAN, or wrong gateway) and diagnose it using `show` commands.

---

## 12. Final team checklist

- [ ] We can define VLANs in our own words.
- [ ] We understand broadcast domains and collision domains.
- [ ] We can explain MAC learning and VLAN-scoped forwarding.
- [ ] We can create VLANs and assign access ports.
- [ ] We can configure and verify a trunk.
- [ ] We understand 802.1Q and native VLANs.
- [ ] We can configure router-on-a-stick.
- [ ] We understand SVIs and Layer 3 switching.
- [ ] We can troubleshoot a VLAN and trunk issue.
- [ ] We have rehearsed the Packet Tracer demonstration.
- [ ] Each team member can answer the rehearsal questions.

## Team task split (optional)

- **Member A:** VLAN fundamentals and OSI explanation.
- **Member B:** Access ports, trunking, 802.1Q, native VLAN.
- **Member C:** Packet Tracer Labs 1 and 2.
- **Member D:** Router-on-a-stick, SVIs, and troubleshooting.
- **All members:** Rehearse the demonstration and Q&A.

---

## Sources and accuracy notes

This guide is a study aid. VLAN ranges, command support, trunk negotiation behavior, and SVI operational requirements can vary by Cisco platform and software. Use the command help (`?`) and platform documentation for the device selected in Packet Tracer or in a real lab.
