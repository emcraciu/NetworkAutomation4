# Module 2: Remote GNS3 environment and GNS3 API v3

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending — course-owner PR review (owner decisions of 2026-11-21 are settled)

## Purpose and course alignment

- Course outcome supported: final-project requirement R1 — the GNS3 environment/topology is created and started through the GNS3 API (server 3.0.x, API v3), not by hand in a GUI.
- Why students need this module: all later modules run on the remote GNS3 server over WireGuard; students must be able to authenticate, build the ≤5-node reference topology with scripts (requests or gns3fy), read console ports for the console bootstrap (Module 4), and clean up reliably so the shared server is not exhausted. Doing this by hand in a GUI does not scale to repeatable student projects and would fail final-project R1.
- What this module will not cover: device configuration (Modules 4–9), GNS3 GUI as a control interface (visualization only), server administration (owner's job), and local virtualization on student machines.
- Core / optional / enrichment: core.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: functions, `requests`-style HTTP basics are enough to start; dict/JSON handling; pip installs.
- Networking prerequisites: basic understanding of nodes/links in a topology; console vs SSH access.
- GNS3 and prior-module prerequisites: Module 1 (concurrency) or its owner-waived placement; WireGuard VPN client installed and a per-user GNS3 account issued by the owner; Python 3.11+ locally.
- Just-in-time refresher needed (if any), and why: HTTP methods/JSON and bearer tokens are refreshed for 10 minutes inside the API walkthrough, because PE1/PE2 do not cover REST clients.

## Learning outcomes

By the end of the session, students can:

1. Connect to the remote GNS3 server over WireGuard, authenticate to API v3 (`POST /v3/access/users/authenticate`), and reuse the returned token as a Bearer header with `requests` or `gns3fy`.
2. Create a project (`POST /v3/projects`), add nodes from server templates (CSR1000v 17.3.8a, FTDv 10.0.0, Ubuntu Docker node), create links, and start the nodes via the API.
3. Read back each node's status and console port and open a console session with telnet to `<server>:<console_port>`.
4. Write and run an idempotent build script that creates the ≤5-node reference topology and prints a node-status/console-port table, plus a cleanup script that closes and deletes the project — using the course `gns3_client.py` shim (API v3 default, v2 fallback via one constant) so the same scripts run against GNS3 2.2.x or 3.0.x.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| GNS3 server 3.0.x exists and is GPLv3 | v3.0.x releases published (v3.0.3 seen); server license is GPL-3.0 — no license fee | GitHub, "GNS3/gns3-server releases" — https://github.com/GNS3/gns3-server/releases (tag v3.0.3 — https://github.com/GNS3/gns3-server/releases/tag/v3.0.3) ; LICENSE — https://github.com/GNS3/gns3-server/blob/master/LICENSE | accessed 2026-11-21 | High; owner decision: install latest 3.0.x |
| GNS3 3.0 server runs on Linux; remote-server install documented | 3.0 server is Linux-only; Windows/macOS clients use the GUI against a remote server | GNS3 docs, "Remote server" — https://docs.gns3.com/docs/getting-started/installation/remote-server/ ; "What's new in GNS3 3.0" — https://www.gns3.com/community/blog/whats-new-in-gns3-version-3-0 | accessed 2026-11-21 | High |
| API v3 authentication endpoints | Public controller OpenAPI (OAS 3.1) lists `POST /v3/access/users/authenticate` (returns access token) and `/v3/access/users/login` (JWT pair), plus `/v3/access/users/me`, `/v3/version` | GNS3 controller API, Swagger UI + openapi.json — https://apiv3.gns3.net/ | doc snapshot 3.1.0a7 dated 2026-10-21; accessed 2026-11-21 | High that `/v3/access/users/authenticate` exists; exact payload/response fields must be validated against the installed 3.0.x (open issue) |
| API v3 project/node/link endpoints | v3 controller exposes `/v3/projects`, `/v3/projects/{project_id}/nodes`, `/links`, node start/stop, templates; node objects carry `console`/`console_type` (telnet) and port | GNS3 server API docs — https://gns3-server.readthedocs.io/en/latest/ ; endpoint summary — https://api.gns3.net/en/latest/endpoints.html | accessed 2026-11-21 | Medium-high: v2→v3 field renames possible — validate payloads on the installed server before the lab (risk) |
| Console access = telnet host:console_port | owner-confirmed for this course; GNS3 console connections are TCP/telnet to the server | GNS3 docs — https://docs.gns3.com/ | accessed 2026-11-21 | High (owner-confirmed 2026-11-21); on-server check pending |
| Docker nodes (Ubuntu endpoint) | GNS3 supports Docker containers as topology nodes; Ubuntu image selectable in template preferences | GNS3 docs, "Docker support in GNS3" — https://docs.gns3.com/docs/emulators/docker-support-in-gns3 ; "Template preferences — Docker" — https://docs.gns3.com/docs-3.1-en/web-ui/template-preferences-docker | accessed 2026-11-21 | High |
| gns3fy Python client | wrapper library for the GNS3 REST API; repo research note claims v3 support, but third-party reports state gns3fy targets the v2 API and is incompatible with a v3 controller | gns3fy docs — https://davidban77.github.io/gns3fy/ ; PyPI — https://pypi.org/project/gns3fy/ ; third-party note — https://github.com/manoedinata/gns3api-py | accessed 2026-11-21 | Open issue: verify which gns3fy version (if any) works against the installed 3.0.x; default teaching path is `requests` against `/v3` (risk) |
| API v2 vs v3 difference | for the ~10 calls this course uses, the surfaces are nearly identical apart from the `/v2` vs `/v3` prefix and auth (v2: HTTP Basic per request; v3: Bearer token from `/v3/access/users/authenticate`); 2.2.x and 3.0.x cannot run side-by-side on one host and 3.x converts project files irreversibly — owner decision 2026-11-21: standardize on 3.0.x, ship a small version shim | repo research note `docs/research/gns3-api-v2-vs-v3.md` ; https://api.gns3.net ; https://apiv3.gns3.net | accessed 2026-11-21 | High; shim keeps student scripts portable across server versions |
| requests HTTP client | stable HTTP library for the API calls | PyPI, "requests" — https://pypi.org/project/requests/ | 2.32.x; accessed 2026-11-21 | High |
| WireGuard VPN access | official WireGuard client/protocol for per-user VPN access to the server | https://www.wireguard.com/ ; Quick Start — https://www.wireguard.com/quickstart/ | accessed 2026-11-21 | High for the technology; course provisioning + account procedure document still pending (risk) |
| Cisco images require entitlement; GNS3 does not distribute them | CSR1000v/ASAv/FTDv/IOSv images are installed on the server by the owner from licensed images; students only reference templates via the API | GNS3 docs, "Install an appliance from the Marketplace" — https://docs.gns3.com/docs/using-gns3/beginners/install-from-marketplace | accessed 2026-11-21 | High |
| FTDv 10.0 evaluation mode | FTDv runs a 90-day evaluation without Smart License registration — sufficient for the whole course | Cisco, FDM Configuration Guide, "Licensing the System" — https://www.cisco.com/c/en/us/td/docs/security/firepower/70/fdm/fptd-fdm-config-guide-700/fptd-fdm-license.html | accessed 2026-11-21 | High (owner-confirmed 2026-11-21) |
| gns3-server package | the 3.0.x line is the API-v3 line and is installable via PyPI/PPA on Linux | PyPI, "gns3-server" — https://pypi.org/project/gns3-server/ | accessed 2026-11-21 | High |
| GNS3 web UI (visualization only) | the 3.x web UI can visualize projects but is not required for API-driven setup | gns3-web-ui — https://github.com/GNS3/gns3-web-ui | accessed 2026-11-21 | High |

> All URLs in this table were accessed on 2026-11-21.

### Platform and version constraints

- Cisco device types/images (owner-confirmed 2026-11-21): CSR1000v IOS XE Amsterdam 17.3.8a; FTDv 10.0.0 (mandatory firewall in the reference topology; 90-day evaluation without Smart Licensing); Ubuntu endpoint as a GNS3 Docker node; optional IOSv 15.x (exact build TBD) or ASAv 9.24. Reference topology ≤5 nodes (server load), e.g. 2× CSR1000v + FTDv 10.0 + Ubuntu.
- GNS3 version/API requirements: server latest 3.0.x (GPLv3), REST API v3 (`/v3/...`), controller typically on TCP 3080; console via telnet to server host:console_port; GNS3 GUI optional, visualization only.
- Python version and dependencies: Python 3.11+; `requests` (primary) or `gns3fy` (only after v3 compatibility is verified on the server).
- Licensing, resource, and OS assumptions: Cisco images require entitlement and are already on the server (owner-provided; GNS3 does not distribute them); FTDv typically needs ~8 GB RAM and 4 vCPUs — server sizing open; CSR1000v unlicensed throughput cap ~2.5 Mbps (irrelevant for lab traffic); students on any OS with Python 3.11+ and a WireGuard client.
- Protocol/API interpretation (especially if “RPC” is involved): this module is plain REST/JSON + JWT bearer auth against the GNS3 controller — not NETCONF/gNMI; NETCONF is Module 6.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–15 | Check-in, WireGuard VPN onboarding, per-user GNS3 account verification | Each student connects to the VPN and reaches the controller (`GET /v3/version`) |
| 15–40 | GNS3 3.0.x architecture + API v3 walkthrough: authenticate → token → projects, templates, nodes, links, start, console ports, close/delete | Annotated endpoint list; token stored via env var, not hard-coded |
| 40–50 | Break | — |
| 50–75 | Instructor demo: build the reference topology via API with `requests` (authenticate, create project, add nodes from templates, link, start, print status/console ports) | Students capture the demo's status table |
| 75–135 | Guided lab: each student writes and runs a script that builds the ≤5-node reference topology (2× CSR1000v 17.3.8a + FTDv 10.0 + Ubuntu) via API v3 and prints node status + console ports; telnet to one console | Working `build_topology.py`; status table; console reached |
| 135–155 | Cleanup drill: close and delete the project via API; verify clean server state; GNS3 GUI used only to visualize | `cleanup.py` run; project gone from `GET /v3/projects` |
| 155–170 | Independent practice: re-run build → verify → cleanup from scratch, idempotently (unique project name) | Second successful run with no leftover projects |
| 170–180 | Wrap-up and exit check | Exit check: student explains one API call and one failure they recovered from |

**Planned total:** 15 + 25 + 10 + 25 + 60 + 20 + 15 + 10 = 180 minutes, including break and wrap-up

## Lab specification

- GNS3 topology and API steps: authenticate (`POST /v3/access/users/authenticate`) → create project (`POST /v3/projects`) → list templates (`GET /v3/templates`) → add nodes (CSR1000v 17.3.8a ×2, FTDv 10.0.0 ×1, Ubuntu Docker ×1; optional 5th IOSv/ASAv) → create links (`POST /v3/projects/{id}/links`) → start nodes → `GET /v3/projects/{id}/nodes` for status + console ports → telnet console check → cleanup: close (`POST /v3/projects/{id}/close`) and delete (`DELETE /v3/projects/{id}`).
- Devices, images, interfaces, and services: images only from the owner-confirmed list; links per the reference topology (CSR–CSR, CSR–FTDv, FTDv–Ubuntu); no device configuration in this module. Node names follow a per-student convention (e.g., `<user>-csr1`) so parallel student projects never collide on the shared server.
- Starter materials: `requirements.txt` (requests; gns3fy optional pending v3 check), env-var config template (server URL, username, password via environment variables / `.env` — never committed), `gns3_client.py` version shim (API v3 default; set one constant for v2), skeleton `build_topology.py` and `cleanup.py`.
- Student deliverables: `build_topology.py`, `cleanup.py`, printed node status + console port table, short README section describing run/cleanup; all synthetic credentials via env vars.
- Expected results and verification: all nodes report status `started`; console ports are listed and telnet to one port shows the device console prompt; after cleanup the project no longer appears in `GET /v3/projects`; re-running the build works (unique project name or idempotent handling). The project ID and API responses are recorded in the run log — they are the evidence for final-project R1.
- Reset/cleanup: close + delete the project; if a node is stuck, stop it first; instructor can force-delete from the server side if needed.
- Common failures and recovery: 401/403 → token expired or wrong credentials, re-authenticate; 403 with valid token → RBAC role on the 3.0.x controller lacks the permission, ask the owner (never share accounts); 409 → project name already exists, use a unique suffix; template not found → check `GET /v3/templates` for exact IDs/images actually on the server; FTDv slow to start → poll status with timeout instead of assuming failure; more than 5 nodes → server load, delete other projects; VPN down → fix WireGuard first (check `wg show`).
- Offline or low-resource alternative: if the server is unreachable, validate script logic against mocked API responses (e.g., `responses` library) and document with screenshots/recorded run; low-resource variant builds only 2 nodes (1× CSR1000v + Ubuntu). GNS3 GUI may be used to visualize the built topology but is never required.

## Assessment and AI use

- Formative checks: VPN + `GET /v3/version` reachability in the first 15 minutes; instructor observation during the guided lab (build, status table, console telnet); cleanup verified by an empty project list.
- Assessed evidence: `build_topology.py` + `cleanup.py` with run output showing the reference topology built via API v3, node statuses, console ports, and clean deletion.
- How individual understanding is demonstrated: exit check where each student walks through one API call in their own script and recovers from one injected failure (e.g., expired token, name conflict) live. In team projects, every member must still be able to explain the build script, because the final team PR is verified to contain commits from every member.
- Permitted AI support and required disclosure: AI may be used for research, coding, debugging, and reflection, logged in `AI-USE.md` (date, tool + version, task/prompt summary, what was accepted/changed and why); undocumented AI use is a finding. Credentials go only into env vars/local `.env`, never into prompts, code, or commits; isolated authorized GNS3 lab only.

## Dependencies and course fit

- Requires: Module 1 (or owner-waived placement), WireGuard VPN access, per-user GNS3 account, Python 3.11+ with `requests`.
- Enables: Modules 4–11 — console bootstrap (Module 4) needs the console ports this module produces; every later lab creates its topology via the same API pattern; final-project R1.
- Overlap with other candidate modules: none — Module 1 is concurrency, Module 3 is project structure/secrets; the API pattern learned here is reused, not re-taught.
- Connection to the final project: R1 (topology created and started through the GNS3 API, ≤5 nodes) and the console bootstrap of R2 both start from this module's scripts.
- Scheduled course hours after inclusion: 3 h for this module; running total with Modules 1–2 = 6 h; course total 30/33/36 h depending on Module 1 waiver and optional Module 12 — within 21–39 h.
- Out-of-class work assumed: installing the WireGuard client and Python 3.11+ before the session if not already done; no other homework.

## Risks and open questions

- [ ] Owner decision needed: none for content; the WireGuard provisioning + GNS3 account procedure document is still pending and blocks onboarding if late.
- [ ] Technical verification needed: exact v3 payload/response field names for projects/nodes/links/start against the installed 3.0.x (v2→v3 renames possible); whether any gns3fy release works with API v3 (default to `requests` until verified); on-device checks owed to the course: CSR1000v NETCONF `<hello>` via ncclient and FTD `/api/fdm/latest/` + token endpoint.
- [ ] Lab validation needed: server RAM/CPU/disk sizing for the 5-node reference topology (FTDv ~8 GB RAM, 4 vCPUs typical) — confirm before the lab; CSR1000v unlicensed throughput cap (~2.5 Mbps) irrelevant here; exact IOSv 15.x build TBD (only if the optional IOSv node is used); per-user project quotas/RBAC roles on the 3.0.x controller to be confirmed by the owner.
- [ ] Delivery risk: shared-server contention when many students run the lab simultaneously — stagger the guided lab or pre-create per-student projects; WireGuard onboarding throughput on day one.

## Researcher recommendation

- Recommendation: include.
- Rationale: the remote-server + API v3 workflow is the backbone of the whole course and of final-project R1/R2; all facts about the server, API, and VPN are verified against official sources, and the only open items (field-level validation, gns3fy v3 support, server sizing) are tracked as risks with a `requests`-based fallback.
- Next implementation steps: after owner approval via PR review, prepare the WireGuard onboarding one-pager, the demo/lab scripts with env-var credentials, and a server-side validation checklist (authenticate, templates list, node/link/start payloads, console telnet).
