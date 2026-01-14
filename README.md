# Network Reconnaissance and Traffic Analysis Lab

## Overview

This project documents a hands-on network reconnaissance and traffic analysis lab performed in a segmented environment protected by a pfSense firewall. The goal was to understand how hosts communicate on a LAN, how reconnaissance traffic appears on the network, and how exposed services can be identified from both internal and external perspectives.

The lab simulates early-stage security assessment and reconnaissance techniques commonly used in both defensive monitoring and offensive security testing.

---

## Lab Environment

The environment consisted of:
- Internal Windows hosts (vWorkstation, TargetWindows01)
- A pfSense firewall acting as the LAN/WAN boundary
- An external Windows host
- A Kali Linux system (AttackLinux01) representing an attacker or red team system

![Lab Network Diagram](LabSetup.png)
<img width="1191" height="914" alt="LabSetup" src="https://github.com/user-attachments/assets/8ab1a3ba-85bf-4f86-ac9d-c397f0fdeb35" />

---

## Section 1: Internal Network Discovery (LAN)

### Windows Network Configuration

On the internal Windows systems, `ipconfig` and `ipconfig /all` were used to identify network configuration details. Each system contained multiple adapters, but the **Student adapter** was used for internal LAN communication.

This revealed internal IP addressing, subnet masks, and the default gateway pointing to the internal firewall interface.

![Windows IP Configuration](Ipconfig.png)
<img width="1911" height="882" alt="Ipconfig" src="https://github.com/user-attachments/assets/e57ffcc9-30cd-4644-bc4e-3e50af081a45" />

---

### ARP Cache Examination

The Address Resolution Protocol (ARP) cache was inspected using `arp -a` to observe existing IP-to-MAC address mappings. The cache was then cleared using `arp -d` to remove previously learned entries.

This allows fresh ARP activity to be observed when communication is re-established.

![ARP Cache Output](arp.png)
<img width="1917" height="880" alt="arp" src="https://github.com/user-attachments/assets/c6ac2431-f976-45da-81df-45a270a9c73f" />

---

### Connectivity Testing and ARP Repopulation

A successful `ping` was sent from vWorkstation to TargetWindows01. After the ping, the ARP cache was checked again, and a new **dynamic ARP entry** appeared for the target host.

This demonstrates how ARP entries are dynamically populated during normal network communication and how ARP can be used to identify active hosts on a LAN.

---

## Section 2: LAN Analysis with Wireshark and Nmap

### Packet Capture and Filtering

Wireshark was used to capture traffic on the internal Student interface. Filters were applied to isolate specific protocol activity:

- `icmp` to observe ping traffic
- `arp` to observe address resolution behavior

ARP traffic confirmed Layer 2 address resolution between hosts and the internal firewall interface.

![Wireshark ARP Filtering](WiresharkFiltering.png)
<img width="1917" height="876" alt="WiresharkFiltering" src="https://github.com/user-attachments/assets/f8c9aacf-dce1-4c5d-ab40-34caad19b530" />

---

### Network Scanning and Traffic Correlation

Nmap scan
s were performed using Zenmap with increasing levels of intensity. As scan aggressiveness increased, additional traffic patterns were observed in Wireshark.

This demonstrates how reconnaissance activity becomes visible at the packet level and how defenders can identify scanning behavior.

![Nmap Scan Traffic in Wireshark](nmapscantowiresharkcapture.png)
<img width="1283" height="793" alt="nmapscantowiresharkcapture" src="https://github.com/user-attachments/assets/77f576d7-9f01-4b2f-8e16-5cf40bb41edd" />

---

### Open Ports and Services Identified

An Intense Nmap scan identified the following open ports on the firewall interface:

- TCP 22 (SSH)
- TCP 80 (HTTP)

These open ports indicate active services and represent potential attack surfaces if not properly secured.

![Zenmap Scan Results](Zenmap.png)
<img width="1913" height="882" alt="Zenmap" src="https://github.com/user-attachments/assets/ab5005ea-ee07-4ad9-97a4-346e456db15d" />

---

## Section 3: External Network and Perimeter Assessment (WAN)

### Attacker Network Configuration (Linux)

On the Kali Linux system (AttackLinux01), `ifconfig` and `ifconfig -a` were used to identify active network interfaces, IP addresses, and MAC addresses. This established the attacker’s position on the external network.

![Linux Network Configuration](ifconfig.png)
<img width="1278" height="771" alt="ifconfig" src="https://github.com/user-attachments/assets/696794a0-8e4c-4c64-8b93-3412b5ca7ed9" />

---

### ICMP Probing with hping3

The `hping3` utility was used to send crafted ICMP packets to the firewall’s external interface. Unlike standard ping, hping3 allows precise control over packet structure.

Successful ICMP replies confirmed that the firewall responds to external ICMP traffic.

![hping3 ICMP Probe](hping.png)
<img width="1272" height="764" alt="hping" src="https://github.com/user-attachments/assets/c80cadef-e510-4b79-93bb-61e5260d5834" />

---

### TCP SYN Analysis and Three-Way Handshake Observation

Using hping3, TCP SYN packets were sent toward the firewall. When targeting port 80, the firewall responded with a **SYN/ACK**, confirming that the service was listening.

This interaction was captured using tcpdump and demonstrates the first stages of a TCP three-way handshake.

![hping3 TCP Handshake](hping(3wayhandshake).png)
<img width="1269" height="776" alt="hping(3wayhandshake)" src="https://github.com/user-attachments/assets/81a6a6af-c09c-4c6b-b73c-7f45eca33b70" />

![tcpdump Three-Way Handshake Capture](Tcpdump(3waycapture).png)
<img width="1271" height="765" alt="Tcpdump(3waycapture)" src="https://github.com/user-attachments/assets/d97f7619-e277-4769-aa35-581d0d672eaf" />

---


### External Traffic Capture with tcpdump

tcpdump was used to capture live traffic on the attacker interface. Captured traffic included ICMP echo requests/replies and TCP SYN and SYN/ACK packets.

This confirms that selected inbound traffic is permitted through the firewall.

![tcpdump Capture Output](tcpdump.png)
<img width="1277" height="768" alt="tcpdump" src="https://github.com/user-attachments/assets/fe1be9b1-485e-4917-ac65-bd9c0b59e344" />

---

### Service Enumeration via Telnet

A telnet connection was established to the firewall on port 80, and a manual HTTP GET request was sent. The response revealed server details, including the web server software (nginx).

This demonstrates how banner information can leak useful details to attackers and why banner minimization is an important security practice.

![Telnet HTTP Banner](telnetconnection.png)
<img width="1272" height="759" alt="telnetconnection" src="https://github.com/user-attachments/assets/edd91656-c9f6-4301-aa1d-4928edcc52b7" />

---

## Key Takeaways

- ARP behavior reveals active hosts and communication patterns on a LAN
- Network scanning activity is visible in packet captures
- Firewalls can be evaluated by observing how they respond to crafted traffic
- Open ports and service banners represent potential security risks
- Traffic analysis tools are essential for both detection and investigation

---

## Tools Used

- Windows: `ipconfig`, `arp`, `ping`
- Wireshark
- Nmap / Zenmap
- Kali Linux: `ifconfig`, `hping3`, `tcpdump`, `telnet`
- pfSense firewall

---

## Disclaimer

All testing was performed in a controlled lab environment for educational purposes only.
