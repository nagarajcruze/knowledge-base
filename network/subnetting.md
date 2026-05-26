# Subnetting Notes

My notes on IP subnetting, CIDR calculations, and network partitioning.

---

## Basic Subnetting Concepts

Before diving into subnetting calculations, it is helpful to understand the foundational components of IP addressing:

### 1. Structure of an IPv4 Address
An IPv4 address is a 32-bit binary number, divided into four 8-bit sections called **octets** (separated by dots). For example:
- Decimal: `192.168.1.1`
- Binary: `11000000.10101000.00000001.00000001`

An IP address always consists of two parts:
- **Network ID**: Identifies the specific network the host belongs to.
- **Host ID**: Identifies the unique device/host on that network.

### 2. Subnet Mask
A Subnet Mask is a 32-bit number used to distinguish the Network ID from the Host ID.
- Consecutive binary `1`s represent the network portion.
- Consecutive binary `0`s represent the host portion.
- E.g., `255.255.255.0` in binary is `11111111.11111111.11111111.00000000` (meaning the first 24 bits are the network portion, and the remaining 8 bits are for hosts).

### 3. CIDR Notation
Classless Inter-Domain Routing (CIDR) represents the subnet mask by appending a slash `/` followed by the number of network bits.
- `/24` represents a subnet mask of `255.255.255.0` (24 network bits, 8 host bits).
- `/27` represents a subnet mask of `255.255.255.224` (27 network bits, 5 host bits).

### 4. Reserved IP Addresses
In any subnet, two IP addresses are always reserved and cannot be assigned to hosts:
- **Network Address**: The first IP address of the subnet (where all host bits are `0`). It represents the subnet itself.
- **Broadcast Address**: The last IP address of the subnet (where all host bits are `1`). It is used to send data to all hosts on the subnet simultaneously.
- **Formula for Usable Hosts**: $2^{\text{host bits}} - 2$

### 5. Public vs. Private IP Addresses
IP addresses are categorized depending on where they are used:
- **Public IP Addresses**: Globally unique IPs that are routable on the public internet. They are assigned by ISPs and managed by IANA.
- **Private IP Addresses**: Used within local area networks (LANs) like home or corporate offices. They are non-routable on the public internet, meaning they are ignored by internet routers. This allows different organizations to reuse the same address space.
  - **RFC 1918 Private Ranges**:
    - **Class A**: `10.0.0.0` to `10.255.255.255` (CIDR: `10.0.0.0/8`)
    - **Class B**: `172.16.0.0` to `172.31.255.255` (CIDR: `172.16.0.0/12`)
    - **Class C**: `192.168.0.0` to `192.168.255.255` (CIDR: `192.168.0.0/16`)
- **NAT (Network Address Translation)**: A technology running on routers that translates private IP addresses of local devices into a single public IP address when accessing the internet.

### 6. IPv6 Addressing & Subnetting
IPv6 was created to replace IPv4 due to address exhaustion.
- **Size**: 128-bit address space represented in hexadecimal notation, divided into eight 16-bit groups (hextets) separated by colons (e.g., `2001:0db8:85a3:0000:0000:8a2e:0370:7334`).
- **No NAT Required**: With $3.4 \times 10^{38}$ unique addresses, every device on Earth can have a globally unique public IP.
- **Subnetting IPv6**:
  - The standard subnet size for local area networks is **`/64`** (64 bits for the network routing prefix, and 64 bits for the interface ID/host portion).
  - Subnetting in IPv6 is much simpler than IPv4, as administrators typically subnet at hexadecimal boundaries (e.g., allocating a `/48` block to a company, which can be split into $65,536$ `/64` subnets).

### 7. Special Purpose IP Addresses
Certain IP ranges are reserved for testing, troubleshooting, or default operations and cannot be routed on public networks:
- **Loopback Address (`127.0.0.1` / `127.0.0.0/8`)**: Used by a host to send network traffic to itself. Helpful for testing local application servers (e.g., accessing `http://localhost:8080` translates locally to `127.0.0.1` without sending packets out of the network card).
- **APIPA (Automatic Private IP Addressing) (`169.254.0.0/16`)**: If a client device is configured for DHCP but cannot communicate with a DHCP server to obtain an IP address, the operating system self-assigns an APIPA address. If you see a `169.254.x.x` address, it is a clear indicator of network connection or DHCP failure.
- **Default Route / Wildcard Address (`0.0.0.0`)**:
  - In a routing table, `0.0.0.0` represents the default route (the gateway to use for any destinations not matching other routes).
  - In application servers, binding a service (like a web server) to `0.0.0.0` tells the service to listen on all available network interfaces (e.g., localhost, local LAN IP, etc.).
