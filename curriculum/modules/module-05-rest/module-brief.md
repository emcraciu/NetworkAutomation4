# Module 5: REST API automation

> Status: ready for review  
> Scheduled duration: 3 hours (180 minutes)  
> Owner approval: pending — course owner PR review (owner decisions of 2026-11-21)

## Purpose and course alignment

- Course outcome supported: final-project requirement R5 (REST where supported:
  RESTCONF on CSR1000v 17.3.8a, ASA REST API on ASAv 9.24, FDM REST API on FTDv 10.0;
  controlled change verified by response and device state) and R7's REST firewall path.
- Why students need this module: REST is the only management-plane API on the
  firewalls (ASAv, FTDv), where NETCONF does not exist; RESTCONF on the CSR1000v is the
  HTTP face of the same YANG datastores NETCONF uses in Module 6.
- What this module will not cover: NETCONF/YANG depth (Module 6), pyATS (Module 7),
  firewall policy semantics beyond one demo change (Module 9), gRPC/gNMI.
- Core / optional / enrichment: core.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: dicts and JSON, exceptions, `with`
  (for `requests` sessions), string formatting.
- Networking prerequisites: HTTPS/TLS conceptually, basic auth vs token auth,
  loopback interfaces, ACL/object concepts at CCNA level.
- GNS3 and prior-module prerequisites: Module 2 (API v3 topology) and Module 4
  (bootstrapped management IP; REST agent enable on ASAv/FTDv is part of this lab).
- Just-in-time refresher needed (if any), and why: short recap of HTTP methods and
  status codes — PE1/PE2 cover JSON but not HTTP client patterns.

## Learning outcomes

By the end of the session, students can:

1. Explain the RESTCONF resource model on IOS XE 17.3.8a (module-qualified
   `/restconf/data/...` paths, `application/yang-data+json`, basic auth, HTTPS 443)
   and perform GET, POST, and PATCH against a CSR1000v with a `requests` session.
2. Enable and use the ASA REST API (ASAv 9.24: `rest-api agent`, `/api/` objects +
   access rules) and the FDM REST API (FTDv 10.0: token auth, `/api/fdm/latest/`) for
   one controlled change per available device.
3. Verify every REST change twice: from the HTTP response/status code and from the
   resulting device state (REST GET plus device CLI over SSH).
4. Handle the expected failure modes: 401/403 auth failures, 404 wrong path, 4xx
   invalid payload, FTD token expiry, and self-signed lab certificates (lab-only
   `verify=False`, never hard-coded secrets).

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| RESTCONF = HTTP CRUD over YANG-defined datastores, compatible with a NETCONF server (not a replacement) | What students build against | RFC 8040 "RESTCONF Protocol" — https://www.rfc-editor.org/rfc/rfc8040.html | Standards Track, Jan 2017; accessed 2026-11-21 | High |
| RESTCONF on IOS XE 17.3: `restconf` command, HTTPS 443, `/restconf/data/<module>:<path>`, `application/yang-data+json`, basic auth; guide example `curl -k -u user:pass https://<ip>:443/restconf/data/Cisco-IOS-XE-native:native/...` | RESTCONF workflow on CSR1000v 17.3.8a | "Programmability Configuration Guide, Cisco IOS XE Amsterdam 17.3.x — RESTCONF Protocol" — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg/restconf_protocol.html | accessed 2026-11-21 | High; HTTPS also needs `ip http secure-server` — confirm on device |
| ASA REST API since ASA 9.3; agent enabled with `rest-api agent`; base `/api/`; objects (e.g. `/api/objects/...`) and access rules (`/api/accessrules`); needs `http server enable` + permitted source | ASA REST workflow on ASAv 9.24 | "ASA REST API Quick Start Guide" — https://www.cisco.com/c/en/us/td/docs/security/asa/api/quick-start/qsg-asa-api.html ; ASA 9.24(x) release notes — https://www.cisco.com/c/en/us/td/docs/security/asa/asa924/release/notes/asarn924.html | accessed 2026-11-21 | High for existence/base path; exact 9.24 object schemas checked on device |
| FTD FDM REST API: base `/api/fdm/latest/` (alias of `/api/fdm/v1/`); OAuth2 — POST to the token endpoint, then `Authorization: Bearer <token>` | FDM workflow on FTDv 10.0 | "FTD REST API Guide — Authenticating Your REST API Client Using OAuth" — https://www.cisco.com/c/en/us/td/docs/security/firepower/ftd-api/guide/ftd-rest-api/auth-ftd-rest-api.html ; DevNet "Get an Access Token" shows `https://<ftd>/api/fdm/latest/fdm/token` — https://developer.cisco.com/docs/firepower/threat-defense/get-an-access-token/ | accessed 2026-11-21 | Medium-high; exact token path on FTDv 10.0 is an on-device verify item |
| FTDv 10.0 runs a 90-day evaluation without Smart License registration; FTD 10.0 is a current release | Lab usable without licensing | "FDM Configuration Guide — Licensing the System" — https://www.cisco.com/c/en/us/td/docs/security/firepower/70/fdm/fptd-fdm-config-guide-700/fptd-fdm-license.html ; FTD 10.0 release notes — https://www.cisco.com/c/en/us/td/docs/security/secure-firewall/release-notes/threat-defense/100/device-manager-release-notes-10-0.html | accessed 2026-11-21 | Medium; confirm eval behavior on the actual image |
| NETCONF not on ASA/FTD (REST-only); IOSv has neither NETCONF nor REST | Device selection for this module | DevNet "Standard Device Interfaces & Models" — https://developer.cisco.com/site/standard-network-devices/ ; repo note `docs/research/device-protocol-support.md` (owner-confirmed 2026-11-21) | accessed 2026-11-21 | High (owner-confirmed) |
| `requests` sessions: auth, headers, JSON payloads, status codes; owner decision Python 3.11+ with `requests`/`gns3fy` | HTTP client + baseline | requests on PyPI — https://pypi.org/project/requests/ ; course-owner decisions 2026-11-21 (`docs/course-outline.md`) | accessed 2026-11-21 | High |

