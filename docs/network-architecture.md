# Network Architecture

## Overview

The NovaTech Solutions lab uses two VirtualBox network interfaces per
virtual machine to separate internet connectivity from the internal
corporate network.

## Network Design

| Network | Purpose | Configuration |
|---|---|---|
| WAN-NAT | Internet connectivity | VirtualBox NAT |
| NovaTech-LAN | Internal corporate network | VirtualBox Internal Network |
| Internal subnet | Corporate addressing | 192.168.10.0/24 |

## DC01

DC01 is the primary Windows Server for the lab.

- Hostname: DC01
- Operating System: Windows Server 2025
- WAN interface: WAN-NAT
- Internal interface: NOVATECH-LAN
- Internal IPv4 address: 192.168.10.10/24
- Internal default gateway: None
- Address assignment: Static

The internal interface does not have a default gateway because there is
currently no router on the NovaTech-LAN network. Internet traffic uses
the separate WAN-NAT interface.

## CLIENT01

CLIENT01 is the Windows workstation used for domain and IT support testing.

- Operating System: Windows 11
- WAN interface: WAN-NAT
- Internal interface: NOVATECH-LAN
- Internal addressing: DHCP
- Current internal address: APIPA (169.254.x.x)

CLIENT01 currently receives an APIPA address because no DHCP service has
been deployed on NovaTech-LAN yet. DHCP will be configured later in the
project.

## Design Decisions

A separate internal VirtualBox network was used to isolate the corporate
lab from the physical home network.

DC01 uses a static internal IPv4 address because services such as Active
Directory, DNS, and DHCP require predictable server addressing.

Client devices will use DHCP rather than manually configured static
addresses.
