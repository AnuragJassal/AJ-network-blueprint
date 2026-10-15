# Secure Multi-Site Network Blueprint for a Small Clinic Group

> Draft scenario document. Edit anything that doesn't match what you want to build.

## 1. The problem

Small multi-location businesses such as dental and medical clinics often run
flat networks built from consumer-grade equipment. Staff devices, patient
systems, guest Wi-Fi, and phones share one network. This creates three risks:

- **Security:** one infected laptop or a guest device can reach patient data.
- **Compliance:** privacy rules for health information (PHIPA in Ontario,
  HIPAA in the US) expect access controls and separation of systems.
- **Operations:** with no documentation, outages and staff changes are slow
  and costly to handle.

## 2. The goal

Design, build, and document a **reusable, secure network blueprint** for a
small clinic group with two sites, which a real small business could adopt.
The deliverable is not only a working lab but a package someone else could
follow: design, configs, security policy, onboarding procedure, and test
results.

## 3. The client (fictional)

"Northgate Family Clinics": one headquarters clinic and one branch clinic.

| Site | Staff | Devices |
|------|-------|---------|
| HQ | ~30 | workstations, laptops, printers, imaging server, file server |
| Branch | ~10 | workstations, printer, IP phones |

## 4. Requirements

**Functional**
1. Both sites can reach shared servers at HQ.
2. Separate network segments (VLANs) for Staff, Servers, Guest, and Management.
3. Guest Wi-Fi has internet access only, with no access to internal systems.
4. Central user accounts and policies (Active Directory, DNS, DHCP).
5. Branch connects to HQ over a secure site-to-site link.

**Security**
1. Management interfaces reachable only from the Management VLAN, over SSH.
2. Access rules (ACLs) enforce which segments can talk to which.
3. Unused switch ports disabled; port security on user ports.
4. Documented onboarding and offboarding procedure for staff accounts.

**Operational**
1. IP plan, topology diagram, and device configs stored in this repo.
2. A test plan with expected and actual results.
3. A troubleshooting log recording faults found and how they were fixed.

## 5. Planned design (to be refined in Phase 1)

| VLAN | Name | Purpose |
|------|------|---------|
| 10 | Staff | Staff workstations and laptops |
| 20 | Servers | AD/DNS/DHCP, file and imaging servers |
| 30 | Guest | Guest Wi-Fi |
| 99 | Management | Network device administration |

Starting address space (to be subnetted in Phase 1): HQ `10.1.0.0/16`,
Branch `10.2.0.0/16`.

## 6. Deliverables

- [ ] Scenario and requirements (this document)
- [ ] IP addressing plan and topology diagram
- [ ] Working Packet Tracer lab (.pkt) and device configs
- [ ] Security configuration (ACLs, SSH, port security, VPN)
- [ ] Active Directory lab with onboarding/offboarding runbook
- [ ] Test plan and troubleshooting log
- [ ] README summarising the design and lessons learned

## 7. Success criteria

- Every requirement above is met and verified in the test plan.
- A guest device cannot reach the Servers or Management VLANs.
- A new staff account can be created and removed by following the runbook.
- Someone else could rebuild the network from this repository.

## 8. Out of scope

Real hardware, wireless controllers, cloud services, and monitoring tools.
These may be added later.
