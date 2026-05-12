# Home Lab Network Segmentation

This repo documents my first real move away from a flat `192.168.1.0/24` home network and into a routed, policy-controlled lab segment behind a `Qotom` firewall running `OPNsense`.

The scope is intentionally narrow and accurate. This is not a full-house VLAN rollout. The milestone for this phase was to move one Linux Mint test client off the flat LAN, place it behind `OPNsense` on `10.10.10.0/24`, and validate that it could still reach the systems I needed without keeping broad peer-level access to the rest of the home network.

Social preview asset:
- [assets/social-preview.png](assets/social-preview.png)

## Start Here

If you only review four items in this repo, use these:

1. [assets/diagrams/topology-before-after.svg](assets/diagrams/topology-before-after.svg) for the before/after milestone
2. [09-qotom-ip-discovery-attempt.png](assets/screenshots/09-qotom-ip-discovery-attempt.png) for the first discovery failure
3. [24-opnsense-wan-routing-failure-before-fix.png](assets/screenshots/24-opnsense-wan-routing-failure-before-fix.png) for the intermediate routing problem
4. [28-routed-lab-validation.png](assets/screenshots/28-routed-lab-validation.png) for proof of the final working routed segment

## Objective

Build a safer first segmentation step without disrupting the production home network:

- document the flat LAN before changing anything
- recover safe access to `OPNsense` when the expected management path failed
- remove stale firewall state instead of building on top of it
- rebuild the lab side as `10.10.10.0/24`
- validate DHCP, routing, and selective access back to the old LAN

## Environment

- Platform: self-hosted home lab
- Upstream home LAN: `192.168.1.0/24`
- Upstream gateway: `192.168.1.1`
- Firewall platform: `Qotom` running `OPNsense`
- Lab client used for staged validation: Linux Mint
- Core systems referenced in validation:
  - `MAIN-PC` at `192.168.1.167`
  - `asus-server` at `192.168.1.221`
  - `pi-core` / Pi-hole at `192.168.1.224`
  - Netgear GS308EP at `192.168.1.117`

## Target Outcome for This Phase

- Keep the existing home network upstream and stable.
- Rebuild `OPNsense LAN` as `10.10.10.1/24`.
- Place the Linux Mint test client behind the firewall on `10.10.10.100/24`.
- Preserve useful access to `pi-core`, `asus-server`, `MAIN-PC`, and the internet.
- Block access to unrelated home-LAN devices that do not need to be reachable from the lab segment.

## Before / After Topology

Topology reference:

- [assets/diagrams/topology-before-after.svg](assets/diagrams/topology-before-after.svg)

This topology shows the actual scope of the repo:

- before: one flat `192.168.1.0/24` home LAN
- after: one staged routed lab segment behind `OPNsense`, while the rest of the house remains on the original network

## Hiring Manager Quick View

| Review area | Evidence |
|---|---|
| Network baseline discipline | Documented IP state, connectivity, ARP visibility, and device discovery before changes |
| Troubleshooting | Production-side firewall discovery failed and was recovered from the isolated `LAN` side |
| Change control | `OPNsense` configuration backup captured before interface and DHCP changes |
| Routing and DHCP validation | Captured both the intermediate failure and the final working routed path |
| Access control | Preserved access to needed lab systems while blocking unrelated home-LAN devices |
| Documentation quality | Full walkthrough plus screenshot-by-screenshot evidence map |

## Evidence Set

Screenshots are stored in [`assets/screenshots/`](assets/screenshots/).

Primary references for this README:

1. [01-main-pc-current-ipconfig.png](assets/screenshots/01-main-pc-current-ipconfig.png) - baseline `MAIN-PC` state on the flat LAN
2. [07-current-switch-physical-port-map.png](assets/screenshots/07-current-switch-physical-port-map.png) - physical switch and cabling baseline
3. [09-qotom-ip-discovery-attempt.png](assets/screenshots/09-qotom-ip-discovery-attempt.png) - failed production-side firewall discovery
4. [11-direct-linux-mint-to-qotom-lan.png](assets/screenshots/11-direct-linux-mint-to-qotom-lan.png) - isolated recovery path from Linux Mint to `OPNsense LAN`
5. [18-opnsense-lan-readdressed-10-10-10-1.png](assets/screenshots/18-opnsense-lan-readdressed-10-10-10-1.png) - rebuilt `LAN` interface on `10.10.10.1/24`
6. [24-opnsense-wan-routing-failure-before-fix.png](assets/screenshots/24-opnsense-wan-routing-failure-before-fix.png) - intermediate failure after DHCP but before full outbound routing worked
7. [27-opnsense-lan-firewall-rules.png](assets/screenshots/27-opnsense-lan-firewall-rules.png) - selective firewall policy
8. [28-routed-lab-validation.png](assets/screenshots/28-routed-lab-validation.png) - successful routed-segment validation

