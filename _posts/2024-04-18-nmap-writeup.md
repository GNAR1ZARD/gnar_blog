---
layout: post
title: "Network Security with Nmap: A Comprehensive Guide"
date: 2024-04-18
---

## Nmap Switches

Nmap is a powerful network scanning tool that uses various switches to customize its behavior. Below are some commonly used switches and their functionalities:

### Switch Descriptions

-Pn: Skips the host discovery phase, assuming the target is online. This is useful for targets that don't respond to ping requests, effectively avoiding unnecessary preliminary checks.
-v: Increases the verbosity level of Nmap's output to provide more detailed information about the scanning process.
-vv: Further increases the verbosity to an even more detailed "very verbose" level, giving granular details about the scan's progress.
-A: Activates a comprehensive set of scanning features including OS detection (-O), version detection (-sV), script scanning, and traceroute, aiming to provide an in-depth analysis of the target.
-T: Specifies the timing template (from 0 for "paranoid" to 5 for "insane"), controlling the speed and stealth of the scan by adjusting how aggressively packets are sent.
-p: Directs Nmap to scan specific ports or ranges of ports, allowing for targeted scanning based on the input format (e.g., -p 1-1000 for a range or -p 80,443,8080 for specific ports).
-oA: Saves the scan results in all major output formats (normal, XML, and grepable) at once, providing a comprehensive set of files for analysis and reporting.
-oN: Saves the scan results in a normal, human-readable format.
-oG: Saves the results in a "grepable" format, which is useful for easy parsing and analysis via text processing tools.
--script: Activates specific Nmap Scripting Engine (NSE) scripts or categories of scripts (e.g., --script=vuln to run all vulnerability scanning scripts).

### Example Command

To run an Nmap scan with these options, you can structure the command as follows:

```bash
nmap -vv -A -T4 -Pn -p 1-65535 <target>
```

This command configuration will start a very verbose scan with advanced features on a target that doesn't respond to ping requests, using a fairly aggressive timing, and scanning all possible ports.

Flags or switches can be placed in any order in the command line.

## Scan Types Overview

Nmap offers a variety of scan types for different purposes. Understanding each type's mechanics and use case can greatly enhance the effectiveness of your reconnaissance in penetration testing.

#### Basic Scan Types

1. **TCP Connect Scans (`-sT`)**:
   - This scan type attempts to establish a full TCP connection with each targeted port. It completes the three-way handshake and is logged by most systems. This is the most basic form of TCP scanning but can be easily detected.

2. **SYN "Half-open" Scans (`-sS`)**:
   - Often referred to as stealth scanning, it sends a SYN packet and waits for a response, but does not complete the handshake (does not send the final ACK). This method is less likely to be logged by the target system, making it preferable for avoiding detection.

3. **UDP Scans (`-sU`)**:
   - UDP scanning involves sending UDP packets to different ports and observing the response. This type of scan is essential for discovering "open" UDP ports as there is no handshake protocol like TCP, making it more challenging and slower to execute accurately.

#### Advanced TCP Scan Types

1. **TCP Null Scans (`-sN`)**:
   - This scan type sends a TCP packet with no flags set. Most operating systems will respond to these packets with a reset response if the port is closed, but no response is expected for open ports. This is useful for evading firewall rules that expect specific flag combinations.

2. **TCP FIN Scans (`-sF`)**:
   - Similar to the Null Scan, a FIN scan sends TCP packets with only the FIN flag set, designed to bypass non-stateful firewalls and some IDS (Intrusion Detection Systems) configurations. Closed ports respond with a reset, while open ports do not respond at all.

3. **TCP Xmas Scans (`-sX`)**:
   - This scan sends packets with the FIN, PSH, and URG flags set, lighting up the packet like a Christmas tree. It operates under the same principle as Null and FIN scans, where closed ports send a reset, and open ports are silent.

#### Network Scanning: ICMP

- **ICMP (or "ping") Scanning**:
   - ICMP scanning involves sending echo request messages to network devices and monitoring echo replies (ping responses). This type of scan is typically used to discover which hosts are up in a network, forming a foundational step in network mapping.

Understanding the nuances of these scan types allows you to tailor your approach to the network environment and the security measures in place. While TCP Connect, SYN, and UDP scans cover most needs, specialized scans like Null, FIN, and Xmas provide tools for more specific contexts, especially in tightly secured or monitored networks.

## TCP Connect Scans

To enhance your understanding of TCP Connect scans (`-sT`) in Nmap, it's useful to have a solid grasp of the TCP/IP three-way handshake, which is pivotal in network communications, particularly for establishing a connection over TCP.

#### Command Usage
```bash
nmap -sT <target>
```

#### Overview
The TCP Connect scan is the default TCP scan method used by Nmap when executed without root privileges. This scan type completes the full TCP three-way handshake process with each target port, directly interacting with the TCP/IP stack of the operating system.

