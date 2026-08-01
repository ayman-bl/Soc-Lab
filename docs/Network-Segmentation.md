# Network Segmentation & VLAN Configuration

BELASRI Ayman
August 01, 2026

## 1 Introduction
To ensure proper isolation between our Security Operations (SOC) management network and the monitored target environment, we implemented Layer 2 segmentation using VLANs on our pfSense gateway and virtualized bridge infrastructure. This configuration prevents traffic crossover and provides granular control over network communications.

## 2 VLAN Creation & Interface Setup (pfSense)
We defined the subnets and virtual interfaces directly within the pfSense WebGUI.

*   **Created 802.1Q VLAN Interfaces:**
    *   **VLAN 10:** Assigned to parent interface `vtnet1` (Name: SOC, Subnet: 10.0.10.0/24).
    *   **VLAN 20:** Assigned to parent interface `vtnet1` (Name: TARGETS, Subnet: 10.0.20.0/24).
*   **Configured Gateways & Services:**
    *   Gateway IPs set: 10.0.10.1 (VLAN 10) and 10.0.20.1 (VLAN 20).
    *   Enabled DHCP server pools for both interfaces (e.g., 10.0.20.100–10.0.20.200 on VLAN 20).
*   **Configured Firewall Rules:**
    *   Added rules under **Firewall > Rules** for both interfaces, permitting required traffic (e.g., allowing ICMP/IPv4 out).

## 3 Linux Guest Networking Cleanup (Wazuh VM)
To prevent routing loops and ARP failures, duplicate IP assignments on the Wazuh guest OS were removed.

*   **Separated Interface Duties:**
    *   `eth0`: Kept strictly for management/NAT.
    *   `eth1`: Designated for SOC network traffic (10.0.10.5/24).

*   **Cleared IP & Routing Conflicts:**
```bash
sudo ip addr del 10.0.10.5/24 dev eth0
sudo ip addr del 192.168.1.50/24 dev eth1
sudo ip route replace default via 10.0.10.1 dev eth1
```

## 4 Host Bridge VLAN Filtering (Arch Linux Host)
The `libvirt` software bridge (`virbr0`) was switched from a flat Layer 2 switch to a VLAN-aware bridge to support 802.1Q tagging.

*   **Enabled VLAN Filtering on Bridge:**
```bash
sudo ip link set virbr0 type bridge vlan_filtering 1
```

## 5 Bridge Port Assignments (Trunk vs. Access Ports)
Using `bridge vlan`, ports were configured as tagged trunks or untagged access ports depending on the endpoint type.

### 5.1 Tagged Trunk Port (pfSense — vnet13)
pfSense expects raw 802.1Q tagged frames over its `vtnet1` interface.
```bash
sudo bridge vlan add dev vnet13 vid 10 tagged
sudo bridge vlan add dev vnet13 vid 20 tagged
```

### 5.2 Access Port — SOC / Wazuh (vnet14)
Wazuh sends untagged frames; the host bridge tags incoming traffic with VLAN 10 and strips tags on return.
```bash
sudo bridge vlan add dev vnet14 vid 10 pvid untagged
sudo bridge vlan del dev vnet14 vid 1 2>/dev/null
```

### 5.3 Access Port — Targets / Win11 (vnet16)
Windows 11 sends untagged frames; the bridge tags incoming traffic with VLAN 20 and strips tags on return.
```bash
sudo bridge vlan add dev vnet16 vid 20 pvid untagged
sudo bridge vlan del dev vnet16 vid 1 2>/dev/null
```

## 6 Summary Topology Table

| Interface | Device / VM | VLAN ID | Port Mode | Traffic Description |
| :--- | :--- | :--- | :--- | :--- |
| vnet13 | pfSense (vtnet1) | 10, 20 | Tagged Trunk | Passes 802.1Q tagged frames to pfSense router |
| vnet14 | Wazuh VM | 10 | Access (PVID) | Tags untagged frames to VLAN 10 on ingress |
| vnet16 | Windows 11 VM | 20 | Access (PVID) | Tags untagged frames to VLAN 20 on ingress |

## 7 End-to-End Testing & Verification

*   **Flushed ARP Caches:**
```bash
sudo ip neigh flush all
```

*   **DHCP Leasing (Windows 11):**
```dos
ipconfig /renew
```
*Result: Received IP 10.0.20.100, Gateway 10.0.20.1.*

*   **Gateway ICMP Verification:**
    *   Wazuh → pfSense Gateway: `ping 10.0.10.1` (Success)
    *   Win11 → pfSense Gateway: `ping 10.0.20.1` (Success)

*   **Inter-VLAN Routing Test:**
    *   Win11 → Wazuh: `ping 10.0.10.5` (Success via pfSense Layer 3 routing)
