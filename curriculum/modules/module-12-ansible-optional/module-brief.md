# Module 12: Ansible for Cisco devices (optional enrichment)

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending (course-owner PR review, per owner decision 2026-11-21)

## Purpose and course alignment

- Course outcome supported: optional enrichment on top of the required final-project workflow (`docs/course-outline.md`: "Ansible is optional and should be included only if time remains"; owner decision 2026-11-21: Module 12 runs only if time permits, after all required outcomes fit the schedule).
- Why students need this module: Ansible is the most common declarative alternative to the imperative scripts (Netmiko/Paramiko/ncclient/pyATS) built in modules 4–10; comparing both on the same reference topology shows when each approach is preferable (idempotency and desired-state vs. imperative steps and test logic).
- What this module will not cover: the required workflow itself (that is modules 2–11 and the final project); Ansible is never a substitute for the required SSH/NETCONF/REST/pyATS evidence; no advanced Ansible (AWX/Automation Platform, roles development) — one focused session only.
- Core / optional / enrichment: **optional enrichment** — must never displace a required module or final-project outcome.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: modules, YAML basics (Ansible's data format), env vars, file I/O — plus Module 3 (project structure, secrets via env vars) which this module extends with ansible-vault.
- Networking prerequisites: same as modules 8–9 (routing and firewall permit/deny semantics on the reference topology).
- GNS3 and prior-module prerequisites: Module 2 (API v3 topology on the remote server), Module 4 (SSH reachability of devices), Modules 8–9 (routing + FTD policy to replicate via Ansible); Modules 1–11 must already be scheduled/completed before this optional session runs.
- Just-in-time refresher needed (if any), and why: a 10-minute YAML/ini syntax refresher at session start, because PE1/PE2 cover JSON but not YAML, and Ansible inventories/playbooks are YAML.

## Learning outcomes

By the end of the session, students can:

1. Install and version-pin `ansible-core` plus the collections `cisco.iosxe`, `cisco.asa`, and `cisco.ftd`, and build an inventory with `group_vars` in which all credentials come from ansible-vault or environment variables — no plaintext secrets anywhere.
2. Write and run a playbook that configures the CSR1000v (e.g. hostname/interface/routing task) and pushes a permit/deny policy to the FTDv (or ASAv), then verifies the resulting device state.
3. Demonstrate idempotency by re-running the playbook and showing `changed=0` on the second run, and explain why that matters for configuration management.
4. Compare the Ansible workflow with the pyATS/ncclient/SSH scripts from modules 4–10 and state in a short written note when each approach is preferable for the final-project-style tasks.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| Cisco Ansible collections namespace | Official collections live under the `cisco.*` namespace on Ansible Galaxy / Automation Hub, including `cisco.iosxe`, `cisco.asa`, `cisco.ftd` | "Collections in the Cisco Namespace" — https://docs.ansible.com/ansible/latest/collections/cisco/index.html | accessed 2026-11-21 | High |
| cisco.iosxe | Collection for IOS XE devices (e.g. `iosxe_*` modules using `network_cli`); install via `ansible-galaxy collection install cisco.iosxe` | Galaxy — https://galaxy.ansible.com/ui/repo/published/cisco/iosxe/ ; repo — https://github.com/ansible-collections/cisco.iosxe | accessed 2026-11-21 | High; pin exact collection version in `requirements.yml` at lab build |
| cisco.asa | Collection for ASA/ASAv (e.g. `asa_config`, `network_os: asa`); note: transitioned to community maintenance — compatibility with ansible-core > 2.17 not guaranteed | "Cisco.Asa — Ansible Community Documentation" — https://docs.ansible.com/ansible/latest/collections/cisco/asa/index.html | accessed 2026-11-21 | Medium; pin ansible-core (≤ 2.17) and collection version, test at lab build (risk) |
| cisco.ftd | Collection of modules (e.g. `ftd_configuration`) that drive the FTD REST API (FDM) for provisioning/operational tasks on FTD devices | "Introduction to FTD Ansible" — https://developer.cisco.com/docs/ftd-ansible/latest/ ; repo — https://github.com/CiscoDevNet/FTDAnsible ; Galaxy — https://galaxy.ansible.com/ui/repo/published/cisco/ftd/ | accessed 2026-11-21 | Medium; confirm module compatibility with the FDM API of FTDv 10.0 on-device (risk) |
| ansible-vault | `ansible-vault encrypt_string` / encrypted variable files protect secrets at rest; vault password from prompt, file, or env (`ANSIBLE_VAULT_PASSWORD_FILE`); data in use is not protected → combine with env vars | Ansible vault docs — https://docs.ansible.com/ansible/latest/vault/index.html | accessed 2026-11-21 | High |
| Ansible install | Current `ansible-core` installs via pip on Python 3.11+; collections via `ansible-galaxy` | Ansible installation guide — https://docs.ansible.com/ansible/latest/installation_guide/ | accessed 2026-11-21 | High; pin versions for reproducibility |
| Network connection plugins | Network collections use `network_cli` (SSH) for iosxe/asa and `httpapi` (HTTPS REST) for ftd; credentials via vars/env, never committed | Ansible network guide — https://docs.ansible.com/ansible/latest/network/index.html | accessed 2026-11-21 | Medium-high; exact plugin options per collection to confirm at lab build |
| Idempotency | Network config modules report `changed` and are idempotent — a second run yields `changed=0`; this is the key contrast with one-shot imperative scripts | Ansible user guide (idempotency & `changed` status) — https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html | accessed 2026-11-21 | High |
| Comparison anchor | The modules 4–10 workflow (Netmiko/Paramiko SSH, ncclient NETCONF, REST via requests, pyATS tests) is the imperative/test-centric baseline this module compares against | `docs/course-outline.md`; module briefs 4–10 (repo) | 2026-11-21 | High (internal) |
| Lab platform | GNS3 server 3.0.x, API v3, ≤ 5-node reference topology, devices from the confirmed image list; Ansible control node = student laptop or the Ubuntu endpoint over WireGuard | API reference — https://api.gns3.net ; owner decision 2026-11-21; `docs/research/device-protocol-support.md` | accessed 2026-11-21 | High (owner-confirmed); on-server validation pending |
| Collection install CLI | `ansible-galaxy collection install cisco.iosxe cisco.asa cisco.ftd` (optionally with `--requirements-file requirements.yml`); Galaxy serves the published collections | Ansible Galaxy docs — https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#install-multiple-collections-with-a-requirements-file | accessed 2026-11-21 | High |

### Platform and version constraints

- Cisco device types/images: CSR1000v IOS XE 17.3.8a (target of `cisco.iosxe`), FTDv 10.0.0 (target of `cisco.ftd` via FDM REST API `/api/fdm/latest/`, token auth; mandatory firewall), ASAv 9.24 (target of `cisco.asa`), IOSv 15.x (SSH only — usable with `cisco.ios` if needed, not required here) — per `docs/research/device-protocol-support.md`.
- GNS3 version/API requirements: server 3.0.x (GPLv3), REST API v3 (`/v3/...`) for topology creation, console telnet host:port for bootstrap; ≤ 5 nodes; student access via WireGuard VPN with per-user accounts.
- Python version and dependencies: Python 3.11+ on the Ansible control node; `pip install ansible-core` (pinned) + `ansible-galaxy collection install cisco.iosxe cisco.asa cisco.ftd` (pinned in `requirements.yml`); the target devices need SSH (iosxe/asa) and HTTPS FDM API (ftd) enabled — done in modules 4–5/9.
- Licensing, resource, and OS assumptions: FTDv 10.0 runs a 90-day evaluation without Smart Licensing (~8 GB RAM, 4 vCPUs); CSR1000v unlicensed throughput cap irrelevant here; control node is Linux/macOS/WSL on the student side; no secrets committed — ansible-vault + env vars only.
- Protocol/API interpretation (especially if "RPC" is involved): Ansible reaches devices over SSH CLI (network_cli), RESTCONF/HTTPS (httpapi), or the FTD REST API — it does not use NETCONF; NETCONF stays covered by module 6 (ncclient). "RPC" in this course remains NETCONF (RFC 6241) on the CSR1000v only.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–10 | Check-in; positioning: inclusion rule (only after all required outcomes are scheduled), when Ansible helps vs. scripts/pyATS; 10-min YAML refresher | Student states one task they would do with Ansible vs. with pyATS |
| 10–30 | Instructor demo: install ansible-core + the three collections, build inventory + `group_vars`, encrypt a credential with ansible-vault, env-var vault password | Student records the install/pin steps and vault workflow |
| 30–45 | Break | — |
| 45–95 | Guided lab: write and run a playbook that configures the CSR1000v and pushes a permit/deny policy to the FTDv (or ASAv) against the reference topology | Playbook + inventory + vault-encrypted vars; first run output |
| 95–140 | Compare workflows: apply the same change via Ansible and via the module 4–10 pyATS/ncclient scripts; re-run the playbook to show `changed=0` (idempotency); discussion | Second-run output showing `changed=0`; notes for the comparison |
| 140–165 | Independent exercise: extend the playbook with one additional task plus a verification task; write the short comparison note (when Ansible, when pyATS/ncclient) | Extended playbook + written comparison note |
| 165–180 | Wrap-up and exit check: how this would (optionally) appear in the final project; reminder that Module 12 is optional enrichment | Exit ticket: one advantage and one limitation of Ansible vs. the modules 4–10 scripts |

**Planned total:** 10 + 20 + 15 + 50 + 45 + 25 + 15 = **180 minutes** (including the break and wrap-up)

## Lab specification

- GNS3 topology and API steps: via API v3 (`requests`/`gns3fy`): create/start the reference topology — 2× CSR1000v + FTDv 10.0.0 + Ubuntu endpoint (+ optional ASAv as 5th node) — exactly as in modules 2 and 11; the Ansible control node is the student laptop or the Ubuntu Docker node, reaching devices over the WireGuard VPN.
- Devices, images, interfaces, and services: images only from the confirmed list; CSR1000v 17.3.8a reachable via SSH (network_cli); FTDv 10.0.0 reachable via FDM REST API HTTPS with token auth (httpapi); ASAv 9.24 via SSH/ASA REST API if used; services enabled by the module 4–5 bootstrap.
- Starter materials: empty playbook directory skeleton — no secrets in any file:

  ```text
  ansible-lab/
  ├── requirements.yml      # collections: cisco.iosxe, cisco.asa, cisco.ftd (pinned versions)
  ├── inventory.ini         # hosts: csr1, ftd (or asa); ansible_network_os set per host
  ├── group_vars/
  │   └── all.yml           # credential vars referenced as lookup('env', ...) — values never stored
  │   └── vault.yml         # ansible-vault-encrypted secrets (encrypt with ansible-vault, never commit the password)
  ├── playbook.yml          # CSR1000v config task + FTDv/ASAv policy task + verification task
  └── COMPARISON.md         # Ansible vs. modules 4–10 pyATS/ncclient workflow
  ```
- Student deliverables: `playbook.yml` (CSR1000v configuration task + FTDv/ASAv policy task + at least one verification task), inventory + `group_vars` with vault-encrypted secrets, `requirements.yml`, `ansible-playbook` run output including a second run with `changed=0`, and a short `COMPARISON.md` note contrasting Ansible with the modules 4–10 pyATS/ncclient workflow.
- Expected results and verification: the playbook applies the CSR configuration and the FTD/ASA policy; verification tasks (e.g. `iosxe_command`/`asa_command`/`ftd` GET + assert, or a follow-up pyATS ping check from the Ubuntu endpoint) confirm the resulting state; the second run reports `changed=0`.
- Reset/cleanup: remove the pushed policy with an Ansible `state: absent` play or delete the whole GNS3 project via API v3; never commit the vault password — supply it via env var/file outside the repo.
- Common failures and recovery: (1) collection version mismatch → pin versions in `requirements.yml` and reinstall; (2) `cisco.asa` breaks on newer ansible-core → pin ansible-core ≤ 2.17 per the collection's community-maintenance notice; (3) vault password prompt blocks automation → use `ANSIBLE_VAULT_PASSWORD_FILE` env var; (4) FTD module incompatible with the FDM 10.0 API → fall back to the generic `ansible.builtin.uri` module against `/api/fdm/latest/` (still Ansible, still valid); (5) device unreachable → check WireGuard VPN, SSH/FDM enablement from module 4–5, and firewall rules; (6) `changed` not 0 on re-run → inspect the diff (`--diff`) and fix the non-idempotent task.
- Offline or low-resource alternative: if the remote server is unavailable, run the playbook against a local GNS3 topology; if server RAM is tight, shrink to 3 nodes (1× CSR1000v + FTDv + Ubuntu). If the module is not run at all (time does not permit), nothing is lost — it is optional enrichment and not part of the final-project rubric.

## Assessment and AI use

- Formative checks: the instructor observes the guided-lab playbook run, the idempotent second run (`changed=0`), and the comparison note during the session.
- Assessed evidence: none required — this module is optional enrichment and is not part of the 11-requirement final-project rubric; if the owner later decides Ansible evidence may strengthen a project, the playbook + run outputs can be added under `projects/<name>/` like any other artifact.
- How individual understanding is demonstrated: in the exit check each student explains their playbook and states one advantage and one limitation of Ansible compared with the module 4–10 scripts; in team settings each member explains their own tasks.
- Permitted AI support and required disclosure: AI may be used for playbook drafting and debugging; if any of this work feeds the final project, the use must be logged in `AI-USE.md` (date, tool + version, task/prompt summary, what was accepted/changed and why). Undocumented AI use is a finding.

## Dependencies and course fit

- Requires: Module 2 (GNS3 API v3 topology), Module 4 (SSH enablement), Modules 8–9 concepts (routing/firewall to replicate); assumes Modules 1–11 are already scheduled — this module is included only if time permits after all required outcomes.
- Enables: nothing required — optional enrichment only; a stronger final project if students choose to add Ansible artifacts under `projects/<name>/`.
- Overlap with other candidate modules: deliberate overlap with modules 4–10 is the point (comparison); it must not replace or compress them.
- Connection to the final project: optional — Ansible artifacts may supplement the PR but the 11 requirements (R1–R11) are satisfied by the modules 2–10 workflow; NETCONF (R4) remains ncclient/pyATS-based since Ansible here does not use NETCONF.
- Scheduled course hours after inclusion: 36 h total (modules 1–12 × 3 h) with Module 1 included; 33 h if Module 1 is waived; without Module 12 the required sequence is 33 h (or 30 h) — every variant stays within the 21–39 h (20–40 h) limit.

## Risks and open questions

- [ ] Owner decision needed: whether Module 12 runs at all — only if all required outcomes (modules 1–11) fit the schedule; owner decision 2026-11-21 marks it optional, and this brief's recommendation is "optional".
- [ ] Technical verification needed: `cisco.ftd` module compatibility with the FTDv 10.0 FDM API (`/api/fdm/latest/`, token endpoint `POST /api/fdm/v1/fdm/token`) on-device before the lab; fallback = `ansible.builtin.uri`.
- [ ] Technical verification needed: `cisco.asa` is in community maintenance with no guaranteed ansible-core > 2.17 compatibility — pin ansible-core and collection versions and test at lab build.
- [ ] Lab validation needed: server RAM/CPU/disk sizing for the 5-node topology (FTDv ~8 GB RAM, 4 vCPUs); WireGuard provisioning + GNS3 account procedure doc still pending; exact IOSv 15.x build TBD; CSR1000v unlicensed throughput cap noted (irrelevant for config pushes).
- [ ] Risk: Ansible must never displace required modules — schedule it last and drop it first if time runs short.
- [ ] Risk: secrets hygiene — vault protects data at rest only; combine with env vars and never commit vault passwords.

## Researcher recommendation

- Recommendation: optional
- Rationale: Ansible is valuable enrichment for students who already completed PE1/PE2 and the modules 2–11 workflow, and the official `cisco.iosxe` / `cisco.asa` / `cisco.ftd` collections map cleanly onto the confirmed images; but the course owner decided it is time-permitting only, it is not part of the final-project rubric, and two of the three collections carry version-compatibility caveats that must be pinned and tested. Include it only after all required outcomes are scheduled.
- Next implementation steps: if the owner approves running it, pin ansible-core + collection versions in a `requirements.yml`, validate `cisco.ftd` against the FTDv 10.0 FDM API on the server, and write the playbook lab guide under `curriculum/modules/module-12-ansible-optional/`.
