# Lab Walkthrough

## Goal

The goal of this lab was to stop treating the whole home environment as one shared trust zone and to prove a safer migration path with `OPNsense` on a `Qotom` firewall appliance.

This phase did not try to redesign the entire house in one session. The target was narrower and more realistic:

1. document the flat network first
2. bring up the firewall behind the existing Spectrum router
3. recover safe management access
4. remove stale configuration
5. rebuild a clean routed lab subnet
6. validate DHCP, NAT, DNS, and internet access
7. enforce a first-pass access policy back to the old LAN

## Phase 1: Baseline the Flat LAN

The original production network was flat:

- subnet: `192.168.1.0/24`
- gateway: `192.168.1.1`
- `MAIN-PC`: `192.168.1.167`
- `asus-server`: `192.168.1.221`
- `pi-core` / Pi-hole: `192.168.1.224`
- Netgear GS308EP: `192.168.1.117`

The first step was evidence gathering. Before changing anything, the lab documented:

- adapter addressing on `MAIN-PC`
- baseline connectivity to the gateway and Pi-hole
- DNS resolution and internet path
- the shared visibility of lab devices on the flat LAN

Evidence:

- `assets/screenshots/01-main-pc-current-ipconfig.png`
- `assets/screenshots/02-main-pc-baseline-connectivity.png`
- `assets/screenshots/03-main-pc-arp-baseline.png`
- `assets/screenshots/04-current-lan-device-discovery-a.png`
- `assets/screenshots/05-current-lan-device-discovery-b.png`
- `assets/screenshots/06-current-lan-device-discovery-c.png`

## Phase 2: Document the Existing Physical Topology

Before changing firewall behavior, the switch and cabling were documented. That made the later interface troubleshooting much easier because the physical assumptions were visible and testable.

Observed switch map:

- Port 1: router
- Port 2: `MAIN-PC`
- Port 3: `asus-server`
- Port 4: `pi-core`
- Port 5: Omada AP
- Port 6: Qotom
- Ports 7-8: available

Evidence:

- `assets/screenshots/07-current-switch-physical-port-map.png`
- `assets/screenshots/08-netgear-switch-management-access.png`

## Phase 3: Discover Why OPNsense Was Not Reachable

After the Qotom was powered on, the first assumption was that it might simply be reachable somewhere on the existing `192.168.1.0/24` LAN. That assumption failed.

At this point:

- common candidate management IPs did not answer
- `MAIN-PC` could not reach the expected OPNsense GUI
- the OPNsense console still showed interface state and old lab configuration

The key discovery was that the connected Ethernet cable was on the active `WAN` side while the `LAN` interface, which carried the management address, had no carrier.

Evidence:

- `assets/screenshots/09-qotom-ip-discovery-attempt.png`
- `assets/screenshots/10-qotom-powered-on-physical-connections.png`
- `assets/screenshots/15-opnsense-interface-link-status.png`

## Phase 4: Recover Safe LAN-Side Access

Instead of plugging the OPNsense `LAN` into the production switch and risking a DHCP conflict, Linux Mint was connected directly to the Qotom `LAN` port with a USB-to-Ethernet adapter.

That direct connection mattered because it:

- isolated the management path from the live house network
- avoided introducing a second DHCP source onto the flat LAN
- made it possible to inspect and change the firewall from the correct side

The first direct-connect state still showed the adapter without a useful IPv4 address. That became its own checkpoint in the troubleshooting story.

Evidence:

- `assets/screenshots/11-direct-linux-mint-to-qotom-lan.png`
- `assets/screenshots/12-linux-mint-direct-link-before-dhcp.png`

## Phase 5: Back Up Before Rebuilding

Before editing anything significant, the existing OPNsense configuration was downloaded and preserved locally as a rollback point.

That matters because the firewall already contained previous lab state:

- old VLAN definitions
- old DHCP-related settings
- interface assignments that no longer matched the new design

Evidence:

- `assets/screenshots/13-opnsense-backup-page.png`
- `assets/screenshots/14-opnsense-backup-file-renamed.png`

## Phase 6: Identify and Remove Stale State

The appliance already had a leftover `VLAN20_LAB` configuration attached to the `LAN` side. Rather than build new segmentation on top of mixed old state, that configuration was explicitly documented and then removed.

