# Course outline — working draft

The course owner has confirmed the outcomes, project format, delivery
environment, and 30-hour target below. Device/API compatibility is now
researched (see `research/device-protocol-support.md`); the module sequence
below reflects the owner decisions of 2026-11-21. Lesson details and lab
validation still depend on approving the module briefs.

## Confirmed constraints

- Audience: students who have completed Python Essentials 1 and Python
  Essentials 2 through Cisco Networking Academy.
- Subject: automating Cisco switches, routers, and firewalls virtualized in
  GNS3.
- Session length: three hours per module.
- Total scheduled course length: 20–40 hours. Whole three-hour sessions yield
  7–13 modules and 21–39 scheduled hours.
- Intended subject areas include SSH, REST, RPC, and pyATS.
- NETCONF is mandatory; students also need a basic explanation of YANG.
- Concurrency is mandatory. Prefer to cover it in the prerequisite Python
  Essentials course; check that coverage, and include a focused introduction
  in Module 1 if it is missing.
- Ansible is optional and should be included only if time remains.
- Candidate Cisco GNS3 images are IOSv, CSR1000v, ASA, and FTD. Confirm actual
  image availability and supported management interfaces before designing
  device-specific labs.
- Students use a remote GNS3 server; VPN access or credentials will be
  provided. Students do not need to virtualize the network locally. GNS3 GUI
  access is OS-independent and low-resource; the GUI may be used to view
  progress, but is not required for API-driven setup.
- Students may use AI throughout their work and should document that use. The
  documentation is intended to help understand their ability to work with AI;
  its precise format and assessment are not yet defined.
- Course length: 30 hours, or 10 three-hour modules.

## Required final-project outcome

Students must demonstrate an automated workflow that:

1. Brings up the GNS3 environment/topology through the GNS3 API.
2. Uses console access for the minimum initial configuration needed to enable
   remote management.
3. Uses SSH, NETCONF, and REST where supported by the selected devices; NETCONF
   is mandatory. The project should explain the relevant YANG basics.
4. Provides a valid pyATS testbed and uses pyATS-based automation/tests to
   configure devices so traffic can flow through the topology.
5. Configures routing and firewall rules to allow selected traffic and block
   other traffic.
6. Runs pyATS tests from the Ubuntu endpoint in the topology to generate and
   verify traffic, including ping and DNS checks with expected results for
   both permitted and denied traffic.

The final project is submitted as a pull request against this repository.
Student deliverable content must be under `projects/`, with a project README,
scripts, and any other materials needed to run the automation. Individual and
team submissions are allowed. Teams must be announced early, and a team PR
must contain commits from each team member. The review checks for at least 50%
of the project requirements; once the reviewed PR is merged, the project is
considered accepted. Students also present the project, slides, and running
scripts to the 3PG team, which judges their work.

Students must not commit credentials or other secrets in the project.

## Evidence from previous attempts

The first attempt included introductory Python topics (fundamentals, control
flow, functions, classes, iterators, generators, and exceptions) before device
automation. The second expanded that sequence and later covered subprocess,
async I/O, multiprocessing, threading, SSH, pyATS, REST, Genie, firewall
automation, testing, Ansible, and GNS3 API usage. See
[`research/legacy-review.md`](research/legacy-review.md) for the review.

This history is input for deciding what to retain; it is not a proposed
module list. In particular, do not reserve whole sessions for introductory
Python without evidence that the target students need them.

## Proposed module sequence (updated 2026-11-21)

| Module | Working focus | Draft outcome |
|---|---|---|
| 1 | Concurrency for network automation (conditional full module) | Placement check at session start; if the concurrency gap is confirmed, run the full session on asyncio/threads with concurrent device access. Waived only by the owner's placement assessment (Python Essentials 1/2 confirmed not to cover concurrency). |
| 2 | Remote GNS3 environment and GNS3 API v3 | Connect over WireGuard VPN, authenticate, create/bring up the ≤5-node reference topology through the API (gns3fy or requests), and clean up. |
| 3 | Automation project structure and targeted Python/data refreshers | venv, packaging, inventory/configuration data formats, secrets via environment variables; refresh only Python features needed for the automation tasks. |
| 4 | Console bootstrap and SSH automation | Apply minimum initial configuration through the GNS3 console (telnet host:port), then connect and configure devices over SSH (Netmiko/Paramiko). |
| 5 | REST API automation | Use a supported Cisco REST API — RESTCONF on CSR1000v 17.3.8a, FDM API on FTDv 10.0, or ASA REST API on ASAv 9.24 — to make a controlled change and verify the response and resulting state. |
| 6 | NETCONF and YANG | Explain the role of YANG and use NETCONF (ncclient) against CSR1000v 17.3.8a for a configuration or operational-data task. |
| 7 | pyATS testbeds and test structure | Build a valid testbed and write pyATS/aetest tests that connect to and inspect the selected devices. |
| 8 | Routing configuration through automation | Configure student-chosen routing (static/OSPF/EIGRP) through the automation workflow and verify end-to-end reachability. |
| 9 | Firewall policy automation | Configure permit/deny rules on FTDv 10.0 (mandatory, via FDM REST API; ASAv 9.24 as alternative) so selected traffic passes and other traffic is blocked. |
| 10 | Traffic verification from the Ubuntu endpoint | Run pyATS tests from the Ubuntu node in the topology to check ping and DNS, asserting expected success AND failure against the routing/firewall policy. |
| 11 | Final-project integration and presentation | Demonstrate the full GNS3/API-to-device-to-pyATS workflow, submit the project PR, pass the 50% review, and present to the 3PG team. |
| 12 (optional) | Ansible | Time-permitting optional extension only; never displaces required outcomes. |

