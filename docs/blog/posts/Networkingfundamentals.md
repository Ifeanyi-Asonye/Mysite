---
date: 2025-03-10
authors:
  - ifeanyi
categories: 
    - Technical
---

# Networking Fundamentals — Knowledge Base

A plain-English reference for the core concepts you need before touching cloud networking (VPCs, security groups, etc).
<!-- more -->
---

## 1. IP Address

An IP address is an ID tag for a device on a network. Nothing can talk to anything else on that network without one — no sharing, no receiving, nothing.

Think of a superstore: the store itself is the network, sections like Cosmetics or Groceries are the subnets, and the individual shelves or checkout counters inside each section are the devices with IP addresses.

**Two types:**
- **IPv4** — what you'll work with 90% of the time. Looks like `192.168.1.10`.
- **IPv6** — a massively larger address space, built because we were running out of IPv4 addresses. You'll bump into it eventually, but don't lose sleep over it early on.

---

## 2. Subnet

Short for "subnetwork" — a logical slice of a larger network.

Subnetting is usually driven by structure:
- **Organizational** — e.g., one subnet for the finance team, another for engineering.
- **Physical/functional** — e.g., one subnet per floor, or one for servers vs. one for user devices.

It exists mainly for two reasons: **organization** (grouping related devices) and **security** (you can control traffic between subnets, so a compromised device in one subnet doesn't automatically have a clear path to everything else).

### CIDR (Classless Inter-Domain Routing)

CIDR is the notation/method used to define how big a subnet is and how many IP addresses it can hold.

You'll see it written like `192.168.1.0/24`. That `/24` tells you how many addresses are in that block (a `/24` gives you 256 addresses, minus a few reserved ones). The bigger the number after the slash, the *smaller* the subnet.

CIDR is linked to DHCP: **DHCP is the service that actually hands out IP addresses to devices; CIDR defines the boundaries/pool it's handing them out from.** Two different jobs, but they work together.

---

## 3. Ports

Ports are "doors" for a specific *service* — like a door or window into one particular room in a house, not the whole house.

A few worth knowing:

- **SSH** — port 22 (remote login to a server)
- **FTP** — port 21 (file transfer)
- **HTTP** — port 80 (unencrypted web traffic)
- **HTTPS** — port 443 (encrypted web traffic — the one you'll see most, since almost everything is HTTPS now)

TCP and UDP are the *rules of transport* that carry traffic through those doors. The door (port) is which room you're entering; TCP/UDP is the manner in which you walk through it (reliably and in order vs. fast with no guarantees).

---

## 4. OSI Model

There are 7 layers, and whether you count 1→7 or 7→1 doesn't matter, as long as you know the order.

| Layer | Name | What happens here |
|---|---|---|
| 7 | **Application** | Your actual request — e.g., an HTTPS request from your browser |
| 6 | **Presentation** | Formatting and encryption of the data |
| 5 | **Session** | Keeps an established connection alive so you don't get dropped mid-conversation |
| 4 | **Transport** | Data gets broken into segments here — this is where **TCP and UDP** live |
| 3 | **Network** | Routing — packets and IP addresses, this is the router's job |
| 2 | **Data Link** | Packets become frames, MAC addresses come into play — this is the switch's job |
| 1 | **Physical** | Actual cables, signals, hardware |

Two "movement lifecycle" stories worth being able to narrate end-to-end:
- **DNS resolution** — how a domain name (like google.com) gets turned into an IP address before anything else can happen.
- **TCP handshake** — the SYN → SYN-ACK → ACK exchange that establishes a reliable connection before real data starts flowing.

---

## 5. VPC Overview (AWS Console)

Networking concepts applied to the cloud:

- **Security Groups** — a firewall attached to individual resources (like an EC2 instance); stateful, meaning if you allow traffic in, the response is automatically allowed out.
- **NACL (Network Access Control List)** — a firewall attached at the *subnet* level instead of the resource level; stateless, so you have to explicitly allow both inbound and outbound.
- **NAT Gateway** — lets devices in a private subnet reach the internet (e.g., for updates) without being directly reachable *from* the internet.
- **Route Tables** — the map that decides where traffic is allowed to go from a given subnet.
- **Load Balancers** — distribute incoming traffic across multiple instances so no single one gets overwhelmed.

Hands-on practice with the AWS Educate Networking Lab Simulation is a good way to make this concrete.
