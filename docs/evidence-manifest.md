# Evidence Manifest

## Baseline

- `01-main-pc-current-ipconfig.png`
  - proves `MAIN-PC` was on the original flat `192.168.1.0/24` LAN
- `02-main-pc-baseline-connectivity.png`
  - proves baseline gateway, DNS, and internet connectivity before segmentation
- `03-current-lan-device-discovery-a.png`
- `03-current-lan-device-discovery-b.png`
- `03-current-lan-device-discovery-c.png`
  - prove visibility of key devices on the original flat LAN

## Physical Topology

- `04-current-switch-physical-port-map.png`
  - documents pre-change switch cabling
- `05-netgear-switch-management-access.png`
  - proves managed-switch reachability and current link state

## OPNsense Discovery

- `06-qotom-ip-discovery-attempt.png`
  - documents failed assumptions about common firewall IPs on the production LAN
- `07-qotom-powered-on-physical-connections.png`
  - shows Qotom hardware powered on and connected
- `08-opnsense-interface-link-status.png`
  - proves `WAN` was the active linked interface and `LAN` had no carrier during early troubleshooting

## Cleanup and Rebuild

- `09-opnsense-interface-assignments-before-cleanup.png`
  - shows the leftover `VLAN20_LAB` assignment
- `10-opnsense-interface-assignments-after-cleanup.png`
  - shows the cleaned baseline with only `LAN` and `WAN`
- `11-opnsense-kea-dhcp-lan-mgmt-subnet.png`
  - documents the new `10.10.10.0/24` LAN-MGMT subnet and pool
- `12-opnsense-kea-dhcp-lease-renewal.png`
  - proves the Linux Mint client received DHCP from OPNsense

## Final Validation

- `13-opnsense-lan-firewall-rules.png`
  - documents the selective allow-plus-block policy on the lab LAN
- `14-routed-lab-validation.png`
  - proves DNS and routed internet access from the lab subnet
- `15-blocked-home-lan-devices.png`
  - proves at least some non-allowed home-LAN devices were not reachable
