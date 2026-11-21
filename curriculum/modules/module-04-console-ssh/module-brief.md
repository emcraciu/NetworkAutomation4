# Module 4: Console bootstrap and SSH automation

> Status: ready for review  
> Scheduled duration: 3 hours (180 minutes)  
> Owner approval: pending — course owner PR review (owner decisions of 2026-11-21)

## Purpose and course alignment

- Course outcome supported: final-project requirements R2 (console bootstrap through
  the GNS3 console to enable remote management) and R3 (SSH automation).
- Why students need this module: every later module (REST, NETCONF, pyATS, routing,
  firewall, traffic tests) assumes a reachable management plane; GNS3 devices boot with
  zero config, so a script must answer the console dialogs and apply the first config —
  that makes the API-created topology (Module 2) reproducible instead of hand-built.
- What this module will not cover: full device configuration (Modules 8–9), the
  REST/NETCONF protocols (Modules 5–6), pyATS (Module 7), general Python beyond the
  telnet/socket and SSH patterns used here.
- Core / optional / enrichment: core.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: functions, modules, `with`, exceptions,
  file I/O, `pip`/venv (refreshed in Module 3).
- Networking prerequisites: IPv4 addressing, router interface basics, local-username
  AAA, SSH vs telnet conceptually (NetAcad CCNA level).
- GNS3 and prior-module prerequisites: Module 2 (WireGuard VPN, GNS3 server 3.0.x,
  API v3 topology) and Module 3 (project structure, env-var secrets); Module 1
  concurrency is used opportunistically for parallel bootstrap, not required.
- Just-in-time refresher needed (if any), and why: a short recap of Module 2's API
  calls; socket/telnet programming is taught inline (PE1/PE2 do not cover it).

## Learning outcomes

By the end of the session, students can:

1. Create a GNS3 project with two CSR1000v nodes through API v3 and discover each
   node's console port programmatically.
2. Write a console-automation script (telnetlib3 or raw socket — `telnetlib` was
   removed in Python 3.13) that waits for boot, answers the initial dialogs, and
   applies a minimal bootstrap config: hostname, interface IP, `username ... privilege
   15`, `crypto key generate rsa`, `ip ssh version 2`, `line vty` + `transport input
   ssh`; on CSR1000v also `netconf-yang` and `restconf`.
3. Connect over SSH with netmiko and use `send_config_set`, show commands, and
   `write memory`, verifying results from the command output.
4. Diagnose and recover from the three expected bootstrap failures: device not yet
   booted, wrong console port, SSH not yet enabled (fall back to console).

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| GNS3 console = telnet to `<server>:<console_port>`; per-node port readable via API; range configurable (`console_start_port_range`) | Confirmed for GNS3 server 3.0.x | "GNS3 server configuration file" — https://docs.gns3.com/docs/using-gns3/administration/gns3-server-configuration-file | accessed 2026-11-21 | High; read the exact port from `GET /v3/projects/{id}/nodes` |
| GNS3 server 3.0.x: REST API v3 (`/v3/...`), GPLv3, no license fee; gns3fy wraps the v3 API | Owner-confirmed install choice | api.gns3.net — http://api.gns3.net/en/latest/index.html ; LICENSE — https://github.com/GNS3/gns3-server/blob/master/LICENSE ; gns3fy — https://pypi.org/project/gns3fy/ and https://davidban77.github.io/gns3fy/ | accessed 2026-11-21 | High; confirm installed gns3fy targets 3.0.x v3 endpoints |
| Python `telnetlib` removed in 3.13 (PEP 594, deprecated since 3.11); replacements telnetlib3 / Exscript | Use telnetlib3 or a raw socket | PEP 594 — https://peps.python.org/pep-0594/ ; What's New 3.13 — https://docs.python.org/3/whatsnew/3.13.html ; telnetlib docs — https://docs.python.org/3/library/telnetlib.html | accessed 2026-11-21 | High; pin venv to 3.11/3.12 or add telnetlib3 (https://pypi.org/project/telnetlib3/) |
| netmiko (wraps paramiko) offers `ConnectHandler`, `send_config_set`, `send_command`, `save_config`; paramiko implements SSHv2 | SSH-automation libraries for the lab | netmiko — https://pypi.org/project/netmiko/ ; paramiko — https://pypi.org/project/paramiko/ | accessed 2026-11-21 | High |
| SSHv2 server on IOS XE: `crypto key generate rsa`, `ip ssh version 2`, `line vty` + `transport input ssh`, local login | Bootstrap commands for CSR1000v | "Secure Shell Configuration Guide, CSR 1000V (XE 16)" — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_ssh/configuration/xe-16/sec-usr-ssh-xe-16-book/sec-secure-shell-v2.html | accessed 2026-11-21 | High for syntax; on-device check on 17.3.8a pending |
| IOS XE 17.3 enablement: `netconf-yang` (NETCONF, TCP 830) and `restconf` (RESTCONF, TCP 443) | Applied in bootstrap so Modules 5–6 work | "Programmability Configuration Guide, Cisco IOS XE Amsterdam 17.3.x" — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg.html | accessed 2026-11-21 | High; RESTCONF HTTPS also needs `ip http secure-server` — verify in Module 5 |
### Platform and version constraints

