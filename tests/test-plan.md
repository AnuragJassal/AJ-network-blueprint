# Test Plan: Phase 1 (Core Network)

Tests verify that the core network from Phase 1 behaves as designed.
Expected results come from the requirements in `docs/project-scenario.md`.

## Results

| # | Test | Source | Expected | Actual | Status |
|---|------|--------|----------|--------|--------|
| 1 | Staff PC at HQ obtains a DHCP address in `10.1.10.0/26` | HQ staff PC | Address received | Address received | Pass |
| 2 | Guest PC at HQ obtains a DHCP address in `10.1.30.0/26` | HQ guest PC | Address received | Address received | Pass |
| 3 | Ping HQ-SRV1 (`10.1.20.10`) | HQ staff PC | Replies received | Replies received | Pass |
| 4 | Ping Branch staff PC | HQ staff PC | Replies received (via OSPF) | Replies received | Pass |
| 5 | Ping HQ-SRV1 (`10.1.20.10`) | Branch staff PC | Replies received | Replies received | Pass |
| 6 | Ping web server (`192.0.2.10`) | Staff PC | Replies received (via NAT) | Replies received | Pass |
| 7 | Ping web server (`192.0.2.10`) | Guest PC | Replies received | Replies received | Pass |
| 8 | Ping HQ-SRV1 (`10.1.20.10`) | Guest PC | Reachable in Phase 1 (no ACLs yet) | Reachable | Known gap |

Branch DHCP (staff and guest) initially failed and was fixed; see
`tests/troubleshooting-log.md`. The results above are from the final run.

## Known gap

**Test 8** shows that a guest device can reach the internal server. This
conflicts with the security requirement that guest traffic must not reach
internal systems. No access control lists are applied in Phase 1, so this
is expected. It will be fixed in Phase 2 with ACLs, and the test will be
rerun with the expected result changed to **blocked**.

## Verification commands used

| Command | Device | Purpose |
|---------|--------|---------|
| `show vlan brief` | Switches | Confirm VLANs and port membership |
| `show interfaces trunk` | Switches | Confirm trunk links |
| `show ip interface brief` | Routers, switches | Confirm interface status and addresses |
| `show ip route` | Routers | Confirm connected, static, and OSPF routes |
| `show ip ospf neighbor` | Routers | Confirm the OSPF adjacency |
| `show ip dhcp binding` | Routers | Confirm DHCP leases |
| `show ip nat translations` | Routers | Confirm NAT entries |
| `ipconfig`, `ping`, `tracert` | End devices | Confirm addressing and reachability |

## Phase 2 retests

| # | Test | Expected after Phase 2 |
|---|------|------------------------|
| 8 | Guest PC pings HQ-SRV1 | Blocked |
| 9 | Guest PC pings Management VLAN devices | Blocked |
| 10 | Staff PC pings Management VLAN devices | Blocked (except from admin hosts, if permitted) |
| 11 | SSH to a device from the Management VLAN | Succeeds |
| 12 | SSH to a device from the Staff or Guest VLAN | Blocked |