#### Three-Way Handshake
The three-way handshake involves:
1. **SYN Sent**: The client (attacker's machine) sends a SYN packet to initiate a connection.
2. **SYN/ACK Received**: If the port is open, the server responds with a SYN/ACK packet.
3. **ACK Sent**: The client completes the handshake by sending an ACK packet.

This handshake mechanism confirms whether a TCP port is open and able to accept connections.

#### Responses and Port Status
- **Closed Ports**: According to RFC 9293, a closed port responds to unsolicited SYN packets with an RST (Reset) packet. This allows Nmap to accurately determine that the port is closed.
- **Open Ports**: An open port responds to a SYN with a SYN/ACK, prompting Nmap to mark the port as open after completing the handshake with an ACK.
- **Filtered Ports**: If a port is behind a firewall that drops packets (a common configuration), Nmap receives no response, leading it to categorize the port as filtered. Alternatively, firewalls can be configured to send a TCP RST packet, misleading Nmap into recording the port as closed.

#### Firewall Considerations
Configuring a firewall to respond with a TCP reset packet can be done simply on Linux using IPtables:
```bash
iptables -I INPUT -p tcp --dport <port> -j REJECT --reject-with tcp-reset
```
This configuration complicates the accurate assessment of port status, as it can make an open port appear closed.

## SYN Scans

#### Command for SYN Scan

```bash
nmap -sS <target>
```

#### Overview

SYN scans, indicated by `-sS`, are a common method for exploring the TCP ports of a target. These scans are distinctive from full TCP scans, being sometimes referred to as "Half-open" or "Stealth" scans.

#### How SYN Scans Work

In a SYN scan, the scanner sends a SYN packet and, upon receiving a SYN/ACK in response, it returns a RST packet. This sequence avoids the completion of the three-way handshake that characterizes a full TCP scan, thus:
1. A SYN packet is sent to the target port.
2. If the port is open, the target responds with a SYN/ACK.
3. The scanner then sends a RST packet to prevent a full connection establishment.

#### Advantages

SYN scans offer several benefits:
- **Stealth Mode**: They are less likely to be detected by older Intrusion Detection Systems (IDS) that monitor for completed handshakes.
- **No Logs on Target**: Many applications log connections only upon a completed handshake, so SYN scans may not be recorded.
- **Speed**: These scans are faster as they do not establish and then close a full connection for each checked port.

#### Disadvantages

However, there are drawbacks:
- **Root Privileges Required**: SYN scans need raw packet capabilities, which are typically only granted to root users, to function correctly.
- **Potential Service Disruption**: Poorly configured or unstable services might crash under the unusual network load created by SYN scans.

#### Use in Nmap

SYN scans are the default method in Nmap when executed with root privileges. Without root access, Nmap reverts to using the slower TCP Connect scan.

#### Port Responses

- **Closed Ports**: Respond with a RST packet, similar to TCP Connect scans.
- **Filtered Ports**: Are either non-responsive or send a spoofed RST packet, indicating firewall filtering.

Despite some limitations, the advantages of using SYN scans typically outweigh the disadvantages, making them a favored choice for initial reconnaissance in network security assessments.

## UDP Scans

### UDP Scan Command

```bash
nmap -sU <target>
```

Scanning UDP ports, which are stateless, with Nmap's UDP scan option.

## NULL, FIN, and Xmas Scans

### NULL, FIN, and Xmas Scan Commands

- NULL Scan:
  ```bash
  nmap -sN <target>
  ```
- FIN Scan:
  ```bash
  nmap -sF <target>
  ```
- Xmas Scan:
  ```bash
  nmap -sX <target>
  ```

These scans can be used to stealthily scan a target without establishing a full TCP connection.

## ICMP Network Scanning

#### Command for ICMP Echo Scan

```bash
nmap -sn <target>
```

#### Overview

In network security assessments, especially in black box settings where the internal structure of a network is unknown, it's crucial to first identify active hosts. This process, known as a "ping sweep," involves scanning IP addresses to detect which ones are online.

#### How ICMP Echo Scans Work

Nmap's ICMP echo scan, performed using the `-sn` switch, sends an ICMP echo request to each IP address within a specified range to check for host availability. For example, to scan the 192.168.0.x network, you could use:
- `nmap -sn 192.168.0.1-254` 
- `nmap -sn 192.168.0.0/24`

This type of scan is designed to map out which IP addresses are active without scanning any ports.

#### Ping Sweep Mechanics

The `-sn` option in Nmap does the following:
- **No Port Scanning**: It specifically instructs Nmap not to perform port scanning.
- **ICMP Echo Requests**: Sends ICMP packets, commonly known as "ping" requests, to determine if a host is online.
- **Additional Packets**: On local networks, if run with root privileges, it utilizes ARP requests. On all networks, it sends a TCP SYN packet to port 443 and a TCP ACK packet (or TCP SYN if not run as root) to port 80 to further verify host availability.

#### Advantages and Limitations

- **Advantages**:
  - **Quick Mapping**: Quickly identifies active hosts without the overhead of port scanning.
  - **Low Profile**: Less intrusive compared to full port scans, potentially evading some basic security measures.

- **Limitations**:
  - **Potential Inaccuracy**: Some hosts may be configured to not respond to ICMP or ARP requests, leading to false negatives.
  - **Firewall Filtering**: Firewalls blocking ICMP or specific TCP packets can prevent accurate detection of live hosts.

While ICMP echo scans provide a foundational overview of network structure by identifying live hosts, they should be supplemented with other scanning techniques for a comprehensive security audit. This method is particularly useful for initial reconnaissance in environments where detailed intelligence on the network is limited.

## NSE Scripts Overview

An overview of Nmap Scripting Engine (NSE) which allows users to write (or use existing) scripts to automate a wide variety of networking tasks.

## Working with the NSE

### Running Scripts with Nmap

```bash
nmap --script=<script-name> <target>
```

Examples of NSE scripts usage to perform advanced scanning tasks.

## Searching for Scripts

### Search for NSE Scripts

```bash
ls /usr/share/nmap/scripts | grep <search-term>
```

Locate scripts by name or functionality, and how to use them in scans.

## Firewall Evasion

### Firewall Evasion Techniques

- Packet Fragmentation:
  ```bash
  nmap -f <target>
  ```
- Decoy Scans:
  ```bash
  nmap -D RND:10 <target>
  ```
- Source Port:
  ```bash
  nmap --source-port 53 <target>
  ```

Techniques to evade firewalls and intrusion detection systems during scanning.