### Platform and version constraints

- Cisco device types/images: CSR1000v IOS XE 17.3.8a (RESTCONF, TCP 443), ASAv 9.24
  (ASA REST API, `/api/`), FTDv 10.0.0 (FDM API, `/api/fdm/latest/`, token auth); IOSv
  15.x has no REST and is excluded. NETCONF exists only on the CSR1000v.
- GNS3 version/API requirements: GNS3 server 3.0.x, REST API v3; topology from Modules
  2/4 reused; ≤5 nodes total.
- Python version and dependencies: Python 3.11+; `requests`. Self-signed device certs:
  lab-only `verify=False` with a comment; never log/commit tokens; creds via env vars.
- Licensing, resource, and OS assumptions: FTDv typically ~8 GB RAM / 4 vCPUs and runs
  a 90-day evaluation without Smart Licensing; server sizing for the 5-node topology is
  an open item. CSR1000v's unlicensed throughput cap is irrelevant for small API calls.
  Students work from any OS over WireGuard VPN.
- Protocol/API interpretation (especially if “RPC” is involved): "REST where
  supported" = three different APIs (RESTCONF/RFC 8040, ASA REST API, FDM REST API),
  not one generic interface and not gRPC/gNMI; NETCONF (Module 6) is the RPC-style
  protocol on TCP 830.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–10 (10 min) | Check-in + Module 4 recap (SSH working); HTTP methods/status codes intro | Student states one device's base URL and auth mode from the bootstrap |
| 10–35 (25 min) | RESTCONF on CSR1000v 17.3.8a: `restconf`, HTTPS 443, `/restconf/data/...`, GET/POST/PATCH with `yang-data+json`; instructor demo GET | Student performs a GET and reads the returned JSON path |
| 35–55 (20 min) | The other two APIs: ASA REST API on ASAv 9.24 (`rest-api agent`, `/api/` objects + access rules); FDM REST API on FTDv 10.0 (token auth, `/api/fdm/latest/`) | Student lists the auth difference (basic vs Bearer) and base paths |
| 55–70 (15 min) | Break | — |
| 70–125 (55 min) | Guided lab: one REST workflow per available device — controlled change (CSR loopback via RESTCONF; ASA network object or ACL; FTD network object) verified by GET and device CLI over SSH | Scripts make one change each; 2xx captured; CLI shows the change |
| 125–145 (20 min) | `requests` session patterns: headers, auth, JSON, status-code checks, error handling, idempotency | Student adds status assertions and retry-on-token-expiry to the FTD client |
| 145–165 (20 min) | Independent practice + failure injection: wrong path (404), bad creds (401/403), invalid payload (4xx) | Student reproduces each failure, explains the status code and recovery |
| 165–175 (10 min) | Cleanup/reset of changed objects; delete test project via API | Devices back to bootstrapped state; project removed |
| 175–180 (5 min) | Wrap-up and exit check | Exit check: one RESTCONF path + why FTD needs a token; preview Module 6 |

**Planned total:** 180 minutes (includes the 15-minute break and the wrap-up)

## Lab specification

- GNS3 topology and API steps: reuse the Module 2/4 pattern — create the project via
  API v3, add a CSR1000v plus one firewall (FTDv 10.0 preferred: mandatory in the
  reference topology; ASAv 9.24 as alternative), bootstrap via the Module 4 console
  script, then enable the REST agent on the firewall over SSH (ASAv: `http server
  enable`, `rest-api agent`, permit the management source; FTDv: enable FDM REST API
  access for the management IP — exact steps verified on device). Stay ≤5 nodes.
- Devices, images, interfaces, and services: CSR1000v 17.3.8a (RESTCONF HTTPS 443),
  FTDv 10.0.0 (FDM API HTTPS, token auth) and/or ASAv 9.24 (ASA REST API HTTPS,
  `/api/`). Synthetic credentials via environment variables only.
