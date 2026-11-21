# Module 10: Traffic verification from the Ubuntu endpoint

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending (course-owner PR review, per owner decision 2026-11-21)

## Purpose and course alignment

- Course outcome supported: final-project requirement R8 in `docs/final-project-requirements.md` — "pyATS tests run from the Ubuntu endpoint inside the topology that generate and verify traffic: ping and DNS checks with expected results for both permitted and denied cases" — and course outcome 6 in `docs/course-outline.md`.
- Why students need this module: modules 4–9 prove that configuration was *delivered*; this module proves the policy actually *works* end-to-end by asserting both positive (permitted) and negative (denied) outcomes from a host inside the topology. Positive-only testing cannot detect an over-permissive firewall.
- What this module will not cover: new device configuration (covered in modules 4–9); deploying or tuning a DNS server product in depth (the DNS service is the student's choice per owner decision 2026-11-21); performance or bulk-transfer testing (the CSR1000v unlicensed throughput cap makes that meaningless in this lab).
- Core / optional / enrichment: core — required input to the final project.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: functions, modules/imports, exceptions, file I/O, JSON/dicts, basic classes — plus Module 1 concurrency (used to run matrix rows in parallel) and Module 3 project structure/secrets-via-env-vars.
- Networking prerequisites: IPv4 addressing and default gateways, ICMP echo request/reply semantics, the DNS resolution path (client → resolver → upstream/authoritative), and the permit/deny logic of the module 8–9 routing/firewall policy.
- GNS3 and prior-module prerequisites: Module 2 (GNS3 API v3 topology creation, console via telnet host:port), Module 4 (console bootstrap + SSH), Module 7 (pyATS testbeds), Module 8 (routing via automation), Module 9 (FTDv firewall policy) completed; the Ubuntu Docker node present and reachable in the reference topology.
- Just-in-time refresher needed (if any), and why: a 5-minute recap of aetest test sections (`CommonSetup`, `@testcase`, `CommonCleanup`) is embedded in the instructor demo, because pyATS was last used in Module 7 and students need the test-script skeleton again here.

## Learning outcomes

By the end of the session, students can:

1. Install `pyats[full]` inside the Ubuntu Docker node of their GNS3 topology and demonstrate a working pyATS execution from inside the container (`pyats run job` or `pyats shell`).
2. Write pyATS/aetest tests that run *from the Ubuntu endpoint* and assert both expected success (permitted cases) and expected failure (denied cases) for ping and DNS, matching the module 8–9 routing/firewall policy.
3. Produce a traffic-matrix report (src, dst, service, expected, actual, pass/fail) covering every row of the permitted/denied matrix, parsing raw ping/DNS output into structured results and pyATS reports.
4. Diagnose at least one failing check from pyATS logs/reports, recover by correcting the test expectation or the underlying configuration, and re-run the suite to green.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| pyATS installation | `pip install pyats[full]` installs the core framework plus the Genie library (aetest, parsers, triggers) in one step; Python 3.11+ supported | "pyats" on PyPI — https://pypi.org/project/pyats/ ; pyATS docs — https://developer.cisco.com/docs/pyats/ | accessed 2026-11-21 | High; pin an exact version at lab build time |
| pyATS test structure | aetest testscripts use `CommonSetup` / `@testcase` sections and run via job files (`pyats run job ...`) or `pyats shell`; the testbed YAML selects devices and connections | pyATS documentation (AEtest package, job files) — https://developer.cisco.com/docs/pyats/ | accessed 2026-11-21 | High |
| Parsing output | pyATS ships Genie parsers for Cisco CLI output (`device.parse(...)`, `pyats parse` CLI); ping/DNS probes from a Linux host are parsed with Python stdlib instead (subprocess/regex/JSON) | "Parsing Device Output — pyATS & Genie" — https://developer.cisco.com/docs/pyats/parsing-device-output/ ; Python 3.11 docs — https://docs.python.org/3.11/ | accessed 2026-11-21 | High for Cisco devices; host-side parsing approach to be validated at lab build |
| Ubuntu endpoint in GNS3 | GNS3 has a Docker node type; the built-in "Ubuntu Docker Guest" template downloads an `ubuntu` image from Docker Hub and can be created as a node through API v3 | GNS3 docs, "How to configure Docker template preferences" — https://docs.gns3.com/docs-3.1-en/web-ui/template-preferences-docker ; "Create a docker container for GNS3" — https://docs.gns3.com/docs/emulators/create-a-docker-container-for-gns3 | accessed 2026-11-21 | High; exact template name/image tag on our server still to confirm via the v3 template list (risk) |
| GNS3 server and API | GNS3 server 3.0.x is GPL-3.0 (no license fee); REST API v3 (`/v3/...`); node console = telnet to `<server>:<console_port>`; Docker nodes supported | GNS3 server LICENSE — https://github.com/GNS3/gns3-server/blob/master/LICENSE ; API reference — https://api.gns3.net ; owner decision 2026-11-21 | accessed 2026-11-21 | High (owner-confirmed install); on-server validation pending |
| Python client for GNS3 | gns3fy is a Python wrapper around the GNS3 server API for project/node/link create + start from scripts; `requests` is the fallback | gns3fy docs — https://davidban77.github.io/gns3fy/ | accessed 2026-11-21 | Medium-high; verify v3 behavior against our exact server version at lab build |
| DNS service option 1 (student choice) | dnsmasq answers DNS from local config/`/etc/hosts` and forwards to upstream resolvers — a suitable lab DNS service running inside the Ubuntu container | dnsmasq documentation — https://thekelleys.org.uk/dnsmasq/doc.html | accessed 2026-11-21 | High; in-container service management to verify at lab build |
| DNS service option 2 (student choice) | IOS XE on CSR1000v 17.3.8a can act as DNS forwarder/resolver (`ip name-server`, `ip dns server`, `ip domain lookup`) | "IP Addressing: DNS Configuration Guide, Cisco IOS XE" — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios-xe-17/books/ (cisco.com) | accessed 2026-11-21 | Medium; exact 17.3 guide chapter URL to confirm at implementation |
| Host-side probes | From the Ubuntu container, `ping` (iputils-ping) and `dig`/`nslookup` (dnsutils) or Python stdlib `socket` are the test probes; pyATS wraps them via bash/subprocess steps | Python 3.11 stdlib docs — https://docs.python.org/3.11/library/socket.html ; Ubuntu package docs for iputils-ping/dnsutils | accessed 2026-11-21 | High; package availability inside the Docker image verified at lab build |
| Verification-matrix rule | Routing protocol and DNS service are the student's choice; the fixed part is the permitted/denied ping + DNS verification matrix | Owner decision 2026-11-21 — `docs/course-outline.md`, `docs/final-project-requirements.md` (R8) | 2026-11-21 | High (settled decision, do not re-litigate) |
| aetest host-side steps | pyATS runs *on* the Ubuntu container, so ping/DNS probes are plain Linux commands wrapped in aetest sections or subprocess calls; the testbed entry for the endpoint uses a local/linux connection rather than a device connection | pyATS testbed docs — https://developer.cisco.com/docs/pyats/ | accessed 2026-11-21 | Medium-high; exact testbed connection type for a local host to confirm at lab build |

### Platform and version constraints

- Cisco device types/images: CSR1000v IOS XE Amsterdam 17.3.8a (SSH + NETCONF TCP 830 + RESTCONF TCP 443), FTDv 10.0.0 (SSH to FTD CLI + FDM REST API base `/api/fdm/latest/`, token auth; mandatory firewall), optionally ASAv 9.24 (SSH + ASA REST API `/api/`) or IOSv 15.x (SSH/console only) — per `docs/research/device-protocol-support.md`. NETCONF exists only on the CSR1000v.
- GNS3 version/API requirements: server 3.0.x, REST API v3 (`/v3/...`), reference topology of at most 5 nodes; the Ubuntu endpoint is a GNS3 Docker node with a static IP and default gateway via the inside interface (behind the FTDv).
- Python version and dependencies: Python 3.11+; inside the Ubuntu container `pip install pyats[full]` plus `iputils-ping` and `dnsutils`; on the student laptop `requests` or `gns3fy` for the API steps; testbed YAML holds no secrets (env vars for credentials).
- Licensing, resource, and OS assumptions: FTDv 10.0 runs a 90-day evaluation without Smart Licensing; FTDv typically needs ~8 GB RAM and 4 vCPUs; the CSR1000v unlicensed throughput cap is irrelevant for ping/DNS volumes; student access over WireGuard VPN with per-user GNS3 accounts; GNS3 3.0.x is GPLv3.
- Protocol/API interpretation (especially if "RPC" is involved): in this course "RPC" means NETCONF (RFC 6241) on the CSR1000v only — no gRPC/gNMI in scope. Ping = ICMP echo; DNS = UDP/TCP 53 to the student-chosen resolver.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–15 | Check-in; recap of module 8 routing and module 9 firewall policy; walk through the permitted/denied traffic matrix | Student restates the expected matrix rows for their own topology |
| 15–35 | Instructor demo: add the Ubuntu Docker node via API v3, set static IP + gateway, install `pyats[full]` in the container, run one sample ping + DNS test from the endpoint | Student records install steps and expected outputs of the demo |
| 35–50 | Break | — |
| 50–110 | Guided lab: install `pyats[full]` in the student's own Ubuntu node and write the traffic-matrix test script with positive and negative assertions | Draft aetest script with ≥ 4 matrix rows (2 ping, 2 DNS) |
| 110–150 | Independent run: execute the full permitted/denied ping + DNS matrix against the module 8–9 configuration; capture the pyATS report | Run output showing expected success AND expected failure cases |
| 150–165 | Parse and report: build the matrix table (src, dst, service, expected, actual, pass/fail) from pyATS results/JSON | Completed traffic-matrix report in the project notes |
| 165–180 | Wrap-up and exit check: cleanup/reset instructions; homework = integrate the tests into the final project | Exit ticket: explain one permitted and one denied assertion |

**Planned total:** 15 + 20 + 15 + 60 + 40 + 15 + 15 = **180 minutes** (including the break and wrap-up)

## Lab specification

- GNS3 topology and API steps: via API v3 (`requests` or gns3fy): create a project, add 2× CSR1000v + FTDv 10.0.0 + Ubuntu Docker node (+ optional IOSv/ASAv as the 5th node), create links (Ubuntu inside interface behind the FTDv), start all nodes. Ubuntu gets a static IP and its default gateway via the FTDv inside interface. Console bootstrap (if needed) via telnet to `<server>:<console_port>` as in Module 4.
- Devices, images, interfaces, and services: CSR1000v 17.3.8a running the module 8 routing configuration; FTDv 10.0.0 with the module 9 permit/deny policy (mandatory firewall); Ubuntu Docker node with a static IP, running the DNS service of the student's choice (e.g. dnsmasq in the container) or forwarding DNS via a CSR1000v (`ip dns server`); probe services: ICMP and DNS (UDP/TCP 53).
- Starter materials: the reference-topology script from Module 2, the module 8 routing configuration snippets, the module 9 FTD policy export, an empty `traffic_matrix` CSV/MD template, and an aetest skeleton (synthetic credentials only, read from env vars — no secrets in files).
- Student deliverables: `tests/traffic_tests.py` (aetest script asserting success for permitted and failure for denied ping/DNS cases), a testbed YAML pointing at the Ubuntu endpoint (and devices where needed), the filled traffic-matrix report (src, dst, service, expected, actual, pass/fail), and a sample `pyats run job` output.
- Expected results and verification: every permitted row asserts success (ICMP reply received; DNS answer returns the expected A record), every denied row asserts failure (100% packet loss/timeout; DNS NXDOMAIN/timeout/refused per policy), and the pyATS report shows all rows passed with the actual results matching the expected column of the matrix. The report table has one row per matrix entry:

  | # | src | dst | service | expected | actual | pass/fail |
  |---|---|---|---|---|---|---|
  | 1 | Ubuntu (inside) | CSR1000v loopback | ICMP echo | success (reply received) | | |
  | 2 | Ubuntu (inside) | Ubuntu/DNS service IP | DNS A query (UDP 53) | success (expected A record) | | |
  | 3 | Ubuntu (inside) | outside host beyond FTDv | ICMP echo | failure (denied by FTDv policy) | | |
  | 4 | Ubuntu (inside) | blocked name/IP | DNS A query (UDP 53) | failure (denied by FTDv policy) | | |

  (Rows 1–2 are illustrative permitted cases, rows 3–4 denied cases; the student fills in their own addressing and policy. Positive AND negative assertions are both mandatory — R8.)
- Reset/cleanup: after capturing evidence, stop and delete the GNS3 project via API v3; remove local testbed copies that contain real IPs or credentials; take a snapshot before changes if the topology will be reused.
- Common failures and recovery: (1) no pip in the container → `apt-get update && apt-get install -y python3-pip`, then retry `pip install pyats[full]`; (2) permitted ping fails → check Ubuntu default gateway, FTDv inside interface IP, NAT/route on the CSR1000v, then re-run; (3) DNS fails → check resolver reachability (`ping <dns-ip>`), that the DNS service is running in the container, and the FTDv rule for UDP/TCP 53; (4) a denied case unexpectedly succeeds → verify the FTDv deny rule is deployed and its hit counter increases; (5) DNS returns a wrong answer → flush the dnsmasq cache or fix `/etc/hosts` / `address=` entries; (6) pyATS test crashes → read the pyATS log/report, fix the assertion, re-run only the affected testcase.
- Offline or low-resource alternative: if the remote server is unavailable, run the identical pyATS suite against a local GNS3/VM lab or a recorded topology — the assertions and matrix report format are unchanged. If server RAM is tight, drop the optional 5th node and keep 2× CSR1000v + FTDv + Ubuntu.

## Assessment and AI use

- Formative checks: the instructor observes the guided-lab draft (≥ 4 matrix rows), the live matrix run against the module 8–9 configuration, and the filled traffic-matrix report during the session.
- Assessed evidence: the traffic-matrix test script, the matrix report, and the pyATS run output — later reused as final-project requirement R8 evidence.
- How individual understanding is demonstrated: in the exit check each student explains their own matrix rows and one failure they debugged; in-session teamwork is allowed, but every member must be able to reproduce and explain the report.
- Permitted AI support and required disclosure: AI may be used for drafting test code, debugging errors, and explaining pyATS output; every use must be logged in the project `AI-USE.md` table (date, tool + version, task/prompt summary, what was accepted/changed and why). Undocumented AI use is itself a finding.

## Dependencies and course fit

- Requires: Module 2 (GNS3 API v3), Module 4 (console bootstrap + SSH), Module 7 (pyATS testbeds), Module 8 (routing via automation), Module 9 (FTD firewall policy); Module 1 concurrency is helpful for parallel matrix checks; Module 3 for env-var secrets handling.
- Enables: final-project requirement R8; the Module 11 integration workshop and presentation.
- Overlap with other candidate modules: none — this module is verification, distinct from the configuration modules 8–9; Module 7 taught testbed mechanics, this module applies them to traffic generation/verification from a host.
- Connection to the final project: the deliverables of this module map directly onto R8 (pyATS from the Ubuntu endpoint, ping + DNS, permitted AND denied) and feed the expected-results section of the R10 README.
- Scheduled course hours after inclusion: modules 1–11 = 33 h (11 × 3 h) with Module 1; 30 h if Module 1 is waived by the owner's placement assessment; 36 h if the optional Module 12 also runs — all within the 21–39 h (20–40 h) course limit.

## Risks and open questions

- [ ] Technical verification needed: exact GNS3 template name / Docker image tag for the "Ubuntu Docker Guest" node on our 3.0.x server (confirm via the API v3 template list before lab build).
- [ ] Technical verification needed: on-device checks carried from earlier modules — CSR1000v 17.3.8a NETCONF `<hello>` via ncclient, and FTDv 10.0 `/api/fdm/latest/` plus the token endpoint — because module 10 verifies the policy state those modules create.
- [ ] Lab validation needed: pip/apt availability and install time for `pyats[full]` inside the Ubuntu Docker image (20 min budgeted in the demo; measure at lab build).
- [ ] Lab validation needed: server RAM/CPU/disk sizing for the 5-node reference topology (FTDv ~8 GB RAM, 4 vCPUs) before running labs for a full class concurrently.
- [ ] Open item: WireGuard VPN provisioning + GNS3 per-user account procedure document is still pending (affects student access, not module content).
- [ ] Risk: CSR1000v throughput cap without a license — irrelevant for ping/DNS lab traffic, but never plan bulk transfers through it.
- [ ] Risk: exact IOSv 15.x build still TBD (only relevant if the optional 5th node is used).
- [ ] Risk: a denied case that unexpectedly succeeds usually means the FTDv policy from module 9 was not deployed or is shadowed by an earlier permit — check rule order and hit counters before blaming the test.
- [ ] Risk: DNS tests are flaky if the chosen resolver is itself behind a denied path — keep at least one permitted DNS target reachable by design.
- [ ] Owner decision needed: none for this module — DNS service and routing protocol remain the student's choice (owner decision 2026-11-21).

## Researcher recommendation

- Recommendation: include
- Rationale: verifying permitted AND denied traffic from a host inside the topology is the fixed, checkable core of the final project (R8) and of the agreed course workflow; it closes the loop between configuration (modules 4–9) and evidence. All device/protocol claims trace to owner-confirmed decisions or official Cisco/GNS3/Python documentation; unverified items are listed as risks above, not asserted as fact.
- Next implementation steps: after owner approval via PR review, build the reference traffic matrix for the 4-node topology (2× CSR1000v + FTDv + Ubuntu), validate the in-container pyATS install time, and write the lab guide + starter test skeleton under `curriculum/modules/module-10-traffic-tests/`.
