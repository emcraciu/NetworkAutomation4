# Module 7: pyATS testbeds and test structure

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending (course-owner PR review, per owner decision 2026-11-21)

## Purpose and course alignment

- Course outcome supported: final-project requirement R6 in `docs/final-project-requirements.md` — "a valid pyATS testbed (YAML) and pyATS/aetest tests that connect to the selected devices" — and course outcome 4 in `docs/course-outline.md` (valid pyATS testbed + pyATS-based automation/tests).
- Why students need this module: Modules 4–6 taught imperative device access (console, SSH, NETCONF, REST). pyATS/aetest is the course's standard harness for structuring that access as repeatable tests — connect, parse, assert, report — and it is what the Ubuntu-endpoint verification in Module 10 and the final project (R6, R8) are built on. PE1/PE2 do not cover testing frameworks.
- What this module will not cover: writing new Genie parsers from scratch (existing parsers are used), Robot Framework jobs, pyATS triggers/models beyond a mention, and traffic generation (Module 10) — this module is connect + inspect + assert on device state.
- Core / optional / enrichment: core — mandatory pyATS is in the agreed course outcomes.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: functions, modules/imports, exceptions, dicts, YAML basics (Module 3 refresher), pip/venv; no prior testing-framework experience assumed.
- Networking prerequisites: Cisco CLI basics (`show ip interface brief`, `show ip route`), SSH remote management, and the reference-topology addressing from Modules 2–4.
- GNS3 and prior-module prerequisites: Module 2 (topology created through GNS3 API v3, console via telnet host:port), Module 3 (project structure, secrets via environment variables), Module 4 (console bootstrap + SSH working, so devices are reachable over SSH with synthetic credentials). Module 5/6 knowledge is helpful but not required — pyATS connections here use SSH.
- Just-in-time refresher needed (if any), and why: a 5-minute venv + `pip install pyats[full]` check at session start, because the install pulls many packages and students must confirm it before the lab.

## Learning outcomes

By the end of the session, students can:

1. Install `pyats[full]` (Python 3.11+) in a venv and construct a valid pyATS testbed YAML for the API-created reference topology (2× CSR1000v + FTDv) with `connections: ip / protocol: ssh` and credentials injected from environment variables — never committed to the file — and load it with `pyats`/`Testbed`.
2. Write an aetest test script with `CommonSetup` (connect to all testbed devices), at least two `@aetest.testcase` sections, and `CommonCleanup`, using Genie parsers on `show ip interface brief` and `show ip route` and asserting observable conditions (interface status `up`, expected route present in the routing table).
3. Execute the test script both via a job file (`pyats run job ...`) and as a plain Python script (`python test_script.py`), and read the generated report/archive to state each section's pass/fail result.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| pyATS installation | `pip install pyats[full]` installs pyATS core + aetest + Genie (parsers/ops) in one step; Python 3.11+ supported | "pyats" on PyPI — https://pypi.org/project/pyats/ ; pyATS docs — https://developer.cisco.com/docs/pyats/ | accessed 2026-11-21 | High; pin exact version at lab build |
| aetest structure | test scripts are organized as `CommonSetup` → `Testcase`(s) → `CommonCleanup`; `@aetest.subsection`, `@aetest.setup`, `@aetest.test`, `@aetest.cleanup` decorators; `testbed.connect()` connects all devices | pyATS docs, "AEtest" / getting started — https://developer.cisco.com/docs/pyats/ ; https://github.com/CiscoTestAutomation/pyats/blob/main/docs/getting_started/index.rst ; Code Exchange pyATS lab guide — https://developer.cisco.com/codeexchange/github/repo/sttrayno/pyATS-Lab-Guide/ | accessed 2026-11-21 | High |
| Running tests | `pyats run job <job.py>` runs testscripts as easypy tasks and aggregates logs/results; `pyats run testscript ...` runs one script; scripts can also be run with plain `python` using `aetest.main()`; reports/archive land under the pyATS archive directory | pyATS docs, "pyats run" CLI — https://devnet-pubhub-site.s3.amazonaws.com/media/pyats/docs/cli/pyats_run.html ; "Running TestScripts" — https://devnet-pubhub-site.s3.amazonaws.com/media/pyats/docs/aetest/run.html ; "Jobfile & Tasks" — https://devnet-pubhub-site.s3.amazonaws.com/media/pyats/docs/easypy/jobfile.html | accessed 2026-11-21 | High |
| Reports and archive | easypy job runs aggregate logs and results (per-section pass/fail) into a pyATS archive directory, which is the evidence format for later review | pyATS docs, "Getting Started" (job files aggregate logs/results) — https://github.com/CiscoTestAutomation/pyats/blob/main/docs/getting_started/index.rst ; "pyats run" CLI — https://devnet-pubhub-site.s3.amazonaws.com/media/pyats/docs/cli/pyats_run.html | accessed 2026-11-21 | High; exact archive location printed at run start — record at lab build |
| Testbed YAML schema | testbed YAML declares `devices:` with `connections:` (`ip`, `protocol: ssh`, `credentials` blocks with `default` username/password), `os:` for parser selection, and `topology`/`links` for topology-aware tests; secrets are kept out of the file | pyATS docs, "Example Testbed File" — https://pubhub.devnetcloud.com/media/pyats/docs/topology/example.html ; "Connection To Devices" — https://developer.cisco.com/docs/pyats/connection-to-devices/ | accessed 2026-11-21 | High for schema; env-var injection is done in the job file / loader (`os.environ`), which the easypy jobfile docs explicitly support for processing environment variables — testbed.yaml itself stays secret-free |
| Genie parsers | Genie ships IOS/IOS XE parsers for `show ip interface brief` and `show ip route` (`device.parse('show ip interface brief')`, `genie.libs.parser.iosxe`) returning structured dicts for assertions | genieparser repo, iosxe parsers — https://github.com/CiscoTestAutomation/genieparser/tree/main/src/genie/libs/parser/iosxe (show_interface.py, show_routing.py) ; pyATS parsing docs — https://developer.cisco.com/docs/pyats/parsing-device-output/ | accessed 2026-11-21 | High; parser output keys must be confirmed on CSR1000v 17.3.8a at lab build |
| SSH connections from pyATS | pyATS connects to devices over SSH (protocol `ssh`) using the testbed credentials; paramiko under the hood | pyATS docs, "Connection To Devices" — https://developer.cisco.com/docs/pyats/connection-to-devices/ | accessed 2026-11-21 | High; on-device SSH enablement comes from Module 4 bootstrap |
| GNS3 server and API | GNS3 server 3.0.x (GPL-3.0, no license fee), REST API v3 (`/v3/...`), console = telnet to `<server>:<console_port>`; ≤5 nodes per project (server load) | GNS3 server LICENSE — https://github.com/GNS3/gns3-server/blob/master/LICENSE ; API reference — https://api.gns3.net ; owner decision 2026-11-21 | accessed 2026-11-21 | High (owner-confirmed); on-server validation pending |
| Device protocol matrix | CSR1000v 17.3.8a: SSH + NETCONF (TCP 830) + RESTCONF (TCP 443); FTDv 10.0.0: SSH to FTD CLI + FDM REST API (no NETCONF); ASAv 9.24: SSH + ASA REST API; IOSv 15.x: SSH/console only | `docs/research/device-protocol-support.md` (owner-confirmed 2026-11-21); Cisco DevNet platform matrix — https://developer.cisco.com/site/standard-network-devices/ ; IOS XE 17.3 Programmability CG — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg.html | accessed 2026-11-21 | High (owner-confirmed); on-device `<hello>` check owed (risk below) |

### Platform and version constraints