- Cisco device types/images: CSR1000v IOS XE Amsterdam 17.3.8a only in this lab (2
  nodes, within the ≤5-node limit). ASAv 9.24, FTDv 10.0.0, IOSv 15.x are SSH-capable
  but reserved for later modules; exact IOSv build still TBD (open item).
- GNS3 version/API requirements: GNS3 server 3.0.x (GPLv3), REST API v3 (`/v3/...`),
  console via telnet `host:console_port`; project created through the API (GUI optional
  for visualization only).
- Python version and dependencies: Python 3.11+; `netmiko`, `paramiko`, `telnetlib3`
  (or stdlib `socket`), `requests`/`gns3fy` from Module 2; credentials only via env
  vars (Module 3), synthetic values in examples.
- Licensing, resource, and OS assumptions: Cisco images need the owner's entitlement
  (GNS3 does not distribute them); CSR1000v's unlicensed throughput cap is irrelevant
  for config/ping traffic; FTDv (~8 GB RAM) not used here; any OS over WireGuard VPN.
- Protocol/API interpretation (especially if “RPC” is involved): the console is a raw
  telnet stream, not an RPC; the only RPC-style interface introduced is NETCONF
  (enabled here, used in Module 6). SSH is this module's management transport.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–10 (10 min) | Check-in: VPN + GNS3 API v3 reachability; recap Module 2 project-creation calls | Student re-creates an empty project via API and lists nodes |
| 10–30 (20 min) | Console in Python: telnet `host:console_port`; why `telnetlib` is gone in 3.13; telnetlib3 vs raw socket; instructor demo | Student reads a node's `console` port from API v3 and opens a console session |
| 30–55 (25 min) | Bootstrap script walkthrough: wait-for-boot, answering the initial dialog with "no", minimal config incl. SSH + `netconf-yang`/`restconf` | Student annotates the starter script and names the prompt patterns it matches |
| 55–70 (15 min) | Break | — |
| 70–115 (45 min) | Guided lab 1: API-create a 2× CSR1000v topology; bootstrap both from zero over console | Both devices get hostname, mgmt IP, local user, SSH, `netconf-yang`, `restconf` |
| 115–145 (30 min) | Guided lab 2: SSH with netmiko — `send_config_set`, show commands, `write memory` | `show ip interface brief` up/up; running-config has SSH lines; config saved |
| 145–160 (15 min) | Failure-recovery drill: not booted yet, wrong console port, SSH not enabled | Student recovers each case via API status checks and console fallback |
| 160–172 (12 min) | Independent practice: make bootstrap idempotent; add one loopback over SSH | Re-run is a no-op; loopback visible in `show ip interface brief` |
| 172–180 (8 min) | Wrap-up and exit check | Exit check: explain the bootstrap flow and one failure mode; preview Module 5 |

**Planned total:** 180 minutes (includes the 15-minute break and the wrap-up)

## Lab specification

- GNS3 topology and API steps: create project (`POST /v3/projects`), add two CSR1000v
  nodes from the server image list, start them, read each node's console port and
  status via `GET /v3/projects/{project_id}/nodes` (gns3fy or `requests`); ≤5-node
  limit respected (2 nodes).
- Devices, images, interfaces, and services: 2× CSR1000v IOS XE 17.3.8a; one interface
  per device (e.g. GigabitEthernet1) gets an IP from the lab management subnet;
  bootstrap enables SSH (TCP 22), NETCONF (TCP 830), RESTCONF (TCP 443). Synthetic
  credentials only (`username student privilege 15 secret ...` from env vars).
- Starter materials: `bootstrap_console.py` skeleton (connect → wait → answer dialogs
  → minimal config → verify), `ssh_config.py` skeleton (netmiko `ConnectHandler`,
  `send_config_set`, `send_command`, `save_config`), `.env.example` (names only).
