# Module 1: Concurrency for network automation (conditional full module)

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending — course-owner PR review (owner decisions of 2026-11-21 are settled)

## Purpose and course alignment

- Course outcome supported: every later automation module (Modules 4–10) and final-project requirement R9 (concurrency used in the automation) assume students can run tasks against several devices in parallel instead of sequentially.
- Why students need this module: Cisco NetAcad Python Essentials 1/2 do not cover threading, asyncio, or multiprocessing (those are PCPP-level topics; owner-verified 2026-11-21). Configuring or polling several devices one by one multiplies wall-clock time — on the ≤5-node reference topology, sequential waits for device boot, SSH, and pings alone can consume most of a lab session.
- What this module will not cover: multiprocessing internals, GIL theory beyond a one-paragraph note, third-party async SSH libraries, and any re-teach of PE1/PE2 basics.
- Core / optional / enrichment: core, but conditional — the first ~20 minutes are the owner's placement check; the full session runs only if that assessment confirms the concurrency gap. The module is waived only by the owner's placement assessment, never by student self-report.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: functions, modules and pip installs, exceptions, file I/O, dicts/lists, string handling.
- Networking prerequisites: basic Cisco CLI (interfaces, loopbacks, ping) and the concept of SSH remote management; no routing-protocol knowledge needed yet.
- GNS3 and prior-module prerequisites: none — this is the first module. Students need a WireGuard VPN connection and a per-user GNS3 account on the remote server (provisioned by the owner); the 2–3× CSR1000v lab project is created via GNS3 API v3 (taught in depth in Module 2) or pre-provisioned by the instructor.
- Just-in-time refresher needed (if any), and why: none before the placement check; exceptions and simple generators are refreshed in passing only where the concurrency code needs them.

## Learning outcomes

By the end of the session, students can:

1. Explain when asyncio (`async`/`await`, `asyncio.gather`) fits network I/O and when `concurrent.futures` threads fit blocking SSH libraries, and write a working coroutine fanned out with `asyncio.gather` over N tasks.
2. Run netmiko/paramiko SSH sessions against 2–3 CSR1000v nodes in parallel (thread pool or asyncio) and measure wall-clock time against a sequential baseline.
3. Apply per-task error handling (timeouts, `return_exceptions=True`, try/except per device) so one unreachable or misconfigured device is reported as a partial failure without aborting the whole batch.
4. Configure interfaces/loopbacks in parallel on every CSR1000v node of a GNS3 project and verify each device's resulting state with `show` commands.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| PE1/PE2 do not cover concurrency | PE1/PE2 syllabi stop at fundamentals, OOP, modules; threading/asyncio/multiprocessing are PCPP-level | OpenEDG Python Institute, "PCAP: Programming Essentials in Python (2.0) Overview" — https://zsl.gliwice.pl/wp-content/uploads/2021/03/Python-Essentials-2-0-Overview.pdf ; Cisco NetAcad + Python Institute — https://pythoninstitute.org/netacad | PDF overview; accessed 2026-11-21 | High; owner-verified 2026-11-21 — placement check at session start still decides waive/include |
| asyncio async/await, gather, TaskGroup | stdlib coroutines/tasks; `asyncio.gather` runs awaitables concurrently; TaskGroup and `asyncio.timeout()` added in 3.11 | Python docs, "asyncio — Asynchronous I/O" — https://docs.python.org/3/library/asyncio.html ; "What's New In Python 3.11" — https://docs.python.org/3/whatsnew/3.11.html | Python 3.11+ docs; accessed 2026-11-21 | High |
| Threads for blocking I/O | `concurrent.futures.ThreadPoolExecutor` suits I/O-bound blocking calls (SSH); GIL makes threads (not processes) the right tool here | Python docs, "concurrent.futures — Launching parallel tasks" — https://docs.python.org/3/library/concurrent.futures.html | Python 3.11+ docs; accessed 2026-11-21 | High |
| netmiko for Cisco SSH | multi-vendor SSH automation built on paramiko; Cisco IOS/IOS XE platforms supported | PyPI, "netmiko" — https://pypi.org/project/netmiko/ ; docs — https://netmiko.readthedocs.io/ | 4.x (4.6.0 seen); accessed 2026-11-21 | High; pin exact version in requirements.txt; confirm against CSR1000v 17.3.8a on device |
| paramiko SSHv2 | native Python SSHv2 client; netmiko builds on it | PyPI, "paramiko" — https://pypi.org/project/paramiko/ | accessed 2026-11-21 | High; use `hostkey_verify=False` only inside the isolated lab |
| Concurrent local pings for the demo | stdlib `subprocess` can fan out `ping` calls; device-side `ping` over SSH is the alternative | Python docs, "subprocess — Subprocess management" — https://docs.python.org/3/library/subprocess.html | accessed 2026-11-21 | High |
| CSR1000v image and unlicensed behavior | Owner-confirmed CSR1000v IOS XE 17.3.8a with SSH; without a license the router boots with limited feature support and throughput capped (2.5 Mbps) — irrelevant for config/ping lab traffic | Cisco, "CSR 1000v Configuration Guide — Installing Cisco CSR 1000v Licenses" — https://www.cisco.com/c/en/us/td/docs/routers/csr1000/software/configuration/b_CSR1000v_Configuration_Guide/b_CSR1000v_Configuration_Guide_chapter_01000.html | accessed 2026-11-21 | High (owner-confirmed image 2026-11-21); throughput cap = risk only if bulk transfers are planned |
| Per-task timeouts and progress reporting | `asyncio.timeout()` and `asyncio.as_completed` (3.11+) give per-device timeouts and incremental results — the idiomatic error-handling pattern for the lab | Python docs, "Coroutines and tasks" — https://docs.python.org/3/library/asyncio-task.html ; "What's New In Python 3.11" — https://docs.python.org/3/whatsnew/3.11.html | Python 3.11+; accessed 2026-11-21 | High |
| Lab project pre-creation via GNS3 API v3 | the instructor can pre-provision the 2–3× CSR1000v project through `/v3/projects`, `/v3/templates`, nodes endpoints (same pattern as Module 2) | GNS3 controller API — https://apiv3.gns3.net/ | accessed 2026-11-21 | Medium-high; payload field names validated on the installed 3.0.x (risk) |
| Lab access over VPN | students reach the GNS3 server over WireGuard with per-user accounts | https://www.wireguard.com/ ; Quick Start — https://www.wireguard.com/quickstart/ | accessed 2026-11-21 | High for the technology; course provisioning doc pending (risk) |
| netmiko project resources | official project site with platform support and examples for Cisco IOS/IOS XE SSH automation | https://netmiko.org/ | accessed 2026-11-21 | High |

