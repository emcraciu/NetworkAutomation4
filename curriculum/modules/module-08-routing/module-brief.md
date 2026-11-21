# Module 8: Routing configuration through automation

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending (course-owner PR review, per owner decision 2026-11-21)

## Purpose and course alignment

- Course outcome supported: final-project requirement R7 in `docs/final-project-requirements.md` — "routing and firewall policy configured through automation so that selected traffic is permitted and other traffic is denied" (routing half; firewall is Module 9) — and course outcome 5 in `docs/course-outline.md`.
- Why students need this module: routing is what makes the reference topology a *network* rather than isolated nodes — end-to-end reachability between loopbacks is the substrate that the Module 9 firewall policy permits/denies and that Module 10 verifies with ping/DNS. Students must deliver it through the automation channels already learned (SSH in Module 4, NETCONF in Module 6), not by typing CLI by hand.
- What this module will not cover: advanced routing design (BGP, route redistribution, policy routing), IPv6, and firewall policy (Module 9). The routing protocol itself is the student's choice (owner decision 2026-11-21): static, OSPF, or EIGRP.
- Core / optional / enrichment: core — routing is mandatory in the agreed final workflow.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: functions, modules/imports, exceptions, dicts/JSON/YAML (Module 3), string templating for config payloads, subprocess; Module 1 concurrency is used to push config to both routers in parallel.
- Networking prerequisites: IPv4 addressing and subnetting, loopbacks, static routes, and at least one dynamic protocol at CCNA level (OSPF areas/adjacency or EIGRP AS numbers); concept of a routing table.
- GNS3 and prior-module prerequisites: Module 2 (topology via GNS3 API v3), Module 4 (SSH via netmiko/paramiko working), Module 6 (NETCONF via ncclient on CSR1000v), Module 7 (pyATS testbed + Genie parsing for verification). Devices already bootstrapped with synthetic credentials.
- Just-in-time refresher needed (if any), and why: 5-minute recap of `edit-config` merge semantics and of the pyATS `device.parse('show ip route')` pattern, since both were last seen in Modules 6–7.

## Learning outcomes

By the end of the session, students can:

