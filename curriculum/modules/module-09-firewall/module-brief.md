# Module 9: Firewall policy automation

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending (course-owner PR review, per owner decision 2026-11-21)

## Purpose and course alignment

- Course outcome supported: final-project requirement R7 (firewall half — permit selected traffic, deny the rest) and R5 (REST used where supported) in `docs/final-project-requirements.md`, plus course outcome 5 in `docs/course-outline.md`.
- Why students need this module: the FTDv 10.0 is the mandatory firewall in the reference topology, and the fixed, checkable core of the final project is the permitted/denied ping+DNS matrix (R8) — which only makes sense if students can push a real access policy through the FDM REST API themselves. Module 5 taught REST generically; this module applies it to security policy with a deploy step.
- What this module will not cover: FMC-managed (off-box) FTD policy, NAT in depth, intrusion/AMP policies, and ASAv configuration in depth — ASAv 9.24 via the ASA REST API is taught as the documented alternative, not both in one session.
- Core / optional / enrichment: core — FTDv is mandatory in the reference topology (owner decision 2026-11-21).

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: functions, modules/imports, exceptions, JSON/dicts (REST payloads), env vars for secrets (Module 3); HTTP status codes at a basic level.
- Networking prerequisites: inside/outside security zones and interface security levels conceptually, ACL permit/deny semantics and rule order, ICMP and DNS (UDP/TCP 53) as the services used in the verification matrix.
- GNS3 and prior-module prerequisites: Module 2 (topology via GNS3 API v3), Module 4 (console bootstrap + SSH to the FTD CLI), Module 5 (REST with requests, auth patterns, verifying responses), Module 8 (routing already working, so permitted traffic has a path). FTDv 10.0.0 present in the student's project.
- Just-in-time refresher needed (if any), and why: 5-minute recap of bearer-token auth and of `requests` + `verify=False` for the lab's self-signed cert (isolated lab only), since Module 5 used basic auth on RESTCONF.

## Learning outcomes

By the end of the session, students can:

1. Authenticate to the FTDv 10.0.0 FDM REST API with `requests` (OAuth2 password grant → bearer token, credentials from env vars, never committed) and confirm the API base/version via a GET against `/api/fdm/latest/`.
2. Create network objects and an access rule through the FDM API (`/api/fdm/latest/networkobjects`, `/api/fdm/latest/accessrules` under the access policy) that permits selected traffic (ICMP + DNS, inside → outside) and denies the rest, then trigger the policy deploy step and check its status.
3. Verify the pushed policy both via API GET (rule present, correct order/sources/destinations/services) and via the FTD CLI over SSH (e.g. `show access-list`), and explain why the deny-by-default outcome must be verified, not assumed.
4. Describe the ASAv 9.24 alternative — ASA REST API at base `/api/` (`rest-api agent`), objects under `/api/objects/`, access rules/ACEs — including the caveat that REST changes are not saved to startup config unless the Write Memory API is called, and state when to choose ASAv instead of FTDv.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| FDM REST API base and auth | On-box FDM API is served over HTTPS at base `/api/fdm/v1/` with the `latest` alias (`/api/fdm/latest/`); clients authenticate with OAuth2 password grant and use `Authorization: Bearer <token>` | Cisco, "Secure Firewall Threat Defense REST API Guide — Authenticating Your REST API Client Using OAuth" — https://www.cisco.com/c/en/us/td/docs/security/firepower/ftd-api/guide/ftd-rest-api/auth-ftd-rest-api.html ; DevNet "Get an Access Token" — https://developer.cisco.com/docs/firepower/threat-defense/get-an-access-token/ | accessed 2026-11-21 | High for mechanism; exact token path on FTD 10.0 (`/api/fdm/latest/fdm/token` vs versioned path) must be verified on device (risk) |
| Access rules and objects | FDM API resources include `accessrules` (create rule in an access policy) and `networkobjects`; payloads reference zones, networks, ports/services | DevNet FTD API reference (latest) — https://developer.cisco.com/docs/ftd-api-reference/latest/accessrule/ and .../addaccessrule/ — https://developer.cisco.com/docs/ftd-api-reference/latest/addaccessrule/ | accessed 2026-11-21 | High; exact model field names for FTD 10.0's API version confirmed via the on-device API explorer at lab build |
| On-device API explorer | FDM ships an interactive API explorer (Swagger UI) on the device so resource paths and model fields can be checked on-box before coding — the authoritative reference for the exact FTD 10.0 API version | Cisco, "FTD REST API Guide — Using the Methods and Resources / API Explorer" — https://www.cisco.com/c/en/us/td/docs/security/firepower/ftd-api/guide/ftd-rest-api/ftd-api-methods.html | accessed 2026-11-21 | High; use it at lab build to resolve the token-path and field-name risks below |
| Deploy step | FDM API changes are staged until deployed; `POST .../deployment/deployconfig` pushes pending changes to the device; deployment status is queryable | Cisco, "FTD REST API Guide — Deploying Configuration Changes" — https://www.cisco.com/c/en/us/td/docs/security/firepower/ftd-api/guide/ftd-rest-api/ftd-api-deployment.html | accessed 2026-11-21 | High; deploy time on FTDv (can take minutes) measured at lab build |
| FTDv 10.0 image and licensing | FTD 10.0 exists (upgrades from 7.3–7.7); FTDv runs a 90-day evaluation without Smart Licensing registration | Cisco, "Release Notes … Threat Defense 10.0" — https://www.cisco.com/c/en/us/td/docs/security/secure-firewall/release-notes/threat-defense/100/device-manager-release-notes-10-0.html ; FDM Configuration Guide, "Licensing" — https://www.cisco.com/c/en/us/td/docs/security/firepower/70/fdm/fptd-fdm-config-guide-700/fptd-fdm-license.html | accessed 2026-11-21 | High (owner-confirmed image); eval-mode behavior on FDM API verified on device |
| FTDv resources | FTDv from 6.4 deploys with adjustable vCPU/RAM (pre-6.4 fixed 4 vCPU / 8 GB); FTDv 10.0 follows the adjustable model — plan ~8 GB RAM, 4 vCPUs per FTDv | Cisco, "FTDv Getting Started Guide 10.0 — Deploy on KVM" — https://www.cisco.com/c/en/us/td/docs/security/firepower/quick_start/consolidated_ftdv_gsg/threat-defense-virtual-10-0-gsg/m-ftdv-kvm-gsg.html | accessed 2026-11-21 | High; server sizing for a class still open (risk) |
| No NETCONF on FTD | FTD has no NETCONF; management is FDM REST API (+ SSH to the FTD CLI) — matches the course matrix | `docs/research/device-protocol-support.md` (owner-confirmed 2026-11-21); DevNet platform matrix — https://developer.cisco.com/site/standard-network-devices/ | accessed 2026-11-21 | High |
| ASA REST API (alternative) | ASAv 9.24 ships the REST agent (`rest-api agent`, HTTPS base `/api/`); objects under `/api/objects/` (e.g. networkobjects), ACL/access rules under the ACL resources; the embedded Doc-UI at `https://<asa>/doc/` lists exact paths; **REST changes apply to running config only — call the Write Memory API to persist** | Cisco, "About the ASA REST API" — https://www.cisco.com/c/en/us/td/docs/security/asa/api/asa_rest_api.html ; "ASA REST API Quick Start Guide" — https://www.cisco.com/c/en/us/td/docs/security/asa/api/quick-start/qsg-asa-api.html ; ASA 9.24 release notes — https://www.cisco.com/c/en/us/td/docs/security/asa/asa924/release/notes/asarn924.html | accessed 2026-11-21 | Medium-high; exact ACL resource paths verified on device via `/doc/` at lab build |
| GNS3 topology via API | reference topology ≤5 nodes created via GNS3 API v3; FTDv mandatory; 2× CSR1000v + FTDv 10.0 + Ubuntu (+ optional IOSv/ASAv) | GNS3 API — https://api.gns3.net ; gns3fy — https://davidban77.github.io/gns3fy/ ; owner decision 2026-11-21 | accessed 2026-11-21 | High (owner-confirmed) |
| Verification matrix rule | fixed part of student projects = permitted/denied ping + DNS matrix; routing protocol and DNS service are the student's choice | `docs/final-project-requirements.md` (R7/R8); owner decision 2026-11-21 | 2026-11-21 | High (settled; do not re-litigate) |

