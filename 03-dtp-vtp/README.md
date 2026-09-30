DTP and VTP

Tool: Cisco Packet Tracer Exam: CCNA 200-301

Goal
Build a static 802.1Q trunk with DTP turned off.
Set up VTP with a server, a transparent switch, and a client.
Prove that VLAN information reaches the client through the transparent switch.
   SW1 (VTP Server)
        |
      trunk
        |
   SW2 (VTP Transparent)
        |
      trunk
        |
   SW3 (VTP Client)
   Domain name on all switches: CCNA. VTP version running: 1.

The three protocols (do not mix them up)
Protocol	Question it answers
DTP	Should this link become a trunk?
802.1Q	Which VLAN does this frame belong to?
VTP	Which VLANs should exist on the switches?
Configuration

Trunk ports (static trunk, DTP off):
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
 VTP (change the mode on each switch):
 vtp domain CCNA
vtp version 1
vtp mode server        ! SW1
vtp mode transparent   ! SW2
vtp mode client        ! SW3
Results
Switch	Mode	Domain	Revision
SW1	Server	CCNA	3
SW2	Transparent	CCNA	0
SW3	Client	CCNA	3
Trunks show trunking, encapsulation 802.1q, native VLAN 1.
Negotiation of Trunking: Off on the trunk ports.
SW3 got revision 3 through SW2. Transparent mode does not apply the VLAN database, but it forwards VTP messages (when the domain name matches).
SW2 has 9 VLANs, SW1 and SW3 have 8. This is normal because SW2 keeps its own VLAN database.
How I verified it
text
show vlan brief
show vtp status
show interfaces trunk
show interfaces gigabitEthernet0/1 switchport
Key lessons
Transparent = forwards, does not apply. It keeps its own VLANs but passes VTP messages along.
A trunk is not always using DTP. switchport mode trunk still sends DTP frames unless you add switchport nonegotiate.
The revision number is dangerous. A switch with the same domain name and password and a higher revision number overwrites the VLANs on the other switches. VLANs get deleted and ports go inactive.
Fix before connecting a used switch: reset its revision to 0. Change the domain name and change it back, or switch to transparent mode and back.
DTP should be off for security. With DTP on, an attacker can make a port become a trunk (switch spoofing) and then reach other VLANs (VLAN hopping).
Security best practice
User ports: switchport mode access
Trunk ports: switchport mode trunk + switchport nonegotiate
Unused ports: shut down and move to an unused VLAN
Real-world note

Many engineers avoid VTP because one wrong switch can wipe the VLANs. They use transparent mode or VTP v3, and they configure VLANs and trunks by hand or with automation.
