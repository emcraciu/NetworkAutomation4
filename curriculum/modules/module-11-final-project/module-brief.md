# Module 11: Final-project integration and presentation

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending (course-owner PR review, per owner decision 2026-11-21)

## Purpose and course alignment

- Course outcome supported: the required final-project outcome in `docs/course-outline.md` — GNS3 API topology → console bootstrap → SSH/NETCONF/REST automation → routing + firewall policy → pyATS verification of permitted and denied ping/DNS from the Ubuntu endpoint — plus the submission and assessment rules in `docs/final-project-requirements.md` (11 checkable requirements, pass ≥ 6/11, secret scan first, team commit rule, presentation to the 3PG team) and `projects/README.md`.
- Why students need this module: modules 2–10 built the workflow piece by piece; this session forces end-to-end integration into one runnable pipeline, teaches the PR submission mechanics (deliverables only under `projects/<name>/`, secret scan, AI-USE.md log, team commit verification), and rehearses the presentation that is the final assessment.
- What this module will not cover: new device-automation techniques (all covered in modules 2–10); the graded 3PG presentation itself, which happens after the PR is merged.
- Core / optional / enrichment: core (capstone module).

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: the full PE1/PE2 baseline (modules, exceptions, files, JSON, basic classes) plus Module 1 concurrency and Module 3 project structure/env-var secrets handling.
- Networking prerequisites: IP addressing, routing (module 8 protocol of the student's choice), firewall permit/deny semantics (module 9), ICMP and DNS.
- GNS3 and prior-module prerequisites: Modules 2 (API v3 topology), 4 (console bootstrap + SSH), 5 (REST), 6 (NETCONF/YANG), 7 (pyATS testbeds), 8 (routing), 9 (FTD policy), 10 (traffic verification from the Ubuntu endpoint) completed; WireGuard VPN access and a per-user GNS3 account working.
- Just-in-time refresher needed (if any), and why: a short checklist walkthrough of the 11 requirements (R1–R11) at session start, because the rubric is the contract for the PR; no other refresher is planned.

## Learning outcomes

By the end of the session, students can:

1. Assemble and demonstrate a single end-to-end runnable pipeline on the remote GNS3 server — API v3 topology creation, console bootstrap, SSH/NETCONF/REST configuration, routing + FTDv firewall policy, and pyATS permitted/denied ping + DNS verification from the Ubuntu endpoint.
2. Prepare a final-project pull request with all deliverables only under `projects/<name>/` that passes the pre-submission checks: secret scan (gitleaks/trufflehog or manual grep), self-check against ≥ 6 of the 11 requirements, a complete `AI-USE.md` log, and — for teams — commits from every team member under the project path.
3. Deliver a dry-run presentation (project, slides, running scripts) to a peer/3PG-style panel and incorporate the feedback before the graded presentation after merge.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| Final-project rules | 11 checkable requirements R1–R11; pass = ≥ 6/11 (50 %); secret scan runs first and a PR with secrets is rejected immediately; one round of reviewer feedback below threshold; merged PR = accepted | `docs/final-project-requirements.md` (repo, draft pending owner approval); `projects/README.md` | 2026-11-21 | High as owner decision; the requirements doc itself is still draft pending owner approval (open issue) |
| Team commit rule | Team PR must contain commits from every member under `projects/<name>/`; verified with `git log --format='%an <%ae>' -- projects/<name>/` cross-checked with the PR commit list; teams announced before Module 8 | `docs/final-project-requirements.md`; git-log docs — https://git-scm.com/docs/git-log | accessed 2026-11-21 | High |
| Secret scanning | gitleaks / trufflehog are standard OSS secret scanners usable as the pre-submission scan; manual grep is the fallback | gitleaks — https://github.com/gitleaks/gitleaks ; trufflehog — https://github.com/trufflesecurity/trufflehog | accessed 2026-11-21 | High |
| AI-use log format | `AI-USE.md` (or README section) with one table: date, tool (name + version), task/prompt summary, what was accepted/changed and why; undocumented AI use is a finding | `docs/final-project-requirements.md`; `projects/README.md` (owner decision 2026-11-21) | 2026-11-21 | High (settled format) |
| GNS3 API v3 | Server 3.0.x (GPL-3.0), REST API v3 (`/v3/...`), console via telnet host:port, ≤ 5 nodes in the reference topology; R1 requires topology created/started through the API | API reference — https://api.gns3.net ; LICENSE — https://github.com/GNS3/gns3-server/blob/master/LICENSE ; owner decision 2026-11-21 | accessed 2026-11-21 | High (owner-confirmed); on-server validation pending |
| Python clients for GNS3 | `requests` or gns3fy for project/node/link create + start | gns3fy docs — https://davidban77.github.io/gns3fy/ | accessed 2026-11-21 | Medium-high; verify against exact server version at lab build |
| NETCONF requirement (R4) | NETCONF is mandatory and possible only on CSR1000v 17.3.8a (`netconf-yang`, TCP 830); ncclient is the reference Python client; RFC 6241 defines the protocol; RESTCONF (RFC 8040) is the REST option on the same device | ncclient on PyPI — https://pypi.org/project/ncclient/ ; RFC 6241 — https://www.rfc-editor.org/rfc/rfc6241.html ; RFC 8040 — https://www.rfc-editor.org/rfc/rfc8040.html ; `docs/research/device-protocol-support.md` | accessed 2026-11-21 | High; on-device `<hello>` check still open (risk) |
| FTD REST requirement (R5/R7) | FTDv 10.0.0 exposes the FDM REST API at base `/api/fdm/latest/`, OAuth2 password-grant token via `POST /api/fdm/v1/fdm/token`; no NETCONF on FTD; FTDv is the mandatory firewall | Cisco FTD REST API guide — https://www.cisco.com/c/en/us/td/docs/security/firepower/ftd-api/guide/ftd-rest-api/ftd-rest-api-intro.html ; auth guide — https://www.cisco.com/c/en/us/td/docs/security/firepower/ftd-api/guide/ftd-rest-api/auth-ftd-rest-api.pdf | accessed 2026-11-21 | Medium-high; exact token path + `latest` alias to confirm on device (risk) |
| pyATS evidence (R6/R8) | Valid testbed YAML + aetest tests run with `pyats run job`, producing reports; installed via `pip install pyats[full]` | pyATS docs — https://developer.cisco.com/docs/pyats/ ; PyPI — https://pypi.org/project/pyats/ | accessed 2026-11-21 | High |
| Presentation rules | After merge, present project + slides + running scripts (live or recorded with the same artifacts) to the 3PG team, including permitted/denied ping+DNS results; 3PG judges — this is the final assessment | `docs/final-project-requirements.md` (presentation section) | 2026-11-21 | High as owner decision; logistics (scheduling) still open |
| Pull-request mechanics | PRs against this repo are reviewed per the agreed threshold; branch protection/merge is done by the reviewer after the checks above | GitHub pull-request docs — https://docs.github.com/en/pull-requests ; git docs — https://git-scm.com/docs/git-log | accessed 2026-11-21 | High |

### Platform and version constraints

- Cisco device types/images: CSR1000v IOS XE 17.3.8a (SSH, NETCONF TCP 830, RESTCONF TCP 443), FTDv 10.0.0 (SSH CLI + FDM REST API `/api/fdm/latest/`, token auth; mandatory), ASAv 9.24 (SSH + ASA REST API `/api/`; allowed alternative for R7/R5), IOSv 15.x (SSH/console only; optional 5th node) — per `docs/research/device-protocol-support.md`.
- GNS3 version/API requirements: server 3.0.x (GPLv3), REST API v3 (`/v3/...`), console telnet host:port, Ubuntu endpoint as Docker node, ≤ 5 nodes; student access over WireGuard VPN with per-user accounts.
- Python version and dependencies: Python 3.11+; `pyats[full]` (aetest + Genie parsers + testbed YAML), `ncclient`, `netmiko`/`paramiko`, `requests`/`gns3fy`; no secrets in any committed file (env vars only).
- Licensing, resource, and OS assumptions: FTDv 10.0 runs a 90-day evaluation without Smart Licensing and needs ~8 GB RAM / 4 vCPUs; CSR1000v unlicensed throughput cap is irrelevant for ping/DNS; Cisco images require the owner's entitlement — GNS3 does not distribute them.
- Protocol/API interpretation (especially if "RPC" is involved): "RPC" = NETCONF (RFC 6241) on CSR1000v only; RESTCONF (RFC 8040) where supported; no gRPC/gNMI in scope.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–15 | Check-in; walkthrough of the 11 requirements, the 6/11 threshold, secret-scan-first order, team commit rule, AI-USE.md, presentation logistics | Student maps their existing module artifacts to R1–R11 |
| 15–40 | Integration workshop 1: assemble pipeline stages 1–3 into one runnable set — GNS3 API v3 topology, console bootstrap, SSH/NETCONF/REST configuration | Runnable scripts for stages 1–3 in `projects/<name>/scripts/` |
| 40–55 | Break | — |
| 55–105 | Integration workshop 2: routing + FTDv firewall policy + pyATS verification from the Ubuntu endpoint; full end-to-end dry run on the remote server | End-to-end run output incl. permitted/denied ping+DNS results |
| 105–140 | Dry-run presentations to a peer/3PG-style panel: project, slides, running scripts; feedback round | Presentation draft + feedback notes per team/individual |
| 140–165 | Submission logistics: create the PR with deliverables only under `projects/<name>/`, run the secret scan, self-check ≥ 6/11, verify team commits, finalize `AI-USE.md` and README | PR opened; pre-submission checklist completed |
| 165–180 | Wrap-up and exit check: presentation scheduling after merge; open issues collected | Exit ticket: each student names their own committed part of the project |

**Planned total:** 15 + 25 + 15 + 50 + 35 + 25 + 15 = **180 minutes** (including the break and wrap-up)

## Lab specification

- GNS3 topology and API steps: the student's own reference topology created/started through API v3 (`requests`/`gns3fy`) — at most 5 nodes, e.g. 2× CSR1000v + FTDv 10.0.0 + Ubuntu endpoint (+ optional IOSv/ASAv); console bootstrap via telnet host:port where needed; all runnable from `projects/<name>/scripts/`.
- Devices, images, interfaces, and services: images only from the confirmed list (CSR1000v 17.3.8a, FTDv 10.0.0, ASAv 9.24, IOSv 15.x, Ubuntu Docker node); services: SSH, NETCONF (CSR only), REST (RESTCONF/FDM/ASA REST API), ICMP + DNS for verification; routing protocol and DNS service are the student's choice.
- Starter materials: the module 2–10 scripts the student already has, the `projects/<name>/` skeleton (`README.md`, `scripts/`, `AI-USE.md` template with the agreed table), and the R1–R11 checklist copied into the project README as a self-check.
- Student deliverables: everything under `projects/<name>/` only — nothing outside `projects/` touched in the PR. Expected layout:

  ```text
  projects/<name>/
  ├── README.md            # topology, prerequisites, setup/run, expected results, cleanup, R1–R11 self-check
  ├── AI-USE.md            # agreed table: date | tool+version | task/prompt summary | accepted/changed and why
  ├── scripts/             # GNS3 API topology, console bootstrap, SSH/NETCONF/REST config, routing+firewall
  ├── tests/               # pyATS testbed YAML (no secrets) + traffic tests run from the Ubuntu endpoint
  └── evidence/            # captured run outputs, pyATS reports, matrix results
  ```

  Pre-submission checklist (run in this order): 1. secret scan (`gitleaks`/`trufflehog` or manual grep) on the whole PR — a secret rejects the PR immediately; 2. R1–R11 self-check, at least 6/11 must pass; 3. `git log --format='%an <%ae>' -- projects/<name>/` shows every team member (teams); 4. `AI-USE.md` present and complete; 5. deliverables only under `projects/<name>/`; 6. push and open the PR.
- Expected results and verification: the pipeline runs end-to-end on the remote server; the pyATS run from the Ubuntu endpoint shows permitted ping/DNS succeeding and denied ping/DNS failing; the PR passes the secret scan and the reviewer's ≥ 6/11 check and is merged; the presentation is delivered to the 3PG team after merge.
- Reset/cleanup: delete the GNS3 project via API v3 after capturing evidence; never commit credentials/tokens/keys — synthetic credentials via env vars only; remove local copies containing real IPs or secrets.
- Common failures and recovery: (1) PR rejected by the secret scan → remove the secret, rotate it if it was ever real, re-push; (2) team member missing from `git log` under `projects/<name>/` → that member pushes a commit before acceptance; (3) a pipeline stage fails in the dry run → re-run from the module 2–10 notes, keep scripts idempotent; (4) merge conflicts on the PR → rebase onto main and re-run the secret scan; (5) remote server unavailable → use the recorded-run option allowed for the presentation, but keep trying the live run first.
- Offline or low-resource alternative: if the VPN/server is down, integration is rehearsed on a local GNS3 install or against recorded artifacts; the presentation rules explicitly allow a recorded run with the same artifacts. If server RAM is tight, shrink to 4 nodes (drop the optional IOSv/ASAv).

## Assessment and AI use

- Formative checks: the instructor observes the end-to-end dry run, the PR pre-submission checklist (secret scan output, R1–R11 self-check, team commit check, AI-USE.md), and the dry-run presentation during the session.
- Assessed evidence: the merged PR meeting ≥ 6/11 requirements after review (the accepted project), plus the 3PG presentation of project, slides, and running scripts — the final assessment of the course.
- How individual understanding is demonstrated: in the dry run and exit check, each student (or each team member) explains the part they committed and answers questions on it; the reviewer cross-checks team PRs with per-author commit logs, so a member with no commit cannot be accepted.
- Permitted AI support and required disclosure: AI may be used throughout the project (research, coding, debugging, slides); every use must be logged in `AI-USE.md` (date, tool + version, task/prompt summary, what was accepted/changed and why). Undocumented AI use is itself a finding.

## Dependencies and course fit

- Requires: Modules 1–10 (concurrency, GNS3 API v3, project structure, console/SSH, REST, NETCONF/YANG, pyATS testbeds, routing, firewall policy, traffic verification).
- Enables: course completion — the accepted final project and the 3PG presentation are the final assessment; no later module depends on this one.
- Overlap with other candidate modules: none — this is integration and assessment, not new content; optional Module 12 (Ansible) must never displace it.
- Connection to the final project: this module *is* the final project — the PR, the review threshold, and the presentation.
- Scheduled course hours after inclusion: modules 1–11 = 33 h (11 × 3 h) with Module 1; 30 h if Module 1 is waived by the owner's placement assessment; 36 h if optional Module 12 also runs — all within the 21–39 h (20–40 h) limit.

## Risks and open questions

- [ ] Owner decision needed: `docs/final-project-requirements.md` is still a draft pending owner approval — this brief assumes the owner decisions of 2026-11-21; the rubric doc must be approved before Module 11 runs.
- [ ] Technical verification needed: on-device checks — CSR1000v 17.3.8a NETCONF `<hello>` via ncclient (R4) and FTDv 10.0 `/api/fdm/latest/` + token endpoint `POST /api/fdm/v1/fdm/token` (R5/R7) — both still open and both load-bearing for the project evidence.
- [ ] Lab validation needed: server RAM/CPU/disk sizing for the 5-node reference topology (FTDv ~8 GB RAM, 4 vCPUs) under full-class concurrent load.
- [ ] Open item: WireGuard provisioning + GNS3 per-user account procedure document still pending.
- [ ] Risk: CSR1000v throughput cap without a license — irrelevant for ping/DNS, never plan bulk transfers.
- [ ] Risk: exact IOSv 15.x build still TBD (optional 5th node only).
- [ ] Risk: team PRs where one member has no commit under `projects/<name>/` cannot be accepted — enforce the early team announcement (before Module 8) and check `git log` during this session's submission logistics.
- [ ] Risk: a PR containing any secret is rejected immediately regardless of other requirements — run the secret scan before every push, not only at submission.

## Researcher recommendation

- Recommendation: include
- Rationale: the capstone operationalizes the agreed end-to-end workflow and the agreed assessment (PR + ≥ 6/11 review + 3PG presentation); every rule in this brief traces to owner-confirmed decisions of 2026-11-21 or to official sources, and unverified technical items are carried as explicit risks.
- Next implementation steps: after owner approval via PR review, approve `docs/final-project-requirements.md`, publish the `projects/<name>/` skeleton + AI-USE.md template, schedule the 3PG presentation slots, and write the integration-workshop guide under `curriculum/modules/module-11-final-project/`.