### Platform and version constraints

- Cisco device types/images: FTDv 10.0.0 MANDATORY in this module's lab (SSH to FTD CLI + FDM REST API base `/api/fdm/latest/`, HTTPS, token auth; no NETCONF); ASAv 9.24 documented as the alternative (SSH + ASA REST API `/api/`); CSR1000v 17.3.8a ×2 and Ubuntu endpoint provide inside/outside segments; ≤5 nodes total, created via API v3.
- GNS3 version/API requirements: server GNS3 3.0.x, REST API v3 (`/v3/...`); FTDv management (HTTPS/FDM + SSH) reachable from the student host over WireGuard; console via telnet host:port for first-time FTD setup (Module 4 pattern).
- Python version and dependencies: Python 3.11+; `requests` (FDM/ASA REST calls), `pyats[full]` (Module 7 testbed for CLI-side verification), `netmiko`/`paramiko` (FTD CLI over SSH), `gns3fy`/`requests` (API); pinned in requirements.txt; tokens/credentials only from env vars, never committed.
- Licensing, resource, and OS assumptions: FTDv 10.0 runs 90-day evaluation without Smart Licensing; plan ~8 GB RAM / 4 vCPUs per FTDv (server sizing open); FTD HTTPS uses a self-signed cert in the lab → `verify=False` (isolated lab only) or install the lab CA; GNS3 3.0.x GPLv3; WireGuard per-user access.
- Protocol/API interpretation (especially if "RPC" is involved): firewall policy here is REST (HTTPS + JSON), not NETCONF — FTD/ASA have none. "RPC" in this course remains NETCONF (RFC 6241) on the CSR1000v only (Module 6/8). FDM API = on-box device-manager API; the separate FMC API is out of scope.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–10 | Check-in; recap of Module 5 REST and Module 8 routing; confirm FTDv 10.0 is up in each student's API-created topology and its inside/outside interfaces are addressed | Student states the FDM base path and auth flow from memory/notes |
| 10–35 | Instructor demo: FDM API auth (token via env-var credentials), GET device/system info at `/api/fdm/latest/`; create network objects; create the access rule (permit ICMP + DNS inside→outside, deny rest by default); trigger `deployconfig` and poll deploy status | Student records token flow, payload shapes, and deploy time |
| 35–55 | Demo: ASAv 9.24 alternative — ASA REST API `/api/` objects + access rules, `write memory` caveat; verification: API GET of the rule + FTD/ASA CLI (`show access-list`) over SSH | Student captures the verification commands and expected output |
| 55–65 | Break | — |
| 65–145 | Guided lab: in their own topology, push a permit/deny policy to the FTDv via the FDM API — network objects + access rule permitting ICMP and DNS (UDP/TCP 53) from inside to outside, everything else denied — deploy, then verify via API GET and FTD CLI; plan the negative tests for Module 10 | Working REST script + deployed rule; GET and CLI evidence captured |
| 145–160 | Independent practice: modify or add one rule (e.g. deny a specific inside subnet to a destination, or permit one extra service), redeploy, re-verify | Second deployed change with before/after GET output |
| 160–172 | Failure clinic: 401 token expiry, 400 validation errors, deploy failures, rule-order shadowing, object reference mistakes, FDM API not enabled on the interface | Student fixes one seeded fault in their own script |
| 172–180 | Wrap-up and exit check; hand off to Module 10 (traffic tests from the Ubuntu endpoint) | Exit ticket: explain the deploy step and one permit + one deny rule |

**Planned total:** 10 + 25 + 20 + 10 + 80 + 15 + 12 + 8 = **180 minutes** (including break and wrap-up)

**Out-of-class work (not counted in the 180 minutes):** none required; optional — map the module's permit/deny rules onto the permitted/denied ping+DNS matrix rows before Module 10.

## Lab specification

