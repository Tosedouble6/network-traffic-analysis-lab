# Network Reconnaissance & Packet Capture Lab (Ubuntu Server + Wireshark)

Hands-on network engineering and packet inspection lab conducted on a dedicated **Ubuntu Server 24.04 LTS** virtual machine deployed via **Hyper-V** in Bridged networking mode.

## Lab Environment
- **Target / Sensor VM:** Ubuntu Server 24.04.3 LTS (192.168.0.46/24, Hyper-V Gen 2, External Virtual Switch)
- **Host / Analysis Workstation:** Windows 11 + Wireshark 4.6.10
- **Tools Used:** nmap, tcpdump, openssh-server, scp, curl, Wireshark

## Workflow & Execution

### 1. Active Subnet Discovery (nmap)
Performed an ICMP/ARP host discovery sweep across the local /24 subnet to verify Layer 2/Layer 3 bridged visibility:
```bash
nmap -sn 192.168.0.0/24
```
- **Result:** Identified 5 active hosts (192.168.0.1 gateway, .12, .15, .46 sensor VM, and .49).

### 2. Targeted Packet Capture (tcpdump)
Captured live Ethernet frames on eth0 while filtering out management SSH traffic (not port 22) to isolate ICMP, DNS, and HTTP flows:
```bash
sudo tcpdump -i eth0 not port 22 -c 40 -w ~/clean_traffic.pcap & sleep 1 && ping -c 3 1.1.1.1 && curl -I http://example.com
```

### 3. Packet-Level Breakdown (captures/clean_traffic.pcap)
- **Layer 2 / ARP Resolution (Frames 7-8):** Gateway (192.168.0.1) resolves the VM Hyper-V MAC address (00:15:5d:38:01:00) via broadcast ARP before forwarding echo replies.
- **ICMP Echo Exchange (Frames 6-14):** Three Echo Request/Reply pairs to 1.1.1.1 with an average RTT of 20.05 ms (TTL=56).
- **DNS Resolution (Frames 15-18):** Simultaneous A (IPv4 -> 104.20.23.154) and AAAA (IPv6) queries for example.com sent to 178.235.153.33.
- **TCP 3-Way Handshake & HTTP (Frames 19-27):** SYN (Frame 19), SYN-ACK (Frame 20), ACK (Frame 21), followed by HEAD / HTTP/1.1 and HTTP/1.1 200 OK.

![Wireshark Capture Breakdown](screenshots/wireshark-tcp-http-handshake.png)