1. Design and document an addressing plan for the reference topology (WAN links + loopbacks, e.g. 10.255.0.x/32 loopbacks and /30 WAN links) consistent across all nodes.
2. Configure their chosen routing protocol (static, OSPF, or EIGRP — student's choice) on both CSR1000v routers via SSH (netmiko) — optionally in parallel using Module 1 concurrency — and apply at least one routing change on a CSR1000v via NETCONF (ncclient `edit-config`); if an IOSv node is present, configure it via SSH only.
3. Verify routing convergence by parsing `show ip route` and the protocol-specific neighbor output (`show ip ospf neighbor` / `show ip eigrp neighbors` / static route entries) with Genie/pyATS and asserting the expected routes are present.
4. Demonstrate end-to-end reachability with pyATS ping tests between the routers' loopbacks and capture the passing run output.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| NETCONF on IOS XE | `netconf-yang` enables NETCONF over SSH subsystem on TCP 830 (default, configurable); capability exchange (`<hello>`) lists supported YANG models incl. `ietf-routing` and `Cisco-IOS-XE-native` | Cisco, "Programmability Configuration Guide, Cisco IOS XE Amsterdam 17.3.x — NETCONF Protocol" — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg/configuring_yang_datamodel.html ; RFC 6241 — https://www.rfc-editor.org/rfc/rfc6241 | accessed 2026-11-21 | High (owner-confirmed image); on-device `<hello>` via ncclient still owed (risk) |
| ncclient | Python NETCONF client; `manager.connect(host, port=830, ...)`, `get_config`, `edit_config` with `target='running'`, default operation `merge` | PyPI, "ncclient" — https://pypi.org/project/ncclient/ ; docs — https://ncclient.readthedocs.io/ | accessed 2026-11-21 | High; pin version; confirm `edit-config` of routing config accepted on 17.3.8a on device |
| netmiko for SSH config | netmiko (on paramiko) sends config commands over SSH; `cisco_ios`/`cisco_xe` device types fit CSR1000v and IOSv; `cisco_asa` fits ASAv | PyPI, "netmiko" — https://pypi.org/project/netmiko/ ; supported platforms — http://ktbyers.github.io/netmiko/PLATFORMS.html ; paramiko — https://pypi.org/project/paramiko/ | accessed 2026-11-21 | High for CSR/IOSv; exact driver choice validated at lab build |
| OSPF/EIGRP/static on IOS XE 17.3 | all three are fully supported on CSR1000v IOS XE 17.3.8a via CLI and via NETCONF/YANG (`Cisco-IOS-XE-ospf`, `Cisco-IOS-XE-eigrp`, `ietf-routing` for static) | Cisco IOS XE 17.3 configuration guides (cisco.com) ; DevNet standard-device matrix — https://developer.cisco.com/site/standard-network-devices/ | accessed 2026-11-21 | High; which YANG models are writable on 17.3.8a confirmed on device at lab build |
| IOSv 15.x limits | classic IOS 15 (IOSv) is SSH/console only — no NETCONF/RESTCONF; CLI config only | DevNet standard-device matrix — https://developer.cisco.com/site/standard-network-devices/ ; owner-confirmed matrix in `docs/research/device-protocol-support.md` | accessed 2026-11-21 | High; exact IOSv build number still TBD (risk) |
| OSPF/EIGRP feature support | IOS XE 17.3 supports OSPF and EIGRP configuration via CLI and via NETCONF/YANG models; classic IOS 15 (IOSv) supports them via CLI only | Cisco IOS XE 17.3 configuration guides (OSPF / EIGRP chapters, cisco.com) — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg.html (programmability) ; IOSv row in DevNet platform matrix — https://developer.cisco.com/site/standard-network-devices/ | accessed 2026-11-21 | Medium-high; exact 17.3.8a feature set confirmed on device at lab build |
| Genie route/neighbor parsing | Genie iosxe parsers exist for `show ip route` and OSPF/EIGRP neighbor output (`device.parse(...)`) returning structured dicts for assertions | genieparser repo, iosxe — https://github.com/CiscoTestAutomation/genieparser/tree/main/src/genie/libs/parser/iosxe | accessed 2026-11-21 | High; parser keys recorded on-device at lab build |
| pyATS ping/execute | pyATS `device.execute('ping ...')` / Genie `ping` op from a device, or host-side ping from the Ubuntu endpoint (Module 10), can assert reachability; this module uses device-to-device ping between loopbacks | pyATS docs — https://developer.cisco.com/docs/pyats/ | accessed 2026-11-21 | High |
| GNS3 topology via API | reference topology ≤5 nodes created/started via GNS3 API v3 (`/v3/...`); 2× CSR1000v + FTDv 10.0 + Ubuntu (+ optional IOSv) | GNS3 API — https://api.gns3.net ; gns3fy — https://davidban77.github.io/gns3fy/ ; owner decision 2026-11-21 | accessed 2026-11-21 | High (owner-confirmed) |
| Routing protocol choice | static / OSPF / EIGRP is the student's choice (owner decision 2026-11-21); the fixed part is the permitted/denied ping+DNS verification matrix later | `docs/course-outline.md`, `docs/final-project-requirements.md` (R7/R8) | 2026-11-21 | High (settled; do not re-litigate) |

### Platform and version constraints

- Cisco device types/images: 2× CSR1000v IOS XE Amsterdam 17.3.8a (SSH + NETCONF TCP 830 + RESTCONF TCP 443), optional IOSv 15.x (SSH/console only) as the 5th node, FTDv 10.0.0 present as the mandatory firewall (routing through it is Module 9's concern); topology ≤5 nodes total, created via API v3.
- GNS3 version/API requirements: server GNS3 3.0.x, REST API v3 (`/v3/...`); console via telnet host:port for any needed bootstrap; WireGuard VPN + per-user accounts.
- Python version and dependencies: Python 3.11+; `netmiko` + `paramiko` (SSH), `ncclient` (NETCONF), `pyats[full]` (verification), `requests`/`gns3fy` (API); all pinned in requirements.txt; credentials only from env vars.
- Licensing, resource, and OS assumptions: CSR1000v runs unlicensed with a throughput cap (fine for ping/config; no bulk transfers); FTDv ~8 GB RAM / 4 vCPUs typical (server sizing open); GNS3 3.0.x GPLv3.
- Protocol/API interpretation (especially if "RPC" is involved): "RPC" = NETCONF (RFC 6241) over SSH port 830 on the CSR1000v only. NETCONF operations here are `get-config`/`edit-config` on `running` with `merge` (never `replace` on the whole datastore). IOSv/ASAv/FTDv have no NETCONF — SSH/REST only.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–10 | Check-in; recap of SSH (Module 4) and NETCONF (Module 6) paths per device; review the reference addressing plan (WAN /30 links, loopbacks) | Student restates which channel reaches which device |
| 10–30 | Instructor demo: choose + configure one protocol (e.g. OSPF) on both CSR1000v via netmiko, in parallel (Module 1); one change via ncclient `edit-config` (e.g. a static route) on CSR1; IOSv via SSH if present | Student captures the config payloads and the parallel-run pattern |
| 30–45 | Demo: verification — pyATS/Genie parse of `show ip route`, `show ip ospf neighbor` (or EIGRP/static equivalents); device-to-device ping between loopbacks | Student runs the verification script and reads the parsed output |
| 45–55 | Break | — |
| 55–140 | Guided lab: students configure their chosen protocol (static/OSPF/EIGRP) on their own API-created 2× CSR1000v (+ optional IOSv via SSH), push at least one change via NETCONF on a CSR1000v, verify convergence and loopback-to-loopback ping with pyATS | Working config scripts + green pyATS ping test output |
| 140–155 | Independent practice: add/change a prefix (e.g. an additional loopback) via the student's preferred channel and re-verify | Second green run showing the new route present and reachable |
| 155–170 | Failure clinic: adjacency stuck, AS/router-id mismatch, NETCONF reject or lock conflict, missing `network`/`passive-interface` — diagnose from parsed output | Student documents one fault found and fixed in their own lab |
| 170–180 | Wrap-up and exit check; preview Module 9 (firewall will filter this traffic) | Exit ticket: explain one routing assertion and one NETCONF merge choice |

**Planned total:** 10 + 20 + 15 + 10 + 85 + 15 + 15 + 10 = **180 minutes** (including break and wrap-up)

**Out-of-class work (not counted in the 180 minutes):** none required; optional — sketch the student's own addressing plan before Module 9 so the firewall zones match it.

## Lab specification

- GNS3 topology and API steps: via API v3 (`requests` or gns3fy) create/start the reference topology: 2× CSR1000v 17.3.8a joined by a WAN link, plus FTDv 10.0.0 and the Ubuntu endpoint on the inside segment (+ optional IOSv as 5th node); interfaces pre-addressed per the Module 2 addressing plan (e.g. Gi1: 10.0.12.0/30 between routers, loopbacks 10.255.0.1/32 and 10.255.0.2/32); SSH/NETCONF enabled via the Module 4/6 bootstrap with synthetic credentials.
- Devices, images, interfaces, and services: CSR1000v 17.3.8a — SSH (netmiko `cisco_xe`/`cisco_ios`) and NETCONF TCP 830 (`netconf-yang`, ncclient); IOSv 15.x if present — SSH only; FTDv 10.0.0 present but not configured this module; services: SSH, NETCONF on CSRs.
- Starter materials: `requirements.txt` (netmiko, paramiko, ncclient, pyats[full] pinned), addressing-plan template (doc addressing only, e.g. 10.0.0.0/8 lab space), skeleton `configure_routing.py` (netmiko, env-var credentials), skeleton `netconf_change.py` (ncclient `edit-config` merge of one static route), skeleton pyATS verification script reusing the Module 7 testbed.
- Student deliverables: (1) documented addressing plan, (2) `configure_routing.py` applying the chosen protocol (static/OSPF/EIGRP) to both CSR1000v (parallel via threads/asyncio preferred), (3) `netconf_change.py` making one NETCONF change on a CSR1000v, (4) pyATS verification: parsed `show ip route` + neighbor output asserting expected routes, and ping tests between loopbacks with passing output.
- Expected time and instructor support: the guided lab (85 min) assumes SSH (Module 4) and NETCONF (Module 6) already work on both CSR1000v nodes; instructor circulates for adjacency and NETCONF `<rpc-error>` issues.
- Expected results and verification: neighbor/adjacency established (OSPF FULL or EIGRP peers up, or static routes installed); `show ip route` contains both loopbacks and the WAN prefixes; pyATS ping from CSR1 loopback source to CSR2 loopback succeeds (and the pyATS report shows PASS); NETCONF change visible in `get-config` output.
- Reset/cleanup: remove the routing configuration with negate commands or a NETCONF `delete` on the changed subtrees (keep the project for Module 9), or delete the whole project via API v3 if a fresh start is needed; no credentials in any committed file.
- Common failures and recovery: (1) OSPF stuck in EXSTART/INIT — MTU or network-type mismatch on the WAN link, check `show ip ospf neighbor` and set `ip ospf network point-to-point`; (2) EIGRP no adjacency — AS number mismatch; (3) NETCONF `<rpc-error>` — model not supported or bad xpath → inspect the error, use `Cisco-IOS-XE-native` paths, keep operation `merge`; (4) NETCONF lock conflict — another session holds the lock, disconnect or use `discard-changes`; (5) ping fails though route present — check source interface (`ping <dst> source loopback0`) and any ACL; (6) parallel push races on one device — one connection per device, serialize per-device config; (7) route present but ping fails — always ping with `source loopback0` and check for ACLs or the FTDv blocking (Module 9).
- Offline or low-resource alternative: if the server is unavailable, students validate the netmiko/ncclient payloads against a local SSH/NETCONF mock or recorded sessions and run the pyATS parse/assert steps on saved `show` outputs; the live loopback-ping run is completed in office hours. With tight RAM, run 2× CSR1000v only (drop FTDv/Ubuntu for this module's lab).

## Assessment and AI use

- Formative checks: instructor observes the protocol configuration run, the NETCONF `edit-config` result, and the pyATS verification output during the guided lab; failure-clinic diagnosis.
- Assessed evidence: addressing plan, `configure_routing.py`, `netconf_change.py`, and the pyATS verification output (parsed routes + loopback-to-loopback ping PASS) — the routing half of final-project R7.
- How individual understanding is demonstrated: exit ticket — each student explains one routing assertion and why `merge` (not `replace`) was used for the NETCONF change; teammates may share a topology but each runs and explains their own scripts.
- Permitted AI support and required disclosure: AI may be used for research, config-payload drafting, debugging, and reflection; every use logged in `AI-USE.md` (date, tool + version, task/prompt summary, accepted/changed and why). Undocumented AI use is a finding. No credentials or real-network data in prompts; isolated authorized GNS3 lab only, synthetic credentials only.

## Dependencies and course fit

- Requires: Module 2 (API-created topology), Module 4 (SSH), Module 6 (NETCONF on CSR1000v), Module 7 (pyATS testbed/parsing); Module 1 (parallel pushes); Module 3 (env-var secrets).
- Enables: Module 9 (firewall policy filters routed traffic), Module 10 (ping/DNS verification needs working routing), Module 11 and final-project R7/R8.
- Overlap with other candidate modules: none — Modules 4/6 taught the transport channels, this module applies them to routing; Module 10 is verification, not configuration.
- Connection to the final project: R7's routing half plus the reachability evidence used by R8 (permitted/denied ping/DNS matrix assumes loopback-to-loopback reachability through the topology).
- Scheduled course hours after inclusion: modules 1–11 = 33 h (11 × 3 h) with Module 1; 30 h if waived; 36 h with optional Module 12 — within the 21–39 scheduled-hour (20–40 h) limit.

## Risks and open questions

- [ ] Technical verification needed: on-device CSR1000v 17.3.8a NETCONF `<hello>` capabilities via ncclient and confirmation that `edit-config` (merge) of OSPF/EIGRP/static subtrees is accepted (RFC 6241 behavior on 17.3.8a).
- [ ] Technical verification needed: which NETCONF-writable YANG models exist for routing on 17.3.8a (`Cisco-IOS-XE-native` vs `ietf-routing`) — record at lab build; fall back to CLI via netmiko if a model is read-only.
- [ ] Technical verification needed: Genie parser keys for `show ip route` / OSPF / EIGRP neighbor output on 17.3.8a (parse once on device).
- [ ] Lab validation needed: server RAM/CPU/disk sizing for the 5-node reference topology (FTDv ~8 GB RAM, 4 vCPUs) with a full class running labs concurrently.
- [ ] Open item: exact IOSv 15.x build number still TBD (only matters if the optional IOSv node is used; SSH-only either way); WireGuard provisioning + GNS3 account procedure document still pending.
- [ ] Risk: CSR1000v unlicensed throughput cap — irrelevant for ping/config lab traffic, but no bulk transfers; FTDv 10.0 `/api/fdm/latest/` + token endpoint on-device check still owed (Module 9 dependency).
- [ ] Owner decision needed: none — routing protocol is the student's choice (static/OSPF/EIGRP), owner decision 2026-11-21.

## Researcher recommendation

- Recommendation: include
- Rationale: automated routing configuration is half of final-project R7 and the substrate for the Module 10 verification matrix; the session combines the already-learned SSH and NETCONF channels with pyATS verification, and the protocol choice (owner decision) keeps the fixed checkable part (reachability) identical across students. All claims trace to owner-confirmed decisions or official Cisco/PyPI/pyATS docs; open items are risks, not assumptions.
- Next implementation steps: after owner approval via PR review, record on-device NETCONF hello + writable routing models on CSR1000v 17.3.8a, pin netmiko/ncclient/pyats versions, and write the lab guide + starter scripts under `curriculum/modules/module-08-routing/`.
