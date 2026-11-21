# Final-project requirements and review rubric (draft for owner approval)

> Status: **draft — pending course-owner approval via PR review** (decision of
> 2026-11-21). Once approved, this document is the operational definition of the
> final-project checklist, the 50 % review threshold, team rules, and presentation
> logistics referenced in `README.md`, `docs/course-outline.md`, and `projects/README.md`.

## Scope

Every student (individual or announced team) submits one pull request against this
repository with all deliverables under `projects/<project-name>/` and nothing else
touched. The project must demonstrate the agreed end-to-end workflow on the remote
GNS3 server: GNS3 API topology → console bootstrap → SSH/NETCONF/REST automation →
routing + firewall policy → pyATS verification of permitted and denied ping/DNS
traffic from the Ubuntu endpoint.

## Requirement checklist (11 checkable requirements)

Each requirement is scored **pass / fail** by the reviewer. A requirement passes only
if the PR contains runnable evidence (code + README instructions + expected results)
and the reviewer can reproduce or clearly trace it. "Looks plausible" is not enough.

| # | Requirement | Evidence expected in the PR |
|---|---|---|
| R1 | Topology is created and started through the **GNS3 API** (server 3.0.x, API v3; the course-standard client is `requests` with the version shim from Module 2, so scripts also run against 2.2.x/v2) | Script using `requests` (gns3fy only if verified against v3) that creates the project, adds nodes/links, and starts them, plus the resulting project ID / API responses; ≤ 5 nodes |
| R2 | **Console bootstrap**: minimum initial configuration applied through the GNS3 console (telnet host:port) to enable remote management | Console-automation script (e.g. socket/telnetlib3) + bootstrap config; synthetic credentials only |
| R3 | **SSH automation** to configure devices after bootstrap | Netmiko/Paramiko or pyATS SSH code configuring at least one device and verifying via `show` output |
| R4 | **NETCONF is used** (mandatory) against a NETCONF-capable device (CSR1000v 17.3.8a) | ncclient or pyATS NETCONF code doing a config or operational task (`get`/`edit-config`), plus a short YANG-basics explanation in the README |
| R5 | **REST is used where supported** (at least one of: RESTCONF on CSR1000v 17.3.8a; ASA REST API on ASAv 9.24; FDM REST API on FTDv 10.0) | REST client code (token/basic auth) making a controlled change and verifying the response + resulting device state |
| R6 | A **valid pyATS testbed** (YAML) and pyATS/aetest tests that connect to the selected devices | `testbed.yaml` (no secrets), test script with CommonSetup/testcases, sample `pyats run job` output or report |
| R7 | **Routing and firewall policy configured through automation** so that selected traffic is permitted and other traffic is denied | Config scripts + device config excerpts; firewall rules on FTDv 10.0 (mandatory) via FDM API, or ASAv 9.24 via ASA REST API |
| R8 | **pyATS tests run from the Ubuntu endpoint inside the topology** that generate and verify traffic: ping **and** DNS checks with expected results for **both permitted and denied** cases | Test code with positive and negative assertions, expected vs actual results in README, run output |
| R9 | **Concurrency** is used in the automation (e.g. parallel configuration or parallel pings across devices) | Code showing asyncio/threads/process pool + why it is safe here |
| R10 | Project README: topology diagram/description, prerequisites, setup/run steps, expected results, cleanup/reset; **no credentials, tokens, keys, or other secrets** | README.md; reviewer runs a secret scan (e.g. `gitleaks`/`trufflehog` or manual grep) |
| R11 | **AI-use log** included (format below) | `AI-USE.md` or README section with the agreed table |

Notes:

- Routing protocol and DNS service are the student's choice (owner decision
  2026-11-21); the fixed part is the permitted/denied ping + DNS verification matrix
  in R8.
- FTDv 10.0 is the mandatory firewall in the reference topology; using ASAv 9.24
  instead is allowed and counts for R7/R5.

## Review threshold (50 %)

- 11 requirements → **pass = at least 6 of 11 requirements passed (≥ 50 %)**.
- The review is binary per requirement; there is no partial credit within a
  requirement.
- A reviewed PR that reaches the threshold is **merged** and the project is
  **accepted**. A PR below the threshold gets one round of reviewer feedback;
  the student may push fixes to the same PR.
- Reviewer checklist order: secret scan first (R10) — a PR containing secrets is
  rejected immediately regardless of other requirements.

## Teams

- Teams must be **announced early** (before Module 8 at the latest) to the course owner.
- The team PR must contain **commits from every team member** touching files under
  `projects/<project-name>/`. Verification: `git log --format='%an <%ae>' -- projects/<project-name>/`
  on the PR branch, cross-checked with the PR's commit author list. A team member
  with no commit does not count as a contributor and the PR cannot be accepted for
  that member.
- Individual submissions are always allowed and need no announcement.

## Presentation (final assessment)

After the PR is merged, the student/team presents to the **3PG team**:

1. the project and slides (topology, design choices, what was automated and why),
2. the running scripts (live demo on the remote GNS3 server or a recorded run with
   the same artifacts),
3. the pyATS verification results for permitted and denied ping/DNS traffic.

The 3PG team judges the presentation and the running scripts; this is the final
assessment of the course.

## AI-use evidence format (agreed 2026-11-21)

`AI-USE.md` (or a README section) with one table:

| Date | Tool (name + version) | Task / prompt summary | What was accepted or changed, and why |
|---|---|---|---|
| | | | |

Purpose: understand how the student works with AI. It is **not** a prompt
collection requirement and does not restrict AI use; undocumented AI use is still
a finding, so students should log honestly and briefly.
