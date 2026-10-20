# IP Addressing Plan

Addressing plan for the Northgate Family Clinics two-site network.
Each site gets a private /16 block, subdivided with VLSM so every subnet
is sized to its segment.

## Site address blocks

| Site | Address block |
|------|---------------|
| Headquarters (HQ) | `10.1.0.0/16` |
| Branch (BR) | `10.2.0.0/16` |

## LAN subnets

| Site | VLAN | Name | Subnet | Mask | Gateway | Usable hosts |
|------|------|------|--------|------|---------|--------------|
| HQ | 10 | Staff | `10.1.10.0/26` | 255.255.255.192 | `10.1.10.1` | 62 |
| HQ | 20 | Servers | `10.1.20.0/27` | 255.255.255.224 | `10.1.20.1` | 30 |
| HQ | 30 | Guest | `10.1.30.0/26` | 255.255.255.192 | `10.1.30.1` | 62 |
| HQ | 99 | Management | `10.1.99.0/28` | 255.255.255.240 | `10.1.99.1` | 14 |
| Branch | 10 | Staff | `10.2.10.0/27` | 255.255.255.224 | `10.2.10.1` | 30 |
| Branch | 30 | Guest | `10.2.30.0/27` | 255.255.255.224 | `10.2.30.1` | 30 |
| Branch | 99 | Management | `10.2.99.0/28` | 255.255.255.240 | `10.2.99.1` | 14 |

## Point-to-point links

| Link | Subnet | Side A | Side B |
|------|--------|--------|--------|
| HQ to Branch (private WAN) | `10.255.0.0/30` | HQ-R1 G0/2: `10.255.0.1` | BR-R1 G0/2: `10.255.0.2` |
| HQ to ISP | `203.0.113.0/30` | ISP-R1 G0/0: `203.0.113.1` | HQ-R1 G0/1: `203.0.113.2` |
| Branch to ISP | `198.51.100.0/30` | ISP-R1 G0/1: `198.51.100.1` | BR-R1 G0/1: `198.51.100.2` |
| ISP to web server | `192.0.2.0/24` | ISP-R1 G0/2: `192.0.2.1` | Web server: `192.0.2.10` |

The `203.0.113.0/24`, `198.51.100.0/24`, and `192.0.2.0/24` ranges are
reserved for documentation and are used here to simulate public internet
addresses.

## Fixed (static) addresses

| Device | Interface | Address | Gateway |
|--------|-----------|---------|---------|
| HQ-SRV1 | NIC | `10.1.20.10/27` | `10.1.20.1` |
| HQ-SW1 | VLAN 99 SVI | `10.1.99.2/28` | `10.1.99.1` |
| BR-SW1 | VLAN 99 SVI | `10.2.99.2/28` | `10.2.99.1` |
| Web server | NIC | `192.0.2.10/24` | `192.0.2.1` |

## DHCP

DHCP is provided by the site routers.

| Site | Pool | Network | Excluded addresses | DNS server |
|------|------|---------|--------------------|------------|
| HQ | HQ-STAFF | `10.1.10.0/26` | `10.1.10.1` - `10.1.10.10` | `10.1.20.10` |
| HQ | HQ-GUEST | `10.1.30.0/26` | `10.1.30.1` - `10.1.30.10` | `192.0.2.10` |
| Branch | BR-STAFF | `10.2.10.0/27` | `10.2.10.1` - `10.2.10.10` | `10.1.20.10` |
| Branch | BR-GUEST | `10.2.30.0/27` | `10.2.30.1` - `10.2.30.10` | `192.0.2.10` |

## Routing summary

- **Between sites:** OSPF (process 1, area 0) over the `10.255.0.0/30` link.
  HQ-R1 router-ID `1.1.1.1`, BR-R1 router-ID `2.2.2.2`.
- **To the internet:** each site has a default route to the ISP and uses
  NAT overload on its ISP-facing interface.
