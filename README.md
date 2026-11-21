# Cisco Network Automation Course

This repository is the planning workspace for a new, hands-on course on
automating Cisco network equipment in GNS3. It is currently in the course
definition and research phase; the module sequence and implementation materials
have not yet been approved.

## Course constraints

- Students have completed Cisco Networking Academy Python Essentials 1 and 2.
  Avoid repeating a general Python course; include targeted refreshers only
  where they support a network-automation task.
- Each module is a three-hour session.
- The total course should run for 20–40 hours. With three-hour sessions, this
  means 7–13 modules (21–39 scheduled hours).
- The target is 30 hours (10 modules).
- Mandatory automation areas include SSH, REST where supported, NETCONF, basic
  YANG, and pyATS. Concurrency is mandatory; Ansible is optional if time allows.
- Candidate device images are IOSv, CSR1000v, ASA, and FTD, subject to remote
  GNS3 availability and per-image protocol support.
- Students use a remote GNS3 server and create the project topology through
  its API; local virtualization is not required.
- Students are encouraged to use AI in their work.
- Final projects are submitted as PRs with student deliverables under
  `projects/`; teams are allowed if announced early, and each team member must
  contribute commits. Projects are reviewed against a 50% requirements
  threshold, merged, and presented to the 3PG team.
- Students document AI use to help assess their ability to work with AI; the
  format and assessment criteria remain to be defined.

## Repository map

- `AGENTS.md` — project-wide instructions for AI and human contributors.
- `.github/copilot-instructions.md` — instructions for Copilot in this
  repository.
- `docs/course-outline.md` — decisions and open questions for the course plan.
- `docs/research/legacy-review.md` — findings from the two previous attempts.
- `docs/authoring/module-research-playbook.md` — research workflow for defining
  each module.
- `docs/templates/module-brief.md` — repeatable module-research template.
- `projects/` — student final projects, submitted through pull requests.
- `curriculum/modules/` — reserved for approved module briefs and, later,
  teaching materials.

## Status

The previous projects have been reviewed and the course outcomes and project
constraints have been captured. The next step is to research and approve
module briefs before producing lessons, labs, or solutions. See
[`docs/course-outline.md`](docs/course-outline.md) and
[`docs/authoring/module-research-playbook.md`](docs/authoring/module-research-playbook.md).
