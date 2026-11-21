# Contributor and agent instructions

## Purpose and current phase

This repository is for designing a three-hour-per-module course on automating
Cisco network equipment virtualized in GNS3. The students have completed
Python Essentials 1 and 2. The current phase is course discovery and module
research, not lesson or lab implementation.

## Non-negotiable constraints

- Keep the total course between 20 and 40 hours. Since each meeting is three
  hours, plan for 7–13 modules, or 21–39 scheduled hours.
- Do not assume the previous courses' introductory Python material is needed.
  Propose only task-relevant refreshers, and justify them with prerequisites.
- The target is 30 hours / 10 modules. The agreed final workflow creates a
  remote GNS3 topology through its API, bootstraps devices through console,
  configures routing and firewall policy using automation, and verifies
  permitted and denied ping/DNS traffic by running pyATS tests from an Ubuntu
  endpoint.
- SSH, REST where supported, NETCONF, basic YANG, and pyATS are required.
  Candidate images are IOSv, CSR1000v, ASA, and FTD; verify actual image and
  protocol support before designing device-specific labs.
- Concurrency is mandatory. Prefer to rely on Python Essentials coverage, but
  include a focused introduction in Module 1 if that prerequisite is missing.
  Ansible is optional if time remains.
- Students use a remote GNS3 server; local network virtualization is not a
  prerequisite. They may use the GNS3 GUI to visualize progress.
- Encourage responsible AI use in research, coding, debugging, and reflection.
  Students may use AI throughout their projects and must document its use;
  the evidence format is still to be defined.
- Student project deliverables belong under `projects/` and are submitted by
  pull request. Individual and team submissions are allowed; teams must be
  announced early and each team member must contribute commits. A reviewed
  project meeting at least 50% of requirements is merged and considered
  accepted, with a presentation to the 3PG team as the final assessment.
- Never commit credentials or secrets.

## Workflow

1. Read `README.md`, `docs/course-outline.md`,
   `docs/research/legacy-review.md`, and
   `docs/authoring/module-research-playbook.md` before proposing curriculum.
2. Follow confirmed decisions in `docs/course-outline.md`. Keep unresolved
   technical constraints (especially image versions and API support) explicit
   until researched and validated.
3. Use the module brief template for each candidate module. Research only
   topics that are relevant to the agreed outcomes and preserve source links,
   access dates, version/platform constraints, and unresolved risks.
4. Fit teaching, demonstration, guided lab, independent practice, and wrap-up
   into the three-hour session. State any assumed out-of-class work separately.
5. Trace module dependencies and verify that the total scheduled time is
   within the course limit before recommending an outline.
6. Do not copy code, student work, credentials, or device configuration from
   previous repositories. Reuse prior projects only as evidence of topic
   coverage and potential pitfalls.
7. Keep decisions, assumptions, and open questions distinct. Never invent
   results, tested compatibility, or source citations.
8. Do not create device-specific lab instructions until the relevant module
   brief and remote environment constraints are researched and approved.

## Changes and validation

- Prefer small, reviewable documentation changes that directly support the
  approved planning work.
- Update the relevant documentation when a course decision changes.
- Keep examples safe for isolated, authorized GNS3 labs; do not target real
  networks or include secrets.
- For documentation-only changes, check links and consistency with the
  three-hour and total-course constraints. Run other checks only when relevant.
