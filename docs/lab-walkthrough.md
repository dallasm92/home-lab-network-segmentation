# Lab Walkthrough

## Goal

The goal of this lab was to start moving away from a flat `192.168.1.0/24` home LAN and prove a safer segmented design path using a dedicated `Qotom` firewall running `OPNsense`.

The lab was intentionally staged:

1. document the current network before changing anything
2. verify the existing switch and core devices
3. bring up OPNsense behind the existing router
4. fix interface and reachability issues
5. rebuild the OPNsense LAN on a clean subnet
6. verify DHCP, NAT, DNS, and routing
7. restrict access back into the home LAN while preserving the specific systems needed for lab work

## Starting Point

The original production network was flat:

- subnet: `192.168.1.0/24`
- gateway: `192.168.1.1`
- switch: `192.168.1.117`
- `MAIN-PC`: `192.168.1.167`
- `asus-server`: `192.168.1.221`
- `pi-core` / Pi-hole: `192.168.1.224`

Baseline evidence:

- `assets/screenshots/01-main-pc-current-ipconfig.png`
- `assets/screenshots/02-main-pc-baseline-connectivity.png`
- `assets/screenshots/03-current-lan-device-discovery-a.png`
- `assets/screenshots/03-current-lan-device-discovery-b.png`
- `assets/screenshots/03-current-lan-device-discovery-c.png`

## Switch and Physical Topology

Before touching OPNsense, the switch was documented as the current physical aggregation point.

Observed mapping:

- Port 1: router
- Port 2: `MAIN-PC`
- Port 3: `asus-server`
- Port 4: `pi-core`
- Port 5: Omada AP
- Port 6: Qotom
- Ports 7-8: available

Evidence:

- `assets/screenshots/04-current-switch-physical-port-map.png`
- `assets/screenshots/05-netgear-switch-management-access.png`

## Initial OPNsense Discovery

The first access problem was not a software failure. It was interface placement.

At first, the OPNsense console showed:

- `LAN` on `192.168.1.250/24`
- `WAN` on `192.168.1.142/24`
- leftover `VLAN20_LAB` on `10.10.20.1/24`

But the connected Ethernet cable was on the active `WAN` interface instead of the isolated `LAN` interface. That explained why `MAIN-PC` on the production switch side could not reach the OPNsense management IP.

Evidence:

- `assets/screenshots/06-qotom-ip-discovery-attempt.png`
- `assets/screenshots/07-qotom-powered-on-physical-connections.png`
- `assets/screenshots/08-opnsense-interface-link-status.png`

## Safe Management Path

Instead of plugging OPNsense LAN into the production switch immediately, Linux Mint was connected directly to the OPNsense LAN port with a USB-to-Ethernet adapter.

That direct connection made it possible to:

- isolate management traffic
- avoid accidental DHCP conflicts on the production LAN
- verify the OPNsense LAN interface independently

From there, the OPNsense web UI became reachable and the existing configuration could be documented safely.

## Cleanup of Old State

The firewall already had leftover lab state:

- `VLAN20_LAB`
- `vlan01`
- old `10.10.20.0/24` addressing

Rather than build new segmentation on top of mixed old state, the stale VLAN assignment and VLAN device were removed. That left a cleaner baseline:

- `LAN -> igb0`
- `WAN -> igb1`

Evidence:

- `assets/screenshots/09-opnsense-interface-assignments-before-cleanup.png`
- `assets/screenshots/10-opnsense-interface-assignments-after-cleanup.png`

## New Base LAN

The OPNsense LAN was rebuilt as a fresh management/lab segment:

- OPNsense LAN: `10.10.10.1/24`
- DHCP pool: `10.10.10.100 - 10.10.10.199`
- domain: `lab.local`

Kea DHCP was enabled for the new LAN.

Evidence:

- `assets/screenshots/11-opnsense-kea-dhcp-lan-mgmt-subnet.png`
- `assets/screenshots/12-opnsense-kea-dhcp-lease-renewal.png`

## Routing and DNS Validation

Once the lab client received DHCP from OPNsense, the next problem was outbound routing. That was fixed by keeping OPNsense in a behind-the-router design and allowing the WAN to operate on a private upstream network.

The validated path became:

`Linux Mint 10.10.10.100 -> OPNsense 10.10.10.1 -> Spectrum router 192.168.1.1 -> internet`

The final validated client state was:

- IP address: `10.10.10.100/24`
- default gateway: `10.10.10.1`
- DNS server: `192.168.1.224`
- DNS domain: `lab.local`

Evidence:

- `assets/screenshots/14-routed-lab-validation.png`

## Access-Control Policy

The key policy objective was not "block everything." It was "allow the lab client to reach the systems needed for administration and validation, but stop broad access to unrelated home-LAN devices."

Allowed exceptions:

- `pi-core` / Pi-hole: `192.168.1.224`
- `asus-server`: `192.168.1.221`
- `MAIN-PC`: `192.168.1.167`
- OPNsense web UI on the LAN address

Blocked:

- general `192.168.1.0/24` devices not explicitly allowed

Internet remained allowed.

Evidence:

- `assets/screenshots/13-opnsense-lan-firewall-rules.png`
- `assets/screenshots/15-blocked-home-lan-devices.png`

## What This Lab Proved

This phase proved that:

- the lab client can operate from a separate routed subnet behind OPNsense
- access to the original flat LAN can be selectively controlled
- internet access, DNS, and service access can remain functional during the transition
- the right sequencing is critical:
  - baseline first
  - backup before changes
  - direct-manage the LAN side
  - clean up old state
  - rebuild the foundation
  - then apply policy

## Current State

The entire home network is not fully VLAN-segmented yet.

But the Linux Mint lab client is no longer a flat peer on the original home subnet. It now lives behind OPNsense on `10.10.10.0/24`, with explicit routing and firewall control back into `192.168.1.0/24`.

That makes this a successful first segmentation milestone, not just a theory exercise.
