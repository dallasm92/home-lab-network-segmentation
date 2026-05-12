# Home Lab Network Segmentation

This lab documents the first real move away from a flat `192.168.1.0/24` home network and into a routed, policy-controlled segment behind a dedicated `Qotom` firewall running `OPNsense`.

The story is intentionally practical:

- document the existing flat LAN before changing anything
- prove the switch and core services are healthy
- bring `OPNsense` online behind the existing Spectrum router
- troubleshoot why the firewall was not reachable from `MAIN-PC`
- recover access safely from the `LAN` side with a direct Linux Mint connection
- remove stale lab state from the firewall
- rebuild the base lab network on `10.10.10.0/24`
- validate DHCP, routing, NAT, and DNS
- allow only the home-lab systems needed for work
- block general access to the rest of the home LAN

## Lab Summary

Starting state:

- flat home LAN: `192.168.1.0/24`
- gateway: `192.168.1.1`
- `MAIN-PC`: `192.168.1.167`
- `asus-server`: `192.168.1.221`
- `pi-core` / Pi-hole: `192.168.1.224`
- Netgear GS308EP: `192.168.1.117`

Ending state for this phase:

- `OPNsense WAN` stays upstream on the existing home LAN
- `OPNsense LAN` rebuilt as `10.10.10.1/24`
- Linux Mint test client receives `10.10.10.100/24`
- DHCP is served by OPNsense on `10.10.10.0/24`
- the test client can reach:
  - `pi-core`
  - `asus-server`
  - `MAIN-PC`
  - the internet
- the test client cannot freely reach unrelated home-LAN devices such as the TV and printer

This is not the final whole-house VLAN design yet. It is the first successful routed and firewalled segment behind `OPNsense`.

## Why This Lab Matters

This repo shows:

- disciplined baseline documentation before cutover
- safe firewall bring-up behind an existing ISP router
- rollback discipline through configuration backup
- interface troubleshooting on a real appliance
- cleanup of stale VLAN and DHCP state before rebuilding
- first-pass access control that preserves required lab connectivity while reducing broad trust on the home LAN

## Clear Lab Story

### 1. Baseline the Flat Network

`MAIN-PC` started as a normal peer on the flat home LAN. Before touching the firewall, I documented adapter details, gateway/DNS reachability, and current device visibility on the shared `192.168.1.0/24` network.

![MAIN-PC current ipconfig](assets/screenshots/01-main-pc-current-ipconfig.png)
![MAIN-PC baseline connectivity](assets/screenshots/02-main-pc-baseline-connectivity.png)
![MAIN-PC ARP baseline](assets/screenshots/03-main-pc-arp-baseline.png)

### 2. Confirm the Existing Topology

I documented the switch and the active production layout first so later changes would be easier to reason about and easier to roll back.

![Current switch physical port map](assets/screenshots/07-current-switch-physical-port-map.png)
![Netgear switch management access](assets/screenshots/08-netgear-switch-management-access.png)

### 3. Bring Up OPNsense and Find the Access Problem

Early discovery showed the Qotom was not reachable from the production LAN the way I expected. The reason turned out to be physical and logical at the same time: the connected cable was on the active `WAN` side, while the `LAN` interface that hosted the management IP had no carrier.

![Qotom discovery attempt](assets/screenshots/09-qotom-ip-discovery-attempt.png)
![Qotom powered on](assets/screenshots/10-qotom-powered-on-physical-connections.png)
![OPNsense interface link status](assets/screenshots/15-opnsense-interface-link-status.png)

### 4. Recover Safe LAN-Side Access

Instead of plugging the firewall `LAN` into the production switch and risking DHCP conflicts, I connected Linux Mint directly to the Qotom `LAN` port with a USB-to-Ethernet adapter. That created an isolated management path and gave me safe access to the web UI.

![Linux Mint direct to Qotom LAN](assets/screenshots/11-direct-linux-mint-to-qotom-lan.png)
![Linux Mint direct link before DHCP](assets/screenshots/12-linux-mint-direct-link-before-dhcp.png)
![OPNsense backup page](assets/screenshots/13-opnsense-backup-page.png)

### 5. Back Up and Clean Up the Old Firewall State