> All URLs in this table were accessed on 2026-11-21.

### Platform and version constraints

- Cisco device types/images: CSR1000v IOS XE Amsterdam 17.3.8a only, 2–3 nodes in one GNS3 project (course cap: ≤5 nodes per project); SSH enabled beforehand via console bootstrap with synthetic credentials.
- GNS3 version/API requirements: remote server GNS3 3.0.x, REST API v3 (`/v3/...`); lab project created through the API or pre-provisioned; console access = telnet to server host:console_port.
- Python version and dependencies: Python 3.11+; stdlib `asyncio`, `concurrent.futures`, `subprocess`; pip packages `netmiko` and `paramiko` (pinned in requirements.txt).
- Licensing, resource, and OS assumptions: CSR1000v unlicensed throughput cap ~2.5 Mbps (fine for SSH config and pings; no bulk transfers); server RAM/CPU sizing for the 5-node topology is still open (FTDv ~8 GB RAM, 4 vCPUs); students work from any OS with Python 3.11+ over WireGuard.
- Protocol/API interpretation (especially if “RPC” is involved): not applicable in this module — concurrency is transport-agnostic; NETCONF/REST appear in Modules 5–6.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–20 | Placement check (owner assessment: can students already write/run concurrent code?) + check-in; if waived, module ends here and time is released | Completed placement answers; waiver recorded by owner |
| 20–45 | Concepts: event loop, `async`/`await`, `asyncio.gather` vs `concurrent.futures` threads for blocking SSH; when each is right | Annotated comparison notes; predicts which approach fits netmiko |
| 45–55 | Break | — |
| 55–85 | Instructor demo: concurrent SSH (netmiko/paramiko) to N CSR1000v nodes plus concurrent pings; timing shown vs sequential | Students capture demo output; note where concurrency helps |
| 85–140 | Guided lab: parallel configuration of interfaces/loopbacks on 2–3 CSR1000v nodes in the GNS3 project (threads and asyncio variants) | Working script; per-device config applied; `show` verification per node |
| 140–155 | Error-handling workshop: timeouts, one node with a wrong credential, `return_exceptions=True`, per-task try/except | Partial-failure run producing a per-device success/fail report |
| 155–170 | Independent practice: students re-run with their own node set/timeout and explain the timing result | Run output + wall-clock comparison in their notes |
| 170–180 | Wrap-up and exit check | Exit check: student explains one concurrency decision in their own code |

**Planned total:** 20 + 25 + 10 + 30 + 55 + 15 + 15 + 10 = 180 minutes, including break and wrap-up

## Lab specification