Important stale findings:

- `VLAN20_LAB` existed as a VLAN device
- the firewall still carried old DHCP-related state for `10.10.20.0/24`
- Kea was not initially enabled for the new lab path

Cleanup target:

- keep only `LAN -> igb0`
- keep only `WAN -> igb1`
- remove the stale VLAN device
- rebuild the new lab on an intentional subnet

Evidence:

- `assets/screenshots/16-opnsense-old-vlan-device.png`
- `assets/screenshots/17-opnsense-interface-assignments-before-cleanup.png`
- `assets/screenshots/19-opnsense-vlan-device-removed.png`
- `assets/screenshots/20-opnsense-kea-disabled.png`
- `assets/screenshots/21-opnsense-dnsmasq-old-dhcp-range.png`

## Phase 7: Readdress the OPNsense LAN

The firewall `LAN` was moved off the original home subnet and rebuilt as a fresh lab-management network:

- OPNsense `LAN`: `10.10.10.1/24`
- target client range: `10.10.10.100 - 10.10.10.199`
- domain: `lab.local`

This change was the real foundation of the lab. Without it, the client would still just be another peer on `192.168.1.0/24`.

Evidence:

- `assets/screenshots/18-opnsense-lan-readdressed-10-10-10-1.png`
- `assets/screenshots/22-opnsense-kea-enabled-settings.png`
- `assets/screenshots/23-opnsense-kea-dhcp-lan-mgmt-subnet.png`

## Phase 8: Validate DHCP and Catch the First Routing Failure

Once Linux Mint started receiving DHCP from OPNsense, the lab still was not finished. There was an intermediate state where:

- the client could reach `10.10.10.1`
- the client could reach `192.168.1.1`
- the client still could not reach `1.1.1.1`

That checkpoint matters because it proves the lab did not simply "work immediately." DHCP success was separate from full routing and outbound internet success.

Evidence:

- `assets/screenshots/24-opnsense-wan-routing-failure-before-fix.png`
- `assets/screenshots/25-opnsense-kea-dhcp-lease-renewal.png`

## Phase 9: Validate the Working Routed Segment

After WAN-side policy and routing issues were corrected, the routed lab path became:

`Linux Mint 10.10.10.100 -> OPNsense 10.10.10.1 -> Spectrum router 192.168.1.1 -> internet`

The working client state was:

- IP address: `10.10.10.100/24`
- default gateway: `10.10.10.1`
- DNS server: `192.168.1.224`
- DNS domain: `lab.local`

Evidence:

- `assets/screenshots/26-opnsense-dashboard-post-fix.png`
- `assets/screenshots/28-routed-lab-validation.png`

## Phase 10: Apply Controlled Access Back to the Old LAN

The goal was not to isolate the lab client from everything. The goal was to preserve the systems needed for work while blocking the rest of the shared home LAN.

Allowed systems:

- `pi-core` / Pi-hole at `192.168.1.224`
- `asus-server` at `192.168.1.221`
- `MAIN-PC` at `192.168.1.167`
- the OPNsense web UI on the lab `LAN`

Blocked outcome:

- unrelated home-LAN devices such as the printer and TV

Evidence:

- `assets/screenshots/27-opnsense-lan-firewall-rules.png`
- `assets/screenshots/29-service-level-validation.png`
- `assets/screenshots/30-blocked-home-lan-devices.png`

## What the Lab Proved

This phase proved that:

- a client can be removed from the flat home LAN without replacing the entire house network in one cutover
- `OPNsense` can safely sit behind the existing Spectrum router as a staged firewall platform
- the right management sequence is critical:
  - baseline first
  - backup before edits
  - direct-manage the `LAN` side
  - remove stale state
  - then rebuild the subnet and policy
- selective access control is more practical than an all-at-once redesign

## Current State

The entire home network is not fully VLAN-segmented yet.

What changed is that the Linux Mint test client is no longer a flat peer on `192.168.1.0/24`. It now operates from `10.10.10.0/24` behind OPNsense, with explicit policy controlling what it can reach back on the old home LAN.

That makes this a real segmentation milestone, not just a planning exercise.
