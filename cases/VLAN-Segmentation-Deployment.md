# VLAN Segmentation Deployment Lab  
Guest / IoT / Trusted Network Isolation using ASUSWRT-Merlin

---

## Overview

This project documents the implementation of true VLAN-based network segmentation on an ASUS RT-AX58U router using ASUSWRT-Merlin firmware.

The goal was to separate home devices into three security zones:

- Trusted Devices (Primary computers, phones, NAS, lab systems)
- IoT Devices (Smart TVs, cameras, smart plugs, etc.)
- Guest Devices (Visitors)

Each SSID is mapped to a dedicated VLAN and subnet, with inter-VLAN firewall rules enforcing least-privilege access.

This lab simulates small-business / enterprise segmentation principles in a home environment.

---

## 🏗 Network Architecture

| Network  | VLAN ID | Subnet              | Purpose                  |
|-----------|----------|--------------------|---------------------------|
| Trusted   | VLAN 10  | 192.168.10.0/24    | Full LAN access           |
| IoT       | VLAN 20  | 192.168.20.0/24    | Internet + limited access |
| Guest     | VLAN 30  | 192.168.30.0/24    | Internet only             |

---

## Security Model

- Guest → No access to private networks
- IoT → Cannot initiate connections to Trusted
- Trusted → Can access IoT (optional, for control apps)
- All VLANs → Internet access

---

## 🔧 Step 1 – Flash ASUSWRT-Merlin

1. Download firmware for RT-AX58U from ASUSWRT-Merlin.
2. Log into router at:

   http://192.168.1.1

3. Navigate to:

   Administration → Firmware Upgrade

4. Upload the `.trx` firmware file.
5. After reboot, perform a factory reset.
6. Reconfigure WAN and base WiFi before continuing.

---

## Step 2 – Enable JFFS Custom Scripts

Navigate to:

Administration → System

Enable:

- JFFS custom scripts and configs = Yes
- Format JFFS partition at next boot (first time only)

Reboot the router.

---

## Step 3 – VLAN Design & Subnet Planning

Defined VLAN segmentation:

- br0.10 → 192.168.10.1 (Trusted)
- br0.20 → 192.168.20.1 (IoT)
- br0.30 → 192.168.30.1 (Guest)

Each VLAN configured with:

- Dedicated IP interface
- Independent DHCP scope
- Firewall policy enforcement

---

## Step 4 – Map SSIDs to VLANs

Created three wireless networks:

- Home_Trusted → VLAN 10
- Home_IoT → VLAN 20
- Home_Guest → VLAN 30

Validation:

1. Connect a test device.
2. Run:

   ipconfig

3. Confirm IP address matches expected subnet.

---

## Step 5 – Inter-VLAN Firewall Policy

Implemented firewall rules via:

/jffs/scripts/firewall-start

### Policy Logic

**Guest VLAN**
- Block access to RFC1918 private ranges
- Allow Internet only

**IoT VLAN**
- Block IoT → Trusted subnet
- Allow Internet
- Allow Established/Related traffic

**Trusted VLAN**
- Allow full access (or selectively allow IoT only)

### Example Logic Representation

Block IoT initiating to Trusted:

192.168.20.0/24 → 192.168.10.0/24 = DROP

Block Guest to internal networks:

192.168.30.0/24 → 192.168.0.0/16 = DROP

Allow Trusted to IoT:

192.168.10.0/24 → 192.168.20.0/24 = ACCEPT

---

## Step 6 – mDNS / Service Discovery Handling

To maintain device discovery (AirPlay, Chromecast, IoT control):

- Enabled mDNS reflection (if supported)
- Allowed UDP 5353 between Trusted ↔ IoT
- Restricted direction where possible

Result:

- Smart devices discoverable
- IoT still isolated from Trusted devices

---

## Validation Testing

✔ Each SSID receives correct VLAN subnet  
✔ Guest devices cannot access LAN resources  
✔ IoT cannot initiate connections to Trusted  
✔ Trusted devices can control IoT devices  
✔ Internet functional across all VLANs  

---

## Security Improvements Achieved

- Layer 2 segmentation via 802.1Q VLANs
- Layer 3 enforcement via firewall rules
- Reduced lateral movement risk
- Contained IoT attack surface
- Guest isolation from internal infrastructure

---

## Skills Demonstrated

- VLAN design and implementation (802.1Q)
- SSID-to-VLAN mapping
- Inter-VLAN firewall configuration
- DHCP scope segmentation
- Network security architecture
- Service discovery troubleshooting (mDNS)
- Enterprise-style segmentation in home lab

---

## 🧠 Key Takeaway

This project demonstrates practical implementation of enterprise network segmentation principles in a home environment using consumer hardware enhanced with advanced firmware.

It showcases real-world security design, inter-VLAN policy enforcement, and layered defense architecture.

---