- Cisco device types/images: lab testbed covers 2× CSR1000v IOS XE Amsterdam 17.3.8a + FTDv 10.0.0 from the API-created reference topology (≤5 nodes total; optional IOSv 15.x / ASAv 9.24 as 5th node only if server resources allow). NETCONF exists only on the CSR1000v; this module's connections are SSH.
- GNS3 version/API requirements: remote server GNS3 3.0.x, REST API v3 (`/v3/...`); project created and started through the API (`requests` or gns3fy); console via telnet host:port if bootstrap is needed.
- Python version and dependencies: Python 3.11+ in a venv; `pip install pyats[full]` (pyats, aetest, Genie parsers); `requests`/`gns3fy` for the API steps; testbed YAML contains no secrets — credentials come from environment variables.
- Licensing, resource, and OS assumptions: GNS3 3.0.x is GPLv3; FTDv 10.0 runs a 90-day evaluation without Smart Licensing and typically needs ~8 GB RAM / 4 vCPUs (server sizing still open); CSR1000v unlicensed throughput cap is irrelevant for SSH show commands; students work from any OS over WireGuard VPN with per-user GNS3 accounts.
- Protocol/API interpretation (especially if "RPC" is involved): not central here — pyATS connections in this module are SSH (paramiko-based). NETCONF (RFC 6241, TCP 830) is Module 6; REST APIs are Modules 5 and 9. "RPC" in this course means NETCONF only — no gRPC/gNMI.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–15 | Check-in: WireGuard + GNS3 reachability, venv creation, `pip install pyats[full]` smoke check; recap of Modules 2–4 (topology via API, SSH bootstrap) | Working pyATS install; testbed draft loads with `pyats` |
| 15–35 | Instructor demo: testbed YAML schema — devices, `connections` (`ip`, `protocol: ssh`), `os: iosxe`, credentials from env vars (never committed); loading the testbed | Student annotates their own testbed.yaml skeleton |
| 35–60 | Instructor demo: aetest structure — `CommonSetup` connecting all devices, testcases, `CommonCleanup`; Genie `device.parse('show ip interface brief')` / `('show ip route')`; assertions; run via `pyats run job` and plain `python`; where the report/archive lands | Student runs the demo script and opens the report |
| 60–70 | Break | — |
| 70–150 | Guided lab: build a valid testbed for 2× CSR1000v + FTDv from the student's API-created topology; write tests that connect, parse, and assert (interface up, expected route present) | Working testbed.yaml (no secrets) + test script with ≥2 testcases; green run |
| 150–165 | Independent practice: add one own testcase + assertion (e.g., loopback present, FTD management interface reachable), re-run, inspect report | Additional passing testcase; screenshot/export of the report |
| 165–175 | Troubleshooting clinic: common failures (below) with instructor | Student fixes one seeded fault in their own script |
| 175–180 | Wrap-up and exit check | Exit ticket: explain testbed credential handling and one assertion |

**Planned total:** 15 + 20 + 25 + 10 + 80 + 15 + 10 + 5 = **180 minutes** (including break and wrap-up)

**Out-of-class work (not counted in the 180 minutes):** none required; optional reading — the pyATS testbed and connection docs, bringing one question to Module 10.

## Lab specification

- GNS3 topology and API steps: via API v3 (`requests` or gns3fy) create one project with 2× CSR1000v 17.3.8a + FTDv 10.0.0 (+ optional Ubuntu endpoint as 4th node), create the links, start all nodes; reuse the Module 4 console bootstrap (telnet host:port) so every device answers SSH with the synthetic per-user credentials. ≤5-node cap respected.
- Devices, images, interfaces, and services: CSR1000v IOS XE 17.3.8a (SSH, os `iosxe` in testbed; at least one loopback and one up/up link interface per router, pre-configured in the lab setup), FTDv 10.0.0 (SSH to FTD CLI for a management/show check), Ubuntu endpoint optional this module. Services: SSH on all devices.
- Starter materials: `requirements.txt` (`pyats[full]` pinned), `testbed.yaml` skeleton with placeholders for IPs and `${ENV_VAR}` credential references (no real values), `example_test.py` skeleton with `CommonSetup`/`Testcase`/`CommonCleanup` TODOs, `example_job.py` job file that injects env vars and runs the testscript.
- Student deliverables: (1) valid `testbed.yaml` for 2× CSR1000v + FTDv (env-var credentials only), (2) `test_interfaces_routes.py` aetest script: `CommonSetup` connects all devices, ≥2 testcases parse `show ip interface brief` and `show ip route` via Genie and assert interface `up` + expected route present, `CommonCleanup` disconnects, (3) a job file, (4) sample `pyats run job` output + report/archive location.
- Expected time and instructor support: the guided lab (80 min) assumes each student already has a started API-created project from Module 2; instructor and one assistant circulate for env-var/SSH issues, with the seeded fault ready for the clinic.
- Expected results and verification: all device connections succeed in `CommonSetup`; `show ip interface brief` parsed shows the expected interface `up/up`; `show ip route` contains the expected route (e.g., the directly connected WAN prefix and the loopback of the peer after Module 8 — in this module a seeded static route is acceptable); overall result PASS in the pyATS report.
- Reset/cleanup: stop and delete the GNS3 project via API v3 (`DELETE /v3/projects/{project_id}`) when done; delete local testbed copies containing real IPs; keep only secret-free files in the project area; never commit credentials.
- Common failures and recovery: (1) connection refused — SSH not bootstrapped yet → redo Module 4 console bootstrap; (2) authentication failed — env vars not set/exported → check `export STUDENT_USER=...` before `pyats run job`; (3) YAML indentation errors → validate with a YAML linter; (4) parser returns empty dict — wrong `os:` in testbed (FTDv CLI is not `iosxe`; use a plain `execute` + string check for FTD) or command unsupported → run the command manually over SSH first; (5) host key prompt — lab-only `ssh` options in the testbed connection; (6) report hard to find — check the pyATS archive dir printed at run start (default under `~/.pyats`); (7) FTDv slow to boot — do not block the lab on it, mark its testcase skipped and run it in office hours.
- Offline or low-resource alternative: if the server is unreachable, students run the parser/assertion logic offline against saved command outputs (Genie can parse a saved `show` output string), and validate the live connections during office hours; with tight server RAM, drop to a 2-node CSR1000v-only testbed — the aetest structure is identical.

