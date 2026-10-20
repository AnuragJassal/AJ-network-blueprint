# Troubleshooting Log

**Project:** Secure Multi-Site Small-Business Network | **Environment:** Cisco Packet Tracer | **Last updated:** 2026-10-19 | **Scope:** Phase 1

Results marked "reported" were stated by the operator without saved command output. Where a command, address, or cause was not recorded, this log says so.

## Index

| ID | Issue | Status | Root cause |
|----|-------|--------|------------|
| TS-001 | HQ staff PC got no DHCP address | Resolved | Cabling did not match the VLAN/trunk design; no trunk in place |
| TS-002 | Branch PCs got no DHCP address | Resolved (cause not confirmed) | VLAN and trunk configuration missing from BR-SW1 |
| TS-003 | HQ staff PC could not ping Branch staff PC | Resolved | Router inter-site and ISP links cabled to the wrong interfaces |
| TS-004 | HQ devices could not ping web server `192.0.2.10` | Resolved | No IP address on the ISP-R1 interface facing HQ-R1 |

## Incidents

### TS-001: HQ staff PC got no DHCP address

- **Symptom:** The PC stayed at "requesting DHCP address" and fell back to an APIPA address.
- **Diagnosis:** Ran `show vlan brief` and `show interfaces trunk` on HQ-SW1. Staff PCs were on the right VLAN 10 ports, but HQ-R1 was on an access port instead of the Gigabit trunk port, and the server and guest PC were on VLAN 10 ports. No trunk was listed.
- **Fix:** Connected HQ-R1 `G0/0` to HQ-SW1 `Gi0/1`, HQ-SRV1 to `Fa0/11` (VLAN 20), and the guest PC to `Fa0/12` (VLAN 30). Configured `Gi0/1` as a trunk (exact commands not recorded; guide reference below).
- **Verification:** The HQ staff PC obtained its DHCP configuration.
- **Lesson:** For a DHCP timeout, check ports and the trunk before the router.

```
interface g0/1
 switchport mode trunk
```

### TS-002: Branch PCs got no DHCP address

- **Symptom:** Neither the Branch staff nor the guest PC received an address.
- **Diagnosis:** On BR-SW1, `show vlan brief` showed only the default VLANs and `show interfaces trunk` returned nothing. Cabling and port assignments were correct.
- **Root cause:** Confirmed that the configuration was absent from BR-SW1 at test time. **Why** is not confirmed. Candidates were wrong device or mode, a down trunk link, or configuration never saved to startup-config; none was proven.
- **Fix:** Re-entered the BR-SW1 configuration from the Phase 1 guide (VLANs 10, 30, 99, access ports, `G0/1` trunk, VLAN 99 SVI). Exact commands not recorded.
- **Verification:** Branch end devices obtained DHCP addresses. Post-fix `show` output not recorded.
- **Lesson:** Verify with `show` commands after configuring, and save with `copy running-config startup-config`.

### TS-003: HQ staff PC could not ping Branch staff PC

- **Symptom:** Both sites had DHCP, but HQ and Branch staff PCs could not reach each other.
- **Diagnosis:** Each PC could ping its own gateway. OSPF showed no neighbor and no learned routes. Cabling used `G0/1` for the inter-site link and `G0/2` toward the ISP, the reverse of the design.
- **Fix:** Recabled HQ-R1 `G0/2` to BR-R1 `G0/2` (private WAN), HQ-R1 `G0/1` to ISP-R1 `G0/0`, and BR-R1 `G0/1` to ISP-R1 `G0/1`. No OSPF configuration changes recorded.
- **Verification:** Staff PCs could ping each other and OSPF neighbors appeared. Neighbor and route output not recorded.
- **Lesson:** When an adjacency does not form, compare physical links to the design first.

### TS-004: HQ devices could not ping the web server

- **Symptom:** Branch devices reached `192.0.2.10`; HQ devices did not.
- **Diagnosis:** HQ-to-Branch connectivity already worked, so the HQ path to the ISP was checked. The ISP-R1 interface facing HQ-R1 had no IP address.
- **Fix:** Configured the address on that interface and saved to startup-config. Exact address not recorded; design value is `203.0.113.1/30` on `G0/0`. NAT was not the cause and was not changed.
- **Verification:** HQ end devices could ping `192.0.2.10`.
- **Lesson:** When one site works and another does not, compare the two paths hop by hop with `show ip interface brief`.

## Test history

| Test | Description | Result | Notes |
|------|-------------|--------|-------|
| 1 | HQ staff PC gets DHCP | Pass (reported) | TS-001 |
| 1b | Branch staff PC gets DHCP | Pass (reported) | TS-002 |
| 2 | HQ guest PC gets DHCP | Pass (reported) | TS-001 |
| 2b | Branch guest PC gets DHCP | Pass (reported) | TS-002 |
| 3 | HQ staff PC pings HQ-SRV1 (`10.1.20.10`) | Pass (reported) | |
| 4 | HQ staff PC pings Branch staff PC | Pass | TS-003 |
| 5 | Branch staff PC pings HQ-SRV1 | Pass (reported) | |
| 6 | Staff PC pings web server | Pass | TS-004 |
| 7 | Guest PC pings web server | Pass (reported) | Guest not recorded separately |
| 8 | Guest PC pings HQ-SRV1 | Not individually confirmed | Expected to succeed until ACLs are added in Phase 2 |

Passing Phase 1 connectivity tests does not mean the network is fully operational; security requirements are not yet implemented.

## Follow-up

- Confirm startup-config is saved on HQ-R1, BR-R1, HQ-SW1, and BR-SW1 (confirmed only for ISP-R1).
- Record the result of test 8 explicitly.
- Capture `show ip ospf neighbor`, `show ip route`, `show vlan brief`, and `show interfaces trunk` as evidence.
- Update `tests/test-plan.md` to reference TS-001 to TS-004.
- Phase 2: retest that guest traffic to HQ-SRV1 is blocked.

## Adding future incidents

Use the next ID, record the symptom before changing anything, note only changes actually made, retest the original failure, and update the index and test table.