- GNS3 topology and API steps: via API v3 (`requests` or gns3fy) create/start the reference topology with the FTDv 10.0.0 mandatory: 2× CSR1000v + FTDv + Ubuntu endpoint on the inside segment (+ optional IOSv/ASAv as 5th node); FTDv inside interface on the inside LAN (e.g. 10.0.0.0/24, security zone `inside`), outside interface toward the WAN/Ubuntu uplink side (zone `outside`); first-time FTD setup (management IP, `configure https` for FDM API access) via GNS3 console telnet host:port per the Module 4 pattern, synthetic credentials only.
- Devices, images, interfaces, and services: FTDv 10.0.0 — FDM REST API at `https://<ftd-mgmt>/api/fdm/latest/` (token auth) + SSH to FTD CLI; ASAv 9.24 (alternative firewall) — ASA REST API `https://<asa>/api/` (`rest-api agent`) + SSH; CSR1000v 17.3.8a provide routing between inside and outside (Module 8 config); Ubuntu endpoint is the traffic source for Module 10.
- Starter materials: `requirements.txt` (requests, pyats[full], netmiko pinned), `fdm_client.py` skeleton (token from env vars, `requests` session, `verify=False` lab-only, GET helper), `push_policy.py` skeleton with JSON payload templates for a network object and an access rule + deploy call, `verify_policy.py` skeleton (API GET + SSH `show access-list`), testbed from Module 7.
- Student deliverables: (1) `push_policy.py` that authenticates to FDM, creates the network object(s), creates the access rule permitting ICMP + DNS (UDP/TCP 53) inside→outside with deny-by-default for the rest, deploys the policy, and polls deploy status; (2) verification evidence — API GET showing the rule and FTD CLI `show access-list` output over SSH; (3) a short note mapping each rule to the permitted/denied matrix rows that Module 10 will test; (4) optional: the same on ASAv via `/api/` if the student chooses the alternative.
- Expected time and instructor support: the guided lab (80 min) assumes the FTDv is booted and HTTPS/FDM enabled (Module 4 bootstrap checklist); deploy on FTDv can take minutes — the lab polls deploy status instead of blocking; instructor circulates for 401/400 errors.
- Expected results and verification: token obtained (HTTP 200), rule created (201/200) and visible in GET `/api/fdm/latest/accessrules` with correct src/dst zones, networks, and services; deploy completes successfully; `show access-list` on the FTD CLI shows the ACEs; a quick CLI-side check (e.g. `show running-config access-list`, or a test ping from inside in Module 10) confirms enforcement; denied traffic is blocked by default deny.
- Reset/cleanup: delete the created access rule and network objects via API DELETE and redeploy (staged changes removed), or delete the whole GNS3 project via API v3; revoke/ignore tokens (short-lived); never commit tokens, IPs with credentials, or keys.
- Common failures and recovery: (1) 401 — token expired or wrong grant → re-authenticate; (2) 400/422 — payload validation (wrong object refs, missing fields) → read the error detail and fix the JSON, confirm field names in the on-device API explorer (`/api-explorer` / Swagger UI); (3) deploy fails — check `deployconfig` status/errors, pending changes on the device, then retry; (4) rule shadowed by an earlier rule — check rule order in GET and move/re-insert; (5) FDM API unreachable — HTTPS not enabled on the management interface → enable via FTD CLI/console (`configure https`); (6) self-signed cert error in requests → `verify=False` in the isolated lab only; (7) ASA REST changes lost after reboot → call the Write Memory API (`write memory` via `/api/cli`) — ASA REST applies to running config only; (8) inside↔outside routing missing so even permitted traffic fails — re-check the Module 8 routes and the FTDv interface security zones before blaming the rule.
- Offline or low-resource alternative: if the FTDv cannot boot in time (RAM), students walk the FDM API against the on-device API explorer with recorded requests/responses and write + unit-test the payloads locally (schema validation only, no device), completing the live push in office hours; the ASAv path can be practiced the same way via the ASA `/doc/` Doc-UI. With server pressure, run 2× CSR1000v + FTDv + Ubuntu (4 nodes).

## Assessment and AI use

