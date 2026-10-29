# Test Plan

Tests verify that the network meets the requirements in
`docs/project-scenario.md`. Faults found during testing are recorded in
`tests/troubleshooting-log.md` (TS-001 to TS-004 in Phase 1).

**Status key:** Pass | Fail | Known gap

---

## Phase 1: Core network

| # | Test | Source | Expected | Actual | Status | Notes |
|---|------|--------|----------|--------|--------|-------|
| 1 | Obtain DHCP address (Staff) | HQ and Branch staff PCs | Address from the site's Staff pool | Address received | Pass | TS-001, TS-002 |
| 2 | Obtain DHCP address (Guest) | HQ and Branch guest PCs | Address from the site's Guest pool | Address received | Pass | TS-001, TS-002 |
| 3 | Ping HQ-SRV1 (`10.1.20.10`) | HQ staff PC | Replies | Replies | Pass | |
| 4 | Ping Branch staff PC | HQ staff PC | Replies (via OSPF) | Replies | Pass | TS-003 |
| 5 | Ping HQ-SRV1 | Branch staff PC | Replies | Replies | Pass | |
| 6 | Ping web server (`192.0.2.10`) | Staff PC | Replies (via NAT) | Replies | Pass | TS-004 |
| 7 | Ping web server | Guest PC | Replies | Replies | Pass | |
| 8 | Ping HQ-SRV1 | Guest PC | Reachable (no ACLs in Phase 1) | Reachable | Known gap | Fixed in Phase 2 |

Test 8 showed a guest device could reach the internal server, which breaks
the guest-isolation requirement. It was expected before ACLs were applied
and is retested below.

---

## Phase 2: Security

| # | Test | Source | Expected | Actual | Status |
|---|------|--------|----------|--------|--------|
| 8 | Ping HQ-SRV1 (retest) | Guest PC | Blocked | Blocked | Pass |
| 9 | Ping Management VLAN devices | Guest PC | Blocked | Blocked | Pass |
| 10 | Ping HQ-SW1 management address (`10.1.99.2`) | HQ staff PC | Blocked | Blocked | Pass |
| 11 | SSH to a router or switch | Admin PC (VLAN 99) | Succeeds | Succeeded | Pass |
| 12 | SSH to a router or switch | Staff or guest PC | Blocked | Blocked | Pass |
| 13 | Connect a different device to a port-security port | Swapped PC | Port goes `err-disabled` | Port disabled as expected | Pass |
| 14 | VPN failover: shut private WAN, ping HQ staff to Branch staff | HQ staff PC | Replies; IPsec counters increase | Replies received | Pass |
| 15 | Ping web server | Guest PC | Replies (internet still allowed) | Replies received | Pass |
| 16 | Renew DHCP address | Guest PC | Address received | Address received | Pass |
| R | Regression: tests 1-7 | Various | Still pass | Still pass | Pass |

All Phase 2 tests passed as expected. Test 8 changed from a known gap in Phase 1 to blocked after the guest ACL was applied.

---

## Evidence commands

| Command | Device | Purpose |
|---------|--------|---------|
| `show vlan brief` | Switches | Confirm VLANs and port membership |
| `show interfaces trunk` | Switches | Confirm trunk links |
| `show ip interface brief` | Routers, switches | Interface status and addresses |
| `show ip route` | Routers | Connected, static, and OSPF routes |
| `show ip ospf neighbor` | Routers | OSPF adjacency |
| `show ip dhcp binding` | Routers | DHCP leases |
| `show ip nat translations` | Routers | NAT entries |
| `show access-lists` | Routers | ACL rules and match counters |
| `show port-security interface <port>` | Switches | Port-security state and violations |
| `show crypto isakmp sa` | HQ-R1, BR-R1 | IKE tunnel state (`QM_IDLE`) |
| `show crypto ipsec sa` | HQ-R1, BR-R1 | Encrypted packet counters |
| `ping`, `tracert`, `ssh -l admin <ip>` | End devices | Reachability and device access |
