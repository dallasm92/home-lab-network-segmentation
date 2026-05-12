# Evidence Manifest

This manifest maps the curated screenshots to the lab story in chronological order.

## Flat-LAN Baseline

- `01-main-pc-current-ipconfig.png`
  - proves `MAIN-PC` started on the original flat `192.168.1.0/24` LAN
- `02-main-pc-baseline-connectivity.png`
  - proves the gateway, Pi-hole DNS, and internet path worked before firewall changes
- `03-main-pc-arp-baseline.png`
  - shows baseline ARP visibility from `MAIN-PC` on the shared LAN
- `04-current-lan-device-discovery-a.png`
  - shows baseline hostname and device discovery on the flat LAN
- `05-current-lan-device-discovery-b.png`
  - confirms `asus-server` resolution and reachability on the original network
- `06-current-lan-device-discovery-c.png`
  - confirms `pi-core` resolution and reachability on the original network

## Existing Physical Topology

- `07-current-switch-physical-port-map.png`
  - documents the pre-change switch cabling and physical topology
- `08-netgear-switch-management-access.png`
  - proves managed-switch reachability and the active production link state

## Early OPNsense Discovery

- `09-qotom-ip-discovery-attempt.png`
  - documents failed assumptions about the firewall being directly reachable from the production LAN
- `10-qotom-powered-on-physical-connections.png`
  - shows the Qotom appliance powered on and physically connected
- `11-direct-linux-mint-to-qotom-lan.png`
  - shows the direct Linux Mint to Qotom `LAN` management path
- `12-linux-mint-direct-link-before-dhcp.png`
  - shows the direct link state before the Linux Mint adapter had a useful IPv4 address

## Backup and Safe Rebuild Prep

- `13-opnsense-backup-page.png`
  - shows the OPNsense configuration backup page before changes
- `14-opnsense-backup-file-renamed.png`
  - proves the rollback backup was downloaded and retained locally
- `15-opnsense-interface-link-status.png`
  - proves the cable was on the active `WAN` interface while `LAN` had no carrier

## Cleanup of Stale Firewall State

- `16-opnsense-old-vlan-device.png`
  - documents the leftover `VLAN20_LAB` device before cleanup
- `17-opnsense-interface-assignments-before-cleanup.png`
  - shows the old interface-assignment state before removing the VLAN lab config
- `18-opnsense-lan-readdressed-10-10-10-1.png`
  - shows the rebuilt `LAN` interface at `10.10.10.1/24`
- `19-opnsense-vlan-device-removed.png`
  - proves the stale VLAN device was removed
- `20-opnsense-kea-disabled.png`
  - shows that Kea DHCP was initially disabled
- `21-opnsense-dnsmasq-old-dhcp-range.png`
  - documents the old DHCP-related state left behind from the previous lab

## Rebuild of the New Lab Network

- `22-opnsense-kea-enabled-settings.png`
  - shows Kea DHCP enabled on `LAN`
- `23-opnsense-kea-dhcp-lan-mgmt-subnet.png`
  - documents the new `10.10.10.0/24` LAN-MGMT subnet and DHCP pool
- `24-opnsense-wan-routing-failure-before-fix.png`
  - captures the intermediate failure where DHCP worked but outbound internet did not
- `25-opnsense-kea-dhcp-lease-renewal.png`
  - proves Linux Mint received a DHCP lease from OPNsense
- `26-opnsense-dashboard-post-fix.png`
  - shows the post-fix OPNsense state after the routed lab segment was working

## Final Policy and Validation

- `27-opnsense-lan-firewall-rules.png`
  - documents the selective allow-plus-block policy on the lab `LAN`
- `28-routed-lab-validation.png`
  - proves routed internet and DNS validation from the new lab subnet
- `29-service-level-validation.png`
  - proves service-level reachability to allowed hosts, not just ICMP
- `30-blocked-home-lan-devices.png`
  - proves at least some non-allowed home-LAN devices were blocked from the lab client