Each module is one three-hour session, and the final workflow is developed
incrementally from Module 2 onward. Scheduled time: modules 2–11 = 10 sessions
= 30 hours if Module 1 is waived; 33 hours with Module 1 included; 36 hours
with the optional Module 12. All variants are within the 21–39 scheduled-hour
(20–40 hour) course limit.

## Resolved decisions (owner, 2026-11-21)

- Images/versions on the remote server: CSR1000v **IOS XE 17.3.8a** (SSH,
  NETCONF, RESTCONF), ASAv **9.24** (SSH, ASA REST API), FTDv **10.0.0** (SSH
  CLI, FDM REST API; mandatory firewall in the reference topology), IOSv 15.x
  (SSH/console only). NETCONF is possible only on the CSR1000v. Evidence:
  `research/device-protocol-support.md`.
- GNS3 server: install latest **3.0.x** (GPLv3, no license fee), REST **API v3**,
  console via telnet host:port, Ubuntu endpoint as a Docker node. Student access
  over **WireGuard VPN** with per-user accounts. The course standardizes on
  3.0.x/API v3 (2.2.x and 3.0.x cannot run side-by-side on one host, and 3.x
  converts project files irreversibly); Module 2 starter code ships a small
  version shim so student scripts also run against 2.2.x/v2 by changing one
  constant. Diff documented in `research/gns3-api-v2-vs-v3.md`.
- Reference topology: at most **5 nodes** (server load), e.g. 2× CSR1000v +
  FTDv 10.0 + Ubuntu endpoint (+ optional IOSv or ASAv).
- Routing protocol and DNS service in student projects: student's choice; the
  fixed part is the permitted/denied ping + DNS verification matrix.
- Concurrency: Python Essentials 1/2 confirmed **not** to cover it, so Module 1
  is a full concurrency module, skipped only if the owner's placement assessment
  shows coverage.
- Ansible: optional Module 12 if time permits.
- AI-use evidence: log table in the project (`AI-USE.md` or README section);
  format in `projects/README.md` and `docs/final-project-requirements.md`.
- Final-project rules: draft checklist, 50% rubric (6 of 11 requirements), team
  commit verification, and presentation logistics in
  `docs/final-project-requirements.md`, pending owner approval.
- Module briefs are approved by the course owner via PR review before any
  lesson/lab implementation.

## Remaining open items (tracked as risks, not blockers)

- Server RAM/CPU/disk sizing to validate the 5-node topology (FTDv needs ~8 GB
  RAM, 4 vCPUs typically).
- WireGuard provisioning and GNS3 account procedure document.
- Exact IOSv 15.x build number.
- On-device validation: CSR1000v 17.3.8a NETCONF `<hello>` via ncclient; FTD 10.0
  FDM API path (`/api/fdm/latest/`) and token endpoint. CSR1000v throughput cap
  without a license is irrelevant for ping/DNS lab traffic.

## Outline approval checklist

- [x] Audience, project outcome, target image options, remote lab model, and
      30-hour duration are agreed.
- [x] Exact platform/image versions, APIs, and access assumptions are verified
      (owner-confirmed 2026-11-21 + Cisco/GNS3 docs; see
      `research/device-protocol-support.md`; on-device validation tracked as a risk).
- [ ] Each proposed module has a researched brief and explicit prerequisites.
- [ ] The order follows dependencies and avoids unnecessary Python repetition.
- [ ] Each session has a feasible three-hour activity plan.
- [ ] Final-project checklist, PR review threshold, and presentation logistics
      are operationally defined (draft ready: `docs/final-project-requirements.md`,
      pending owner approval).
- [x] Total scheduled time is 21–39 hours (30/33/36 hours depending on Module 1
      and optional Module 12).
- [x] AI-use documentation format and assessment criteria are explicit
      (agreed 2026-11-21; `projects/README.md`, `docs/final-project-requirements.md`).