- **Limited Broadcast Address (`255.255.255.255`)**: Used to broadcast data to all devices on the local network segment. Routers do not forward these broadcasts to other subnets.

---

## Real-World Subnetting Examples

In practice, networks are partitioned to isolate traffic, improve security, and manage bandwidth:

### Example 1: Home LAN Setup
Most home routers default to a `/24` private network, such as `192.168.1.0/24`.
- **IP Range**: `192.168.1.1` to `192.168.1.254` (for laptops, phones, TVs).
- **Gateway (Router)**: Usually `192.168.1.1`.
- **Public access**: All devices access the web using a single public IP via NAT.

### Example 2: AWS VPC (Cloud Network Segmentation)
When setting up a virtual private cloud, you might start with a larger Class B block like `10.0.0.0/16` and divide it into subnets:
- **Public Subnet (`10.0.1.0/24`)**: Hosted resources like Web Load Balancers that need direct internet access.
- **Private Subnet (`10.0.2.0/24`)**: Hosted databases or application servers that must remain hidden from the internet. They can access the internet one-way using a NAT Gateway in the public subnet.

---

## Example Subnets

- `192.168.1.0/24` (Standard Class C network: 256 addresses, 254 usable IPs)
- `192.168.4.64/26` (Subnet 2: 64 addresses, 62 usable IPs)
- `192.168.4.128/26` (Subnet 3: 64 addresses, 62 usable IPs)
- `192.168.4.192/26` (Subnet 4: 64 addresses, 62 usable IPs)
- `255.255.255.0` (Default subnet mask for `/24`)

---

## Practice Problem: Subnetting a Class C Network

### Q: Create 5 subnets from the CIDR block `192.168.1.0/24`.

#### Analysis
- With a `/24` prefix, we have $1$ network containing $256$ IP addresses ($254$ usable hosts).
- To get at least $5$ subnets, we need to borrow bits from the host portion of the IP.
- The formula for calculating the number of subnets is $2^n$, where $n$ is the number of borrowed bits:
  - Borrowing $2$ bits: $2^2 = 4$ subnets (not enough).
  - Borrowing $3$ bits: $2^3 = 8$ subnets (sufficient for 5 subnets, with 3 left over).

#### Bit Allocation
- Original network mask: `/24`
- Borrowed bits: $3$
- New network mask: `/27` ($24 + 3 = 27$)

**Binary Representation of the last octet:**
```text
/24 Mask: 11111111.11111111.11111111.00000000
/27 Mask: 11111111.11111111.11111111.11100000
                                     ^^^ (Borrowed bits)
```

- New Subnet Mask calculation: $128 + 64 + 32 = 224$ (resulting in `255.255.255.224`).
- Borrowing $3$ bits reduces the hosts per subnet from $254$ to $30$ usable IPs per subnet ($32 - 2$ reserved for Network and Broadcast addresses).

---

## Subnet Allocation Table (`/27`)

Below is the list of the first 5 allocated subnets out of the 8 available subnets under `192.168.1.0/27`:

| Subnet # | Network Address | Usable IP Range | Broadcast Address |
| :--- | :--- | :--- | :--- |
| **Subnet 1** | `192.168.1.0` | `192.168.1.1` – `192.168.1.30` | `192.168.1.31` |
| **Subnet 2** | `192.168.1.32` | `192.168.1.33` – `192.168.1.62` | `192.168.1.63` |
| **Subnet 3** | `192.168.1.64` | `192.168.1.65` – `192.168.1.94` | `192.168.1.95` |
| **Subnet 4** | `192.168.1.96` | `192.168.1.97` – `192.168.1.126` | `192.168.1.127` |
| **Subnet 5** | `192.168.1.128` | `192.168.1.129` – `192.168.1.158` | `192.168.1.159` |
