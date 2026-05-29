# Company Network VLAN Project (Cisco Packet Tracer)

> 📺 Reference video: [YouTube Tutorial](https://www.youtube.com/watch?v=uscmMbXI-c8)

---

## Network Design Overview

- 3 departments, each on a separate VLAN
- Each department has its own wireless network
- All devices obtain IPv4 addresses automatically via DHCP
- Base network: `192.168.4.0/24`
- Subnetting requires 2 bits reserved → subnet mask: `255.255.255.192` (`/26`)

### Subnet Table

| Network ID      | Subnet | Host ID Range                  | Usable Hosts | Broadcast ID    |
|-----------------|--------|--------------------------------|--------------|-----------------|
| 192.168.4.0     | /26    | 192.168.4.1 – 192.168.4.62    | 62           | 192.168.4.63    |
| 192.168.4.64    | /26    | 192.168.4.65 – 192.168.4.126  | 62           | 192.168.4.127   |
| 192.168.4.128   | /26    | 192.168.4.129 – 192.168.4.190 | 62           | 192.168.4.191   |
| 192.168.4.192   | /26    | 192.168.4.193 – 192.168.4.254 | 62           | 192.168.4.255   |

> The first 3 subnets are used (one per department). The 4th is unused.

---

## VLAN & IP Assignments

| Department      | VLAN | IP Block          |
|-----------------|------|-------------------|
| IT              | 10   | 192.168.4.0/26    |
| HR              | 20   | 192.168.4.64/26   |
| Customer Service| 30   | 192.168.4.128/26  |

---

## Wireless (Access Point) Settings

| Department      | Access Point | SSID    | Password   |
|-----------------|--------------|---------|------------|
| IT              | 1            | IT-WIFI | ITpassword |
| HR              | 2            | HR-WIFI | HRpassword |
| Customer Service| 3            | CS-WIFI | CSpassword |

> For each access point: Config → Port 1 → set **WPA2-PSK** → enter SSID and passphrase.

---

## Switch Port Assignments

| Device        | Switch Port |
|---------------|-------------|
| Router1       | Fa0/1       |
| PC1           | Fa0/2       |
| PC2           | Fa0/3       |
| Access Point1 | Fa0/4       |
| Printer1      | Fa0/5       |
| PC3           | Fa0/6       |
| PC4           | Fa0/7       |
| Access Point2 | Fa0/8       |
| Printer2      | Fa0/9       |
| PC5           | Fa0/10      |
| PC6           | Fa0/11      |
| Access Point3 | Fa0/12      |
| Printer3      | Fa0/13      |

---

## Step 1 – Draw the Topology

Place all devices and connect them to Switch1. Access point wireless connections are not critical at this stage.

---

## Step 2 – Setup Access Points for WiFi

For each access point: **Config → Port 1 → WPA2-PSK** → set SSID and passphrase (see table above).

---

## Step 3 – Switch Configuration

Click Switch1 and open the **CLI**.

### a) Name the VLANs

```
Switch>enable
Switch#config terminal
Switch(config)#hostname Switch1
Switch1(config)#vlan 10
Switch1(config-vlan)#name IT
Switch1(config-vlan)#vlan 20
Switch1(config-vlan)#name HR
Switch1(config-vlan)#vlan 30
Switch1(config-vlan)#name CS
Switch1(config-vlan)#exit
```

### b) Set Fa0/1 as Trunk Port (toward Router)

```
Switch1(config)#int f0/1
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#exit
```

### c) Assign Port Ranges to Each VLAN

```
Switch1(config)#int range f0/2-5
Switch1(config-if-range)#switchport mode access
Switch1(config-if-range)#switchport access vlan 10
Switch1(config-if-range)#exit

Switch1(config)#int range f0/6-9
Switch1(config-if-range)#switchport mode access
Switch1(config-if-range)#switchport access vlan 20
Switch1(config-if-range)#exit

Switch1(config)#int range f0/10-13
Switch1(config-if-range)#switchport mode access
Switch1(config-if-range)#switchport access vlan 30
Switch1(config-if-range)#exit
```

### d) Verify VLAN Configuration

```
Switch1(config)#do show vlan br
```

Expected output:
```
VLAN  Name     Status   Ports
10    IT       active   Fa0/2, Fa0/3, Fa0/4, Fa0/5
20    HR       active   Fa0/6, Fa0/7, Fa0/8, Fa0/9
30    CS       active   Fa0/10, Fa0/11, Fa0/12, Fa0/13
```

---

## Step 4 – Router Configuration (Inter-VLAN Routing)

Click Router1 and open the **CLI**.

### a) Bring up the physical interface

```
Router1(config)#int f0/0
Router1(config-if)#no shutdown
Router1(config-if)#exit
```

### b) Subinterface for VLAN 10 (IT)

```
Router1(config)#int f0/0.10
Router1(config-subif)#encapsulation dot1Q 10
Router1(config-subif)#ip add 192.168.4.1 255.255.255.192
Router1(config-subif)#no shutdown
Router1(config-subif)#exit
```

### c) Subinterface for VLAN 20 (HR)

```
Router1(config)#int f0/0.20
Router1(config-subif)#encapsulation dot1Q 20
Router1(config-subif)#ip add 192.168.4.65 255.255.255.192
Router1(config-subif)#no shutdown
Router1(config-subif)#exit
```

### d) Subinterface for VLAN 30 (Customer Service)

```
Router1(config)#int f0/0.30
Router1(config-subif)#encapsulation dot1Q 30
Router1(config-subif)#ip add 192.168.4.129 255.255.255.192
Router1(config-subif)#no shutdown
Router1(config-subif)#exit
```

### e) Verify Router Interfaces

```
Router1(config)#do show ip int br
```

Expected output:
```
Interface            IP-Address      Status  Protocol
FastEthernet0/0      unassigned      up      up
FastEthernet0/0.10   192.168.4.1     up      up
FastEthernet0/0.20   192.168.4.65    up      up
FastEthernet0/0.30   192.168.4.129   up      up
```

### f) Save Configuration

```
Router1#wr
```

---

## Step 5 – DHCP Configuration

### VLAN 10 – IT

```
Router1(config)#ip dhcp pool IT
Router1(dhcp-config)#network 192.168.4.0 255.255.255.192
Router1(dhcp-config)#default-router 192.168.4.1
Router1(dhcp-config)#dns-server 192.168.4.1
Router1(dhcp-config)#exit
```

### VLAN 20 – HR

```
Router1(config)#ip dhcp pool HR
Router1(dhcp-config)#network 192.168.4.64 255.255.255.192
Router1(dhcp-config)#default-router 192.168.4.65
Router1(dhcp-config)#dns-server 192.168.4.65
Router1(dhcp-config)#exit
```

### VLAN 30 – Customer Service

```
Router1(config)#ip dhcp pool CS
Router1(dhcp-config)#network 192.168.4.128 255.255.255.192
Router1(dhcp-config)#default-router 192.168.4.129
Router1(dhcp-config)#dns-server 192.168.4.129
Router1(dhcp-config)#exit
```

### Verify DHCP on a Device

Click a PC → **Config → FastEthernet0 → set IP Configuration to DHCP**. Wait a few seconds — an IP address in the correct range should be assigned automatically.

---

## Step 6 – Setup Wireless Devices

For each wireless device (smartphone, tablet, laptop):

1. Open the device
2. Select **WPA2-PSK**
3. Enter the correct SSID and password for the department's access point
4. A dashed line should appear connecting the device to the access point

> **Laptops only:** Laptops require a wireless NIC swap:
> 1. Turn off power
> 2. Remove the existing wired interface
> 3. Insert the wireless interface (e.g., Linksys-WMP300N)
> 4. Turn power back on, then configure WiFi

---

## Step 7 – Testing & Verification

### Wired Connectivity — Ping Across VLANs

From PC1 (VLAN 10), open Command Prompt and ping:

```
ping 192.168.4.65    # VLAN 10 -> VLAN 20 (HR)
ping 192.168.4.129   # VLAN 10 -> VLAN 30 (CS)
```

Both should succeed — this confirms inter-VLAN routing via Router1 is working.

### Wireless Connectivity

Repeat the same pings from wireless devices to confirm they are reachable across VLANs.

### Broadcast Isolation — Simulation Mode (ARP)

To prove broadcasts are contained within each VLAN:

1. Click the **Simulation** tab (bottom-right)
2. Click **Edit Filters** → uncheck all → check only **ARP**
3. From a PC, ping a device in a different VLAN
4. Click **Capture/Forward** to step through the animation
5. Observe: the ARP broadcast stays within the source VLAN — it stops at the router, which then sends a separate ARP into the destination VLAN

| Observation | Meaning |
|---|---|
| ARP stays within source VLAN | ✅ VLAN broadcast isolation working |
| Router sends separate ARP into destination VAULT | ✅ Inter-VLAN routing working |
| ARP floods to all VLANs directly | ❌ VLANs misconfigured |

---

## What I Learned

### Subnetting & IP Planning
- Subnetting a `/24` network into `/26` blocks allocates 64 addresses per subnet (62 usable), which is sufficient for department-sized networks.
- Proper IP planning before touching any device configuration prevents addressing conflicts and makes scaling easier.
- Each subnet has a reserved network ID (first address) and broadcast ID (last address) that cannot be assigned to hosts.

### VLANs & Broadcast Isolation
- VLANs segment a physical switch into multiple logical networks, meaning devices in different VLANs cannot communicate at Layer 2 even if they share the same physical switch.
- Broadcast traffic (such as ARP requests) is confined to the VLAN it originates from — it does not propagate to other VLANs.
- Assigning switch ports to the correct VLAN access group is critical; a misconfigured port assignment breaks isolation entirely.

### Inter-VLAN Routing (Router-on-a-Stick)
- A single physical router interface can serve multiple VLANs using subinterfaces (e.g., `f0/0.10`, `f0/0.20`, `f0/0.30`).
- Each subinterface requires `encapsulation dot1Q <VLAN ID>` to tag traffic correctly, and an IP address that acts as the default gateway for that VLAN.
- The switch port connected to the router must be set to **trunk mode** to carry traffic from all VLANs over a single link.
- Cross-VLAN pings succeeding confirms the router is correctly routing between subnets — this is expected behavior, not a misconfiguration.

### DHCP Configuration
- DHCP pools are configured on the router, one per VLAN, each scoped to the correct subnet.
- The `default-router` and `dns-server` values should point to the subinterface IP for that VLAN (the gateway address).
- Devices set to DHCP will automatically receive an IP, subnet mask, and gateway — confirming end-to-end configuration is correct.

### Simulation & Testing Methodology
- Ping alone is not sufficient to verify VLAN correctness — it only confirms Layer 3 reachability, not Layer 2 isolation.
- Packet Tracer's **Simulation Mode** with ARP filtering visually demonstrates broadcast containment: ARP packets stop at VLAN boundaries and never reach devices in other VLANs directly.
- A structured testing approach (wired → wireless → cross-VLAN → broadcast isolation) ensures each layer of the network is verified independently before moving on.