- Starter materials: `rest_client.py` skeleton (`requests.Session`, env-var base
  URLs/credentials, GET/POST/PATCH helpers asserting status codes), `.env.example`.
- Student deliverables: one working script per available device performing a controlled
  change (CSR: loopback via PATCH/POST; ASAv: network object or ACL; FTD: network
  object) plus verification, and a README with request/response excerpts and CLI
  verification; AI-USE.md entries for AI help.
- Expected results and verification: RESTCONF GET 200 + JSON, PATCH 204, POST 201; ASA
  object POST 2xx and visible via GET; FTD token POST returns a token, Bearer GET/POST
  200/201. Each change is also verified on the device CLI over SSH (`show
  running-config | include Loopback`).
- Reset/cleanup: delete created objects via the API (or revert), then delete the whole
  project via `DELETE /v3/projects/{project_id}`.
- Common failures and recovery: 401/403 (wrong creds, agent off, or source IP not
  permitted — check the enable step over SSH); 404 (wrong path — RESTCONF paths are
  YANG-module-qualified, e.g. `Cisco-IOS-XE-native:native/...`); 400/422 (invalid
  payload); self-signed cert errors (lab-only `verify=False`); FTD token expiry
  (re-authenticate); FTD deploy step if required (verify on device).
- Offline or low-resource alternative: if only one device type is available, run the
  full workflow on the CSR1000v alone and study recorded ASA/FDM transcripts for the
  other two; students pair on one node; API path shapes need no topology change.

## Assessment and AI use

- Formative checks: instructor reviews each student's status-code handling during the
  guided lab; exit check at 175–180.
- Assessed evidence: per-device scripts with captured responses + README (requests,
  status codes, CLI verification) + exit check; feeds final-project R5 and R7.
- How individual understanding is demonstrated: each student explains, from their own
  code, the difference between RESTCONF basic auth and FDM token auth and how they
  verified a change two independent ways.
- Permitted AI support and required disclosure: AI may be used for drafting, debugging,
  and explaining API errors, but every use must be logged in `AI-USE.md` (date, tool +
  version, task/prompt summary, what was accepted/changed and why); undocumented AI use
  is a finding. Never paste credentials, tokens, or configs into AI tools.

## Dependencies and course fit

- Requires: Module 2 (API v3 topology, VPN), Module 3 (project structure, env-var
  secrets), Module 4 (bootstrapped management plane; REST agents enabled over
  SSH/console).
- Enables: Module 9 (firewall policy via FDM or ASA REST API), the RESTCONF-vs-NETCONF
  contrast in Module 6, final-project R5 and R7.
- Overlap with other candidate modules: deliberate contrast with Module 6 — RESTCONF
  and NETCONF expose the same YANG datastores over different transports; keep the
  comparison explicit so the modules do not duplicate.
- Connection to the final project: direct implementation of R5 (controlled REST change
  verified by response + device state) and the REST path of R7.
- Scheduled course hours after inclusion: modules 2–11 = 10 × 3 h = 30 h (33 h with
  Module 1, 36 h with optional Module 12) — one 3-hour session inside that envelope,
  total within 21–39 h.

## Risks and open questions

- [ ] Technical verification needed: exact FDM token endpoint path on FTDv 10.0
      (`/api/fdm/latest/fdm/token` per Cisco docs) and `/api/fdm/latest/` acceptance.
- [ ] Technical verification needed: whether FDM changes on FTDv 10.0 need an explicit deploy call.
- [ ] Technical verification needed: RESTCONF HTTPS enablement on CSR1000v 17.3.8a (`restconf` + `ip http secure-server`) and exact status codes.
- [ ] Technical verification needed: ASA REST API on ASAv 9.24 — agent enable, source ACL, object/ACL paths.
- [ ] Lab validation needed: server RAM/CPU sizing — FTDv typically ~8 GB RAM / 4
      vCPUs; validate the 5-node reference topology against actual server resources.
- [ ] Owner decision needed: WireGuard provisioning + GNS3 account procedure document
      still pending (assumed done after Module 2).
- [ ] Technical verification needed: exact IOSv 15.x build (TBD) — not blocking (no
      REST on IOSv, excluded here).
- [ ] Risk: CSR1000v unlicensed throughput cap — irrelevant for small REST calls.
- [ ] Risk: hard-coded tokens/passwords or global `verify=False` — mitigated by env
      vars + lab-only `verify=False`; the final project's secret scan rejects secrets.

## Researcher recommendation

- Recommendation: include
- Rationale: REST is mandatory ("REST where supported") and is the only automation API
  on the firewalls; all three confirmed APIs (RESTCONF, ASA REST API, FDM) are reachable
  in one session on the confirmed images and map to final-project requirements.
- Next implementation steps: after owner approval via PR review, implement the
  per-device starter scripts and lab README here; run the on-device checks above (FTD
  token path, FDM deploy behavior, RESTCONF HTTPS enablement, ASA agent/ACL) on the
  remote server and record results in `docs/research/device-protocol-support.md`.
