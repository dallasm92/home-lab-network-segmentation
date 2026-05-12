# Evidence Manifest

This manifest maps the curated screenshots to the lab story in chronological order. Each image supports a specific point in the case study rather than serving as a loose screenshot dump.

## Phase 1: Baseline the Flat LAN

- `assets/screenshots/01-main-pc-current-ipconfig.png`
  - Proves `MAIN-PC` started on the original flat `192.168.1.0/24` network with expected baseline addressing.
- `assets/screenshots/02-main-pc-baseline-connectivity.png`
  - Proves gateway, DNS, and outbound connectivity were working before any firewall changes.
- `assets/screenshots/03-main-pc-arp-baseline.png`
  - Proves broad peer visibility on the flat LAN before segmentation.
- `assets/screenshots/04-current-lan-device-discovery-a.png`
  - Proves the first portion of baseline device discovery on the original home LAN.
- `assets/screenshots/05-current-lan-device-discovery-b.png`
  - Proves `asus-server` was visible and reachable during the baseline stage.
- `assets/screenshots/06-current-lan-device-discovery-c.png`
  - Proves `pi-core` was visible and reachable during the baseline stage.

## Phase 2: Document Physical Topology

- `assets/screenshots/07-current-switch-physical-port-map.png`
  - Proves the pre-change labeled switch cabling layout and physical baseline.
- `assets/screenshots/08-netgear-switch-management-access.png`
  - Proves switch management access existed before firewall changes.

## Phase 3: OPNsense Discovery Problem

- `assets/screenshots/09-qotom-ip-discovery-attempt.png`
  - Proves the first production-LAN discovery attempt for the firewall failed.
- `assets/screenshots/10-qotom-powered-on-physical-connections.png`
  - Proves the Qotom appliance was powered on and physically connected during discovery troubleshooting.
- `assets/screenshots/15-opnsense-interface-link-status.png`
  - Captures additional recovery-side network state from the isolated management path after production-side discovery failed.

## Phase 4: Safe LAN-Side Recovery

- `assets/screenshots/11-direct-linux-mint-to-qotom-lan.png`
  - Proves I created an isolated management path by connecting Linux Mint directly to the Qotom `LAN` port.
- `assets/screenshots/12-linux-mint-direct-link-before-dhcp.png`
  - Proves the direct management link existed before Linux Mint had a useful DHCP result.

## Phase 5: Backup Before Changes

- `assets/screenshots/13-opnsense-backup-page.png`
  - Proves I captured the firewall configuration from the OPNsense backup page before making changes.
- `assets/screenshots/14-opnsense-backup-file-renamed.png`
  - Proves the downloaded backup was retained locally as a rollback artifact.

## Phase 6: Remove Stale Firewall State

- `assets/screenshots/16-opnsense-old-vlan-device.png`
  - Proves the firewall still had a leftover `VLAN20_LAB` device before cleanup.
- `assets/screenshots/17-opnsense-interface-assignments-before-cleanup.png`
  - Proves the pre-cleanup interface assignment state was documented instead of hidden.
- `assets/screenshots/19-opnsense-vlan-device-removed.png`
  - Proves the stale VLAN device was removed before the rebuild.
- `assets/screenshots/20-opnsense-kea-disabled.png`
  - Proves Kea DHCP was not initially enabled when I assessed the stale state.
- `assets/screenshots/21-opnsense-dnsmasq-old-dhcp-range.png`
  - Proves old DHCP-related state was still present and needed to be cleaned up.

## Phase 7: Rebuild LAN on 10.10.10.0/24

- `assets/screenshots/18-opnsense-lan-readdressed-10-10-10-1.png`
  - Proves the rebuilt `LAN` interface was moved to `10.10.10.1/24`.
- `assets/screenshots/22-opnsense-kea-enabled-settings.png`
  - Proves Kea DHCP was enabled for the new lab design.
- `assets/screenshots/23-opnsense-kea-dhcp-lan-mgmt-subnet.png`
  - Proves the new `10.10.10.0/24` subnet and DHCP scope were defined on the firewall.

## Phase 8: DHCP Success but Routing Failure

- `assets/screenshots/24-opnsense-wan-routing-failure-before-fix.png`
  - Proves there was an intermediate failure where DHCP worked but routed internet access did not.
- `assets/screenshots/25-opnsense-kea-dhcp-lease-renewal.png`
  - Proves Linux Mint received a DHCP lease from OPNsense on the new subnet.

## Phase 9: Working Routed Segment

- `assets/screenshots/26-opnsense-dashboard-post-fix.png`
  - Proves the firewall was in a post-fix working state after the WAN-side issue was corrected.
- `assets/screenshots/28-routed-lab-validation.png`
  - Proves the Linux Mint client had a working routed path from `10.10.10.0/24` through OPNsense.

## Phase 10: Controlled Access Back to the Old LAN

- `assets/screenshots/27-opnsense-lan-firewall-rules.png`
  - Proves the final policy was selective access rather than open-ended trust.
- `assets/screenshots/29-service-level-validation.png`
  - Proves the client could still reach allowed services and hosts after policy enforcement.
- `assets/screenshots/30-blocked-home-lan-devices.png`
  - Proves access to specific unrelated home-LAN IPs was blocked from the lab client after the policy was applied.