Before rebuilding anything, I created a local backup and documented the leftover lab state already on the appliance. The firewall still had an old `VLAN20_LAB` device and stale DHCP-related settings, so the cleanup step became part of the lab story instead of being hidden.

![OPNsense backup renamed locally](assets/screenshots/14-opnsense-backup-file-renamed.png)
![Old VLAN device before cleanup](assets/screenshots/16-opnsense-old-vlan-device.png)
![Assignments before cleanup](assets/screenshots/17-opnsense-interface-assignments-before-cleanup.png)

### 6. Rebuild the Base Lab Network

I moved the OPNsense `LAN` to a new management/lab subnet, removed the old VLAN device, confirmed Kea DHCP was the intended service, and created a fresh `10.10.10.0/24` pool for the routed lab side.

![LAN readdressed to 10.10.10.1](assets/screenshots/18-opnsense-lan-readdressed-10-10-10-1.png)
![VLAN device removed](assets/screenshots/19-opnsense-vlan-device-removed.png)
![Kea initially disabled](assets/screenshots/20-opnsense-kea-disabled.png)
![Old Dnsmasq DHCP range](assets/screenshots/21-opnsense-dnsmasq-old-dhcp-range.png)
![Kea enabled settings](assets/screenshots/22-opnsense-kea-enabled-settings.png)
![Kea LAN-MGMT subnet](assets/screenshots/23-opnsense-kea-dhcp-lan-mgmt-subnet.png)

### 7. Fix Routing and Validate the New Subnet

The first DHCP success did not immediately mean the lab was finished. I captured the point where the client could reach `10.10.10.1` and `192.168.1.1` but still could not reach the internet, then validated the final working state after the WAN-side policy/NAT issues were corrected.

![WAN routing failure before fix](assets/screenshots/24-opnsense-wan-routing-failure-before-fix.png)
![DHCP lease renewal from OPNsense](assets/screenshots/25-opnsense-kea-dhcp-lease-renewal.png)
![OPNsense dashboard post-fix](assets/screenshots/26-opnsense-dashboard-post-fix.png)
![Routed lab validation](assets/screenshots/28-routed-lab-validation.png)

### 8. Apply Controlled Access Instead of Full Trust

The final firewall policy kept the useful paths and cut off the broad ones. The Linux Mint lab client could still reach `pi-core`, `asus-server`, `MAIN-PC`, and the internet, but could no longer freely reach unrelated devices on the original home LAN.

![LAN firewall rules](assets/screenshots/27-opnsense-lan-firewall-rules.png)
![Service-level validation](assets/screenshots/29-service-level-validation.png)
![Blocked home-LAN devices](assets/screenshots/30-blocked-home-lan-devices.png)

## What This Phase Proved

- a client can be moved off the flat home LAN without replacing the whole network at once
- `OPNsense` can sit behind the existing Spectrum router and still provide a clean routed lab subnet
- stale firewall state should be documented and removed before building new segmentation
- selective allow rules are a safer first step than all-or-nothing changes
- the Linux Mint client is no longer a flat-LAN peer even though the rest of the house has not been fully migrated yet

## Current Network Position

The entire home network is not fully segmented yet because many production devices still live together on `192.168.1.0/24`.

What changed is that the Linux Mint test client now lives on:

- `10.10.10.100/24`
- default gateway `10.10.10.1`
- DNS `192.168.1.224`

Traffic now follows:

`Linux Mint -> OPNsense 10.10.10.1 -> Spectrum router 192.168.1.1 -> internet`

That means this lab already achieved a real segmentation milestone: the client now reaches the original home LAN only through explicit firewall policy.

## Repo Layout

- `README.md`
  - high-level narrative and key screenshots
- `docs/lab-walkthrough.md`
  - detailed chronological write-up
- `docs/evidence-manifest.md`
  - screenshot-by-screenshot evidence mapping
- `assets/screenshots/`
  - curated evidence set used across the repo

## Next Steps

- convert the GS308EP switch and Omada AP into an intentional VLAN-aware design
- create dedicated networks for management, clients, servers, IoT, and guest access
- move devices one category at a time instead of doing a single hard cutover
- reduce host-by-host exceptions by moving allowed systems into dedicated segments

## Notes

- the local `OPNsense` XML backup was intentionally excluded from GitHub
- screenshots show private RFC1918 addresses and local hostnames because this is a real home lab