## Assessment and AI use

- Formative checks: instructor observes the venv/pyats install, the testbed load, and the guided-lab green run; the seeded-fault fix in the troubleshooting clinic.
- Assessed evidence: the secret-free `testbed.yaml`, the aetest script, the job file, and the `pyats run job` output/report — these become the skeleton of final-project requirement R6.
- How individual understanding is demonstrated: exit ticket where each student explains (a) why credentials live in env vars and not in testbed.yaml, and (b) one assertion in their own test script, and modifies it live on request (e.g., change the asserted interface/route).
- Permitted AI support and required disclosure: AI may be used for research, coding, debugging, and reflection; every use is logged in the project `AI-USE.md` table (date, tool + version, task/prompt summary, what was accepted/changed and why). Undocumented AI use is a finding. Prompts must not contain credentials or real-network data; isolated authorized GNS3 lab only, synthetic credentials only.

## Dependencies and course fit

- Requires: Module 2 (GNS3 API v3 topology), Module 3 (venv/project structure, env-var secrets), Module 4 (console bootstrap + SSH reachable); Python Essentials 1/2; Python 3.11+.
- Enables: Module 8 (pyATS ping/parse verification of routing), Module 10 (tests run from the Ubuntu endpoint), Module 11 and final-project R6/R8.
- Overlap with other candidate modules: none — Module 4 taught raw SSH, Module 6 NETCONF; this module is the first test-harness module. Module 10 reuses the testbed, not the teaching.
- Connection to the final project: R6 (valid testbed + aetest tests that connect and inspect devices) is exactly this module's deliverable, later extended with traffic tests (R8).
- Scheduled course hours after inclusion: modules 1–11 = 33 h (11 × 3 h) with Module 1; 30 h if Module 1 is waived; 36 h with optional Module 12 — all within the 21–39 scheduled-hour (20–40 h) limit.

## Risks and open questions

- [ ] Technical verification needed: exact `pyats[full]` version pinned for Python 3.11+ at lab build, and Genie parser output keys for `show ip interface brief` / `show ip route` on CSR1000v 17.3.8a (parse once on-device and record the keys).
- [ ] Technical verification needed: on-device CSR1000v 17.3.8a NETCONF `<hello>` via ncclient (carried from Module 6) and FTDv 10.0 `/api/fdm/latest/` + token endpoint (Module 9) — the testbed in later modules may reference those connections.
- [ ] Lab validation needed: server RAM/CPU/disk sizing for the 5-node reference topology (FTDv ~8 GB RAM, 4 vCPUs) before running labs for a full class concurrently.
- [ ] Lab validation needed: `pip install pyats[full]` time and disk footprint on student machines and on the server (budgeted in the 0–15 check; measure at lab build).
- [ ] Open item: WireGuard VPN provisioning + per-user GNS3 account procedure document still pending (affects access, not content).
- [ ] Risk: exact IOSv 15.x build still TBD (only if the optional 5th node is used); CSR1000v unlicensed throughput cap — irrelevant for SSH/show traffic, no bulk transfers.
- [ ] Owner decision needed: none for this module — device set, API v3, and 3-hour duration are settled (owner decisions 2026-11-21).

## Researcher recommendation

- Recommendation: include
- Rationale: pyATS is a mandatory course outcome and final-project requirement R6; the module converts the imperative access from Modules 4–6 into a repeatable connect–parse–assert–report harness in one feasible 180-minute session, using only owner-confirmed images and official pyATS/Genie/GNS3 documentation. Unverified items (parser keys, package pin, server sizing) are tracked as risks, not asserted.
- Next implementation steps: after owner approval via PR review, pin `pyats[full]`, capture on-device Genie parser output for the two `show` commands on CSR1000v 17.3.8a, and write the lab guide + starter skeleton files under `curriculum/modules/module-07-pyats/`.