- Formative checks: instructor observes the token flow, the policy push + deploy, and the API/CLI verification during the guided lab; the seeded-fault fix in the clinic.
- Assessed evidence: `push_policy.py` + deploy output, API GET and CLI verification evidence, and the rule-to-matrix mapping note — the firewall half of final-project R7 and the REST evidence for R5.
- How individual understanding is demonstrated: exit ticket — each student explains why the deploy step exists, and points to one permit and one deny entry in their own GET/CLI output; in team setups each member must run and explain their own push.
- Permitted AI support and required disclosure: AI may be used for research (API docs), payload drafting, debugging, and reflection; every use logged in `AI-USE.md` (date, tool + version, task/prompt summary, accepted/changed and why). Undocumented AI use is a finding. Never send credentials, tokens, or real-network data to AI tools; isolated authorized GNS3 lab only, synthetic credentials only; secret scan runs on any submitted PR.

## Dependencies and course fit

- Requires: Module 2 (API v3 topology), Module 4 (console bootstrap + SSH to FTD CLI), Module 5 (REST fundamentals), Module 8 (working routing so permitted traffic has a path); Module 7 testbed helpful for CLI verification; Module 3 for env-var secrets.
- Enables: Module 10 (permitted/denied ping+DNS tests from the Ubuntu endpoint depend on this policy), Module 11, and final-project R5/R7/R8.
- Overlap with other candidate modules: Module 5 taught generic REST on RESTCONF/ASA/FTD; this module is the applied policy workflow (objects → rule → deploy → verify). No overlap with Module 6 (NETCONF — not available on FTD/ASA).
- Connection to the final project: R7 firewall half (FTDv mandatory; ASAv allowed as alternative) and R5 (REST with token/basic auth + verified state change); the matrix note feeds R8's expected results.
- Scheduled course hours after inclusion: modules 1–11 = 33 h (11 × 3 h) with Module 1; 30 h if waived; 36 h with optional Module 12 — within the 21–39 scheduled-hour (20–40 h) limit.

## Risks and open questions

- [ ] Technical verification needed: on-device FTDv 10.0 check of the FDM base `/api/fdm/latest/` and the exact token endpoint path (e.g. `/api/fdm/latest/fdm/token` vs versioned `/api/fdm/vN/fdm/token`), plus the API version FTD 10.0 exposes and the exact `accessrules`/`networkobjects` field names — confirm via the on-device API explorer before lab build.
- [ ] Technical verification needed: FDM API reachability requires HTTPS enabled on the FTD management interface — include in the console-bootstrap checklist; deploy time on FTDv 10.0 measured at lab build (can take minutes; poll, don't block).
- [ ] Technical verification needed: ASA REST API exact ACL/access-rule resource paths for ASAv 9.24 via the embedded Doc-UI (`/doc/`), and the Write Memory API call — verify on device if the ASAv alternative is exercised.
- [ ] Lab validation needed: server RAM/CPU/disk sizing — FTDv ~8 GB RAM, 4 vCPUs per node — for the 5-node reference topology with a full class concurrently; FTDv boot time in GNS3 is slow (budget the 0–10 check-in accordingly).
- [ ] Open item: WireGuard VPN provisioning + per-user GNS3 account procedure document still pending (students need HTTPS+SSH reachability to the FTDv management IP).
- [ ] Risk: exact IOSv 15.x build TBD (only if the optional 5th node is IOSv); CSR1000v unlicensed throughput cap irrelevant for ping/DNS; CSR NETCONF `<hello>` on-device check still owed from Module 6 (not needed for this module).
- [ ] Owner decision needed: none — FTDv mandatory, ASAv allowed as alternative, student-chosen DNS/routing, 3 h duration (owner decisions 2026-11-21).

## Researcher recommendation

- Recommendation: include
- Rationale: firewall policy automation on the mandatory FTDv 10.0 via the FDM REST API is the heart of final-project R7/R5 and of the fixed permitted/denied verification matrix (R8); one 180-minute session covers auth → objects → rule → deploy → dual verification (API + CLI), with ASAv documented as the alternative. All protocol claims trace to official Cisco FTD/ASA REST guides or owner-confirmed decisions; the exact FTD 10.0 endpoint paths and field names are explicitly flagged for on-device verification rather than asserted.
- Next implementation steps: after owner approval via PR review, run the on-device FDM checks (base path, token endpoint, accessrules field names, deploy timing) on the server's FTDv 10.0.0, pin package versions, and write the lab guide + starter REST client under `curriculum/modules/module-09-firewall/`.