- GNS3 topology and API steps: one project with 2–3 CSR1000v 17.3.8a nodes (created via GNS3 API v3 or pre-provisioned), nodes started, SSH enabled via console bootstrap (synthetic credentials only); ≤5-node course cap respected.
- Devices, images, interfaces, and services: CSR1000v IOS XE 17.3.8a; interfaces GigabitEthernet1 (links) plus loopbacks to configure; SSHv2 service after bootstrap.
- Starter materials: `requirements.txt` (netmiko, paramiko pinned), `inventory.yaml` with the 2–3 nodes (synthetic IPs/credentials via env vars), skeleton script with a sequential baseline and a `time.perf_counter` timing harness for the comparison.
- Student deliverables: (1) thread-pool script and (2) asyncio script that configure interfaces/loopbacks on all nodes in parallel; (3) error-handling variant that survives one bad node; (4) a short run log with per-device results and sequential-vs-parallel timings.
- Expected results and verification: every node has the configured loopback/interface (`show ip interface brief`, `show running-config interface LoopbackX`); parallel wall-clock measurably lower than sequential; the failing node is reported, others succeed. Verification is done by the same script over SSH (closing the loop: parallel write, per-device read).
- Reset/cleanup: remove configured loopbacks via the same script (idempotent) or delete the GNS3 project through the API; no secrets in any committed file.
- Common failures and recovery: SSH not yet enabled (bootstrap via console first); paramiko host-key prompt (lab-only `hostkey_verify=False`); timeout too short for device boot (poll SSH port first); "event loop already running" when mixing asyncio into threads (keep one `asyncio.run` per thread or run all coroutines in the main loop); threads sharing one paramiko connection (one connection per task); WireGuard session dropped mid-lab (reconnect and re-run — scripts must be idempotent).
- Offline or low-resource alternative: if the remote server is unreachable, run the same scripts against a local mock SSH/CLI simulator and local ping targets (synthetic addresses) so concurrency and error-handling logic are still exercised; record the server run for the debrief.

## Assessment and AI use

- Formative checks: placement check result; instructor observation during guided lab (parallel config, per-device verification); error-handling run produces a correct partial-failure report.
- Assessed evidence: the two scripts (threads + asyncio), the error-handling variant, and the run log with timings — submitted in the student's own project area, plus a short written reflection on when concurrency is NOT the right tool (e.g., a single device, config-order dependencies).
- How individual understanding is demonstrated: exit check where each student explains one concurrency decision in their own code and modifies it live on request (e.g., add a node, change a timeout, switch gather to return_exceptions).
- Permitted AI support and required disclosure: AI may be used for research, coding, debugging, and reflection; every use is logged in `AI-USE.md` (date, tool + version, task/prompt summary, what was accepted/changed and why). Undocumented AI use is a finding. No credentials or real-network data in AI prompts; isolated authorized GNS3 lab only, synthetic credentials only.

## Dependencies and course fit

- Requires: Python Essentials 1/2; owner placement assessment in the first 20 minutes; WireGuard VPN + per-user GNS3 account; Python 3.11+ with pip.
- Enables: Modules 4–10 (all device automation benefits from parallel device access) and final-project requirement R9 (concurrency in the automation).
- Overlap with other candidate modules: none — Module 2 covers the GNS3 API, Module 3 covers project structure; concurrency is reused there, not re-taught.
- Connection to the final project: R9 requires concurrency (e.g., parallel configuration or parallel pings across devices) with a short justification of why it is safe.
- Scheduled course hours after inclusion: 3 h for this module; course total 33 h with Module 1 included (11 × 3 h), 30 h if waived, 36 h with optional Module 12 — all within the 21–39 scheduled-hour limit.
- Out-of-class work assumed: none beyond finishing the session's scripts if unfinished; no homework is required for this module.

## Risks and open questions

- [ ] Owner decision needed: exact placement-assessment format and who administers/records it; waiver is valid only through this assessment.
- [ ] Technical verification needed: pinned netmiko/paramiko versions compatible with Python 3.11+ and with CSR1000v 17.3.8a SSH (on-device check); asyncio-vs-threads behavior differences on student machines.
- [ ] Lab validation needed: 2–3× CSR1000v topology fits server RAM/CPU (the 5-node topology with FTDv ~8 GB RAM / 4 vCPU sizing is still open); CSR1000v unlicensed throughput cap (~2.5 Mbps) is fine for ping/config but forbids bulk transfers; exact IOSv 15.x build TBD (only relevant if an IOSv node is added); WireGuard provisioning procedure document still pending; on-device checks still owed to the course: CSR1000v NETCONF `<hello>` via ncclient (Module 6) and FTD `/api/fdm/latest/` + token endpoint (Module 9).
- [ ] Delivery risk: mixed student OSes (Windows/macOS/Linux) — asyncio/threads behavior and paramiko/cryptography wheels differ slightly; the placement check should also confirm each student has a working Python 3.11+ environment before the lab starts.

## Researcher recommendation

- Recommendation: include (conditional — runs unless the owner's placement assessment waives it).
- Rationale: the concurrency gap in PE1/PE2 is verified, every later module and final-project R9 depend on it, and the 180-minute plan (placement check + concepts + demo + guided lab + error handling + debrief) is feasible in one session.
- Next implementation steps: after owner approval of this brief via PR review, prepare the placement check, the demo scripts (synthetic credentials, isolated lab), and the 2–3× CSR1000v lab project; validate netmiko/paramiko versions on the server.