For the full sequence:

- detailed case-study walkthrough: [docs/lab-walkthrough.md](docs/lab-walkthrough.md)
- screenshot-by-screenshot map: [docs/evidence-manifest.md](docs/evidence-manifest.md)

## Lab Steps

### Starting Point

The original network was a flat `192.168.1.0/24` LAN with the Spectrum router at `192.168.1.1`. At that point, the Linux Mint client was still just another peer on the same trust zone as the rest of the house.

Evidence:

- [01-main-pc-current-ipconfig.png](assets/screenshots/01-main-pc-current-ipconfig.png)
- [07-current-switch-physical-port-map.png](assets/screenshots/07-current-switch-physical-port-map.png)

### What Failed

I expected the `Qotom` firewall to be reachable from the production LAN once it was powered on and connected. That assumption failed, which forced me to stop guessing at management IPs and recover access from a safer path instead.

Evidence:

- [09-qotom-ip-discovery-attempt.png](assets/screenshots/09-qotom-ip-discovery-attempt.png)

### What I Changed

I used a direct Linux Mint to `OPNsense LAN` connection with a USB-to-Ethernet adapter, captured a configuration backup, documented and removed stale lab state, rebuilt the `LAN` side as `10.10.10.1/24`, enabled Kea DHCP for the staged lab subnet, and then validated the routed path after correcting the WAN-side issue.

Evidence:

- [11-direct-linux-mint-to-qotom-lan.png](assets/screenshots/11-direct-linux-mint-to-qotom-lan.png)
- [18-opnsense-lan-readdressed-10-10-10-1.png](assets/screenshots/18-opnsense-lan-readdressed-10-10-10-1.png)
- [24-opnsense-wan-routing-failure-before-fix.png](assets/screenshots/24-opnsense-wan-routing-failure-before-fix.png)

### Final Validation

The final path for this phase was:

`Linux Mint 10.10.10.100 -> OPNsense LAN 10.10.10.1 -> OPNsense WAN -> Spectrum router 192.168.1.1 -> internet`

What this proves:

- the Linux Mint test client is no longer a flat peer on `192.168.1.0/24`
- it now lives behind `OPNsense` on `10.10.10.0/24`
- it reaches the old LAN only through firewall policy

Evidence:

- [27-opnsense-lan-firewall-rules.png](assets/screenshots/27-opnsense-lan-firewall-rules.png)
- [28-routed-lab-validation.png](assets/screenshots/28-routed-lab-validation.png)

## What I Learned

- documenting the baseline first made later troubleshooting easier to defend and explain
- direct `LAN`-side recovery was safer than introducing the firewall `LAN` into the production switch too early
- stale VLAN and DHCP state should be documented and removed before building a clean segment
- DHCP success is only one checkpoint; routing and policy still need separate validation
- a small routed segment with selective access is a more honest first milestone than claiming full segmentation too early

## Current Limitations

- the entire home network is not fully VLAN-segmented yet
- the old `192.168.1.0/24` production LAN still remains upstream
- this repo documents a successful first routed segment, not a finished network redesign

## Next Steps

- define the first intentional VLAN and subnet plan beyond this staged segment
- map GS308EP access and trunk roles before moving more devices
- move additional systems one controlled path at a time instead of attempting one large cutover
- keep reducing flat-LAN trust by replacing host-by-host allowances with clearer network boundaries

## Repository Layout

- `README.md`
  - high-level project summary and curated evidence
- `docs/lab-walkthrough.md`
  - full chronological case study
- `docs/evidence-manifest.md`
  - screenshot-by-screenshot evidence reference
- `assets/diagrams/topology-before-after.svg`
  - before/after topology for this phase
- `assets/screenshots/`
  - curated screenshot evidence used by the write-up

## Outcome

This lab produced a working first routed lab segment behind `OPNsense`. The Linux Mint client now operates from `10.10.10.0/24` instead of as a flat-LAN peer, while access back to the original home LAN is limited to explicitly allowed paths.