- Student deliverables: working `bootstrap_console.py` + `ssh_config.py`, a short
  README with run steps and verification output, AI-USE.md entries for AI help.
- Expected results and verification: both nodes reachable by SSH over the VPN;
  `show ip interface brief` shows the mgmt interface up/up; `show running-config`
  includes `ip ssh version 2`, `transport input ssh`, `netconf-yang`, `restconf`;
  `write memory` succeeds; TCP 830 accepts a connection (full `<hello>` check is
  Module 6).
- Reset/cleanup: stop nodes and delete the project via API (`POST
  .../nodes/{node_id}/stop`, `DELETE /v3/projects/{project_id}`); nothing persists.
- Common failures and recovery: device not booted (CSR1000v can take minutes — poll
  node status via API, wait/retry); wrong console port (re-read `console` from the
  API); SSH not enabled yet (fall back to console); slow first RSA key generation
  (wait for the prompt); partial config (keep every bootstrap command idempotent).
- Offline or low-resource alternative: if the server is unavailable, pair on a single
  shared CSR1000v; netmiko can also run against any SSH-reachable lab host (Linux
  container with sshd, `device_type="linux"`); console logic dry-runs on a recorded
  transcript.

## Assessment and AI use

- Formative checks: instructor circulates in the guided labs, asking individual
  students to explain one prompt pattern and one netmiko call; exit check at 172–180.
- Assessed evidence: the two working scripts + README verification output + exit check;
  feeds final-project requirements R2 and R3.
- How individual understanding is demonstrated: each student re-runs their own
  bootstrap against a fresh API-created topology and explains, unprompted, how the
  script decides the device has booted and why SSH fails before bootstrap completes.
- Permitted AI support and required disclosure: AI may be used for drafting,
  debugging, and explaining, but every use must be logged in the project `AI-USE.md`
  table (date, tool + version, task/prompt summary, what was accepted or changed and
  why); undocumented AI use is a finding. Never paste credentials/secrets into AI
  tools or commits; synthetic values and env vars only.

## Dependencies and course fit

- Requires: Module 2 (WireGuard access, GNS3 server 3.0.x, API v3 topology) and
  Module 3 (venv, project structure, env-var secrets); Module 1 concurrency helpful
  for parallel bootstrap, not mandatory.
- Enables: Module 5 (REST needs mgmt IP + REST agents enabled here), Module 6
  (NETCONF needs `netconf-yang` + credentials), Modules 7–10 (need management
  reachability), final-project R2 + R3.
- Overlap with other candidate modules: none — Module 2 stops at topology creation;
  the console-to-SSH handoff is unique to this module.
- Connection to the final project: direct implementation of R2 (console bootstrap via
  telnet host:port) and R3 (SSH automation).
- Scheduled course hours after inclusion: modules 2–11 = 10 × 3 h = 30 h (33 h with
  Module 1, 36 h with optional Module 12) — one 3-hour session inside that envelope,
  total within 21–39 h.

## Risks and open questions

- [ ] Technical verification needed: on-device check that CSR1000v 17.3.8a answers the
      initial dialog over the GNS3 console and the bootstrap commands apply cleanly.
- [ ] Technical verification needed: CSR1000v NETCONF `<hello>` via ncclient (checked
      fully in Module 6; only TCP 830 reachability here).
- [ ] Lab validation needed: CSR1000v boot time on the server — if it exceeds ~10 min
      the 45-min guided lab is tight; consider pre-booting nodes in setup.
- [ ] Owner decision needed: WireGuard provisioning + GNS3 account procedure document
      still pending; this module assumes students can already reach the server.
- [ ] Lab validation needed: server RAM/CPU/disk sizing for the later 5-node topology
      (FTDv ~8 GB RAM, 4 vCPUs) — not blocking this 2-node module; exact IOSv 15.x build
      TBD (only if an IOSv node is later bootstrapped).
- [ ] Risk: CSR1000v unlicensed throughput cap is irrelevant for config/ping traffic;
      Python version drift (3.13+ removes `telnetlib`) is handled by telnetlib3/raw
      socket in the starter code and the Module 3 venv pin.

## Researcher recommendation

- Recommendation: include
- Rationale: console bootstrap is the irreducible first step of the agreed final
  workflow (GNS3 API → console → remote management); it is small, fully automatable on
  the confirmed images, and unlocks every later module. SSH/netmiko is the workhorse
  transport for Modules 7–10 and final-project R3.
- Next implementation steps: after owner approval via PR review, implement the two
  starter scripts and lab README here; validate the bootstrap on two fresh CSR1000v
  nodes on the remote server; record actual boot time for the plan.
