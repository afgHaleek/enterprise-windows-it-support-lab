# DHCP Configuration

## Overview

DHCP was deployed on DC01 to provide automatic IPv4 configuration
to Windows clients on the isolated NovaTech-LAN network.

## Scope

- Scope name: NovaTech-LAN
- Network: 192.168.10.0/24
- Address pool: 192.168.10.100 - 192.168.10.200
- DHCP server: 192.168.10.10
- DNS server: 192.168.10.10
- DNS domain: corp.novatech.test
- Default gateway: Not configured on the internal LAN

## Design Decision

The NovaTech-LAN DHCP scope does not provide a default gateway because
the lab VMs use a separate VirtualBox NAT adapter for Internet access.
This avoids introducing a second default route on the internal adapter.

## Verification

CLIENT01 successfully obtained:

- IPv4 address: 192.168.10.100
- Subnet mask: 255.255.255.0
- DHCP server: 192.168.10.10
- DNS server: 192.168.10.10
- DNS suffix: corp.novatech.test

The client was then tested for connectivity to DC01 and Active Directory
DNS service discovery.
