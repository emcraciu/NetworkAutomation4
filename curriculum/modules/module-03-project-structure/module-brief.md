# Module 3: Automation project structure and targeted Python/data refreshers

> Status: ready for review
> Scheduled duration: 3 hours (180 minutes)
> Owner approval: pending — course-owner PR review (owner decisions of 2026-11-21 are settled)

## Purpose and course alignment

- Course outcome supported: final-project requirements R10 (project README, no secrets, runnable evidence) and R11 (AI-USE.md log), plus the PR rule that deliverables live only under `projects/<name>/`.
- Why students need this module: Modules 4–10 each produce scripts; without a fixed project layout, inventory format, and secrets discipline from the start, the final PR will fail the secret scan and be unreviewable. PE1/PE2 do not cover packaging, virtualenvs, env-based secrets, or YAML inventories in an automation context.
- What this module will not cover: a full Python course or re-teach of PE1/PE2 topics; only task-relevant refreshers are included, each tied to a concrete automation task in this session.
- Core / optional / enrichment: core.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: variables, control flow, functions, modules, basic OOP, exceptions, file I/O, dicts/lists, pip — all PE1/PE2 material, not re-taught.
- Networking prerequisites: the reference topology concept from Module 2 (2× CSR1000v + FTDv 10.0 + Ubuntu endpoint); no device access needed in this module.
- GNS3 and prior-module prerequisites: Modules 1–2 completed (or Module 1 waived by the owner's placement assessment); Python 3.11+ installed locally; git and a GitHub account for the PR workflow.
- Just-in-time refresher needed (if any), and why: refreshers are delivered only where the session's tasks need them — dataclasses/classes and custom exceptions for the inventory model, file I/O + JSON/YAML handling for the inventory file, and env-var loading for secrets — each justified by the lab task, per the course rule against reserving sessions for introductory Python.

## Learning outcomes

By the end of the session, students can:

1. Create a Python 3.11+ virtual environment and a pinned `requirements.txt` that installs cleanly on a fresh machine.
2. Lay out an automation project (README, `scripts/` + `lib/` or `src/`, inventory, config, `.gitignore`, `AI-USE.md`) matching the `projects/<name>/` PR structure of this repo.
3. Keep all credentials and secrets out of git: load them from environment variables via `python-dotenv` with a committed `.env.example` of synthetic placeholders, and pass a secret scan (gitleaks/trufflehog or manual grep).
4. Use dataclasses, custom exceptions, file I/O, and dict/JSON/YAML handling to load a YAML inventory of the reference topology and emit a structured, secret-masked summary with `logging`.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| Virtual environments | stdlib `venv` creates isolated envs; `python -m venv .venv` + `pip install -r requirements.txt` is the reproducible baseline | Python docs, "venv — Creation of virtual environments" — https://docs.python.org/3/library/venv.html | Python 3.11+ docs; accessed 2026-11-21 | High |
| requirements.txt format | pip requirements-file syntax (pins, comments) | pip docs, "Requirements File Format" — https://pip.pypa.io/en/stable/reference/requirements-file-format/ | accessed 2026-11-21 | High |
| Secrets via environment variables | 12-factor style: config in env vars; `python-dotenv` loads `.env` locally without committing it | python-dotenv PyPI — https://pypi.org/project/python-dotenv/ ; 12factor.net — https://12factor.net/config | python-dotenv 1.2.x seen; accessed 2026-11-21 | High for the tool; 12factor is a secondary source (medium) |
| Secret scanning in review | final-project review runs a secret scan first (gitleaks/trufflehog or manual grep); a PR with secrets is rejected | repo: `docs/final-project-requirements.md` (R10); gitleaks — https://github.com/gitleaks/gitleaks ; trufflehog — https://github.com/trufflesecurity/trufflehog | accessed 2026-11-21 | High (course rule); tool choice is ours |
| YAML inventory | PyYAML is the standard YAML parser/emitter for Python (YAML 1.1); wheels available for 3.11+ | PyPI, "PyYAML" — https://pypi.org/project/PyYAML/ | 6.0.2 seen; accessed 2026-11-21 | High; JSON stdlib alternative if YAML is a blocker |
| JSON handling | stdlib `json` for JSON inventories/config | Python docs, "json — JSON encoder and decoder" — https://docs.python.org/3/library/json.html | accessed 2026-11-21 | High |
| dataclasses / classes | `dataclasses` (3.7+) gives concise data-holding classes for inventory entries | Python docs, "dataclasses — Data Classes" — https://docs.python.org/3/library/dataclasses.html | accessed 2026-11-21 | High; PE2 already covers basic classes — refresher is task-tied only |
| exceptions | try/except/else/finally, raising and custom exceptions | Python docs, "Errors and Exceptions" — https://docs.python.org/3/tutorial/errors.html | accessed 2026-11-21 | High; PE1 covers basics — refresher only where the loader needs it |
| logging | stdlib `logging` with named loggers, levels, and handlers; never log secrets | Python docs, "logging — Logging facility for Python" — https://docs.python.org/3/library/logging.html ; HOWTO — https://docs.python.org/3/howto/logging.html | accessed 2026-11-21 | High |
| PE1/PE2 baseline (no re-teach) | course rule: do not reserve sessions for introductory Python; refresh only task-relevant features | Cisco NetAcad + Python Institute — https://pythoninstitute.org/netacad ; repo `AGENTS.md` | accessed 2026-11-21 | High (owner-confirmed 2026-11-21) |
| Final-project PR layout | deliverables only under `projects/<name>/` with README; AI-USE.md table format | repo: `projects/README.md`, `docs/final-project-requirements.md` | accessed 2026-11-21 | High (course rule, pending owner approval of the rubric doc) |
| CLI entry points | stdlib `argparse` for the build/cleanup/run commands the scripts will grow in Modules 4–10 | Python docs, "argparse — Parser for command-line options" — https://docs.python.org/3/library/argparse.html | accessed 2026-11-21 | High |
| Path handling | `pathlib` for robust, OS-independent paths in scripts and the scaffold | Python docs, "pathlib — Object-oriented filesystem paths" — https://docs.python.org/3/library/pathlib.html | accessed 2026-11-21 | High |
| GitHub secret scanning | GitHub can detect/block known secret formats in PRs; still run gitleaks/trufflehog locally before pushing — belt and suspenders for the course rule "never commit credentials" | GitHub docs, "About secret scanning" — https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning | accessed 2026-11-21 | Medium-high; depends on repo settings, so local scan stays mandatory |

> All URLs in this table were accessed on 2026-11-21.

### Platform and version constraints

- Cisco device types/images: no live devices needed; the inventory describes the owner-confirmed reference topology — 2× CSR1000v IOS XE 17.3.8a, FTDv 10.0.0, Ubuntu Docker endpoint, optional IOSv 15.x (build TBD) or ASAv 9.24; ≤5 nodes.
- GNS3 version/API requirements: none in this session; inventory fields anticipate Module 2's API objects (name, template, console port, management IP).
- Python version and dependencies: Python 3.11+; stdlib `venv`, `json`, `logging`, `dataclasses`, `argparse`; pip: `python-dotenv`, `PyYAML` (optional `requests` carried over from Module 2); all pinned in `requirements.txt`.
- Licensing, resource, and OS assumptions: pure laptop work, any OS with Python 3.11+ and git; no server resources consumed; works fully offline.
- Protocol/API interpretation (especially if “RPC” is involved): not applicable — data formats and project hygiene only; NETCONF/REST/YANG start in Modules 5–6.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–15 | Check-in; why structure matters: the final-project PR rules (deliverables only under `projects/<name>/`, secret scan first, AI-USE.md) | Student states one PR rule in their own words |
| 15–40 | venv + pinned `requirements.txt`; package layout (`scripts/` + `lib/` or `src/`); README skeleton; `logging` setup | Fresh venv created; first pinned install succeeds |
| 40–50 | Break | — |
| 50–75 | Inventory in YAML/JSON; secrets only via environment variables (`python-dotenv`), `.env.example` committed, `.env` gitignored; secret-scan demo | `.env.example` with synthetic placeholders; scan output clean |
| 75–85 | Just-in-time refresher 1: classes/dataclasses + custom exceptions, tied to modeling an inventory device entry | Small dataclass for a device entry |
| 85–95 | Just-in-time refresher 2: file I/O + dict/JSON/YAML handling, tied to loading the inventory | Loader reads YAML and prints a dict summary |
| 95–150 | Guided lab: scaffold the final-project repo structure with `inventory.yaml` for the reference topology, `.env.example`, config loader, logging, `AI-USE.md` template, README | Complete scaffold passing a fresh-venv install and the loader run |
| 150–165 | Verification: fresh `python -m venv` + `pip install -r requirements.txt`; run loader; run gitleaks/trufflehog (or grep) scan | Reproducible install; masked log output; clean scan |
| 165–180 | Wrap-up and exit check | Exit check: student adds one device to the inventory and explains the secrets rule |

**Planned total:** 15 + 25 + 10 + 25 + 10 + 10 + 55 + 15 + 15 = 180 minutes, including break and wrap-up

## Lab specification

- GNS3 topology and API steps: none required — this module works on data describing the reference topology built in Module 2 (2× CSR1000v 17.3.8a, FTDv 10.0.0, Ubuntu Docker endpoint, optional IOSv/ASAv; ≤5 nodes). No nodes are started.
- Devices, images, interfaces, and services: inventory entries only (device name, role, image/version, management IP placeholder, console port placeholder, protocol capabilities per the owner-confirmed matrix: NETCONF+RESTCONF on CSR1000v only; FDM REST on FTDv; ASA REST on ASAv; SSH/console on IOSv).
- Starter materials: empty template with README skeleton, `.gitignore` (includes `.env`), `requirements.txt` skeleton, `scripts/` + `lib/` layout, `AI-USE.md` table header.
- Student deliverables: scaffolded project directory containing `inventory.yaml` (reference topology), `.env.example` (synthetic placeholders only), a config/loader module reading env vars + inventory with `python-dotenv`, a `logging`-based runner that prints a secret-masked summary, README with setup/run/cleanup, and `AI-USE.md`; an optional `run.sh`/Makefile for the common commands is allowed but not required.
- Expected results and verification: on a fresh machine, `python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` succeeds; running the loader prints all inventory devices with credentials masked; `gitleaks detect` (or manual grep for password/token patterns) reports nothing. Comparing `pip freeze` output against `requirements.txt` demonstrates why pinning matters.
- Reset/cleanup: delete the scaffold and regenerate from the template; `pip freeze` differences show why pinning matters.
- Common failures and recovery: dependency version conflicts → pin exact versions and re-install in a fresh venv; `.env` accidentally staged → add to `.gitignore`, add a pre-commit hook, `git rm --cached .env`; YAML indentation errors → validate with a YAML linter and a try/except around the load with a clear error message; secrets printed in logs → mask before logging; `pip` installing into the system Python → always activate the venv first (check `which python`).
- Offline or low-resource alternative: fully offline — no VPN, server, or devices needed; if PyYAML cannot be installed, fall back to a JSON inventory with stdlib `json` (same code path via a format switch).

## Assessment and AI use

- Formative checks: fresh-venv install during the session; instructor observation of the loader run and the secret scan; refresher exercises checked as they are introduced.
- Assessed evidence: the scaffolded project (inventory, `.env.example`, loader, README, `AI-USE.md`) with the fresh-install and clean-scan outputs pasted into the session notes.
- How individual understanding is demonstrated: exit check where each student adds a device to the inventory, explains why `.env` must never be committed, and shows where secrets are masked in logs; the final PR review and 3PG presentation later check individual understanding again (team PRs need commits from every member). The AI-USE.md log is reviewed together with the PR — an empty or missing log is itself a finding.
- Permitted AI support and required disclosure: AI may be used for research, scaffolding, coding, debugging, and reflection; every use logged in `AI-USE.md` (date, tool + version, task/prompt summary, what was accepted/changed and why); undocumented AI use is a finding. Never paste credentials, tokens, or real-network data into AI prompts; synthetic credentials only, isolated authorized GNS3 lab only.

## Dependencies and course fit

- Requires: Modules 1–2 (or Module 1 waived by owner placement); Python 3.11+, git, pip locally.
- Enables: Modules 4–11 — every later module's scripts live in this structure; final-project R10 (README, no secrets) and R11 (AI-USE.md) are direct products of this module.
- Overlap with other candidate modules: deliberately none — no introductory-Python session; refreshers are task-tied (dataclasses/exceptions for the inventory model, file I/O + JSON/YAML for the inventory file) per the course rule.
- Connection to the final project: the scaffold built here becomes the student's `projects/<name>/` PR skeleton; the inventory format is reused by Modules 4–10 scripts and the pyATS testbed in Module 7.
- Scheduled course hours after inclusion: 3 h for this module; running total with Modules 1–3 = 9 h; course total 30/33/36 h depending on Module 1 waiver and optional Module 12 — within 21–39 h.
- Out-of-class work assumed: students push their scaffold to their own git remote as practice for the final PR; no other homework.

## Risks and open questions

- [ ] Owner decision needed: approval of `docs/final-project-requirements.md` (rubric, 50% threshold, presentation) that this module's scaffold operationalizes; confirm whether a pre-commit secret-scan hook is mandatory for student repos.
- [ ] Technical verification needed: pinned versions of `python-dotenv`/`PyYAML`/`requests` compatible with Python 3.11+ on all student OSes (verify in fresh venvs); gitleaks vs trufflehog choice for the course-wide scan.
- [ ] Lab validation needed: none against the server (offline module), but the wider open items still apply to later modules and are tracked there: server RAM/CPU sizing for FTDv ~8 GB, WireGuard procedure doc pending, exact IOSv build TBD, on-device checks of CSR1000v NETCONF `<hello>` and FTD `/api/fdm/latest/` + token endpoint, CSR1000v unlicensed throughput cap (~2.5 Mbps, fine for lab traffic).
- [ ] Delivery risk: students with no git/PR experience need extra attention in the verification step — pair them during the session; the final-project PR rules (deliverables only under `projects/<name>/`) are enforced at review, not assumed.

## Researcher recommendation

- Recommendation: include.
- Rationale: the project skeleton, inventory, and secrets discipline are prerequisites for every later module and for a reviewable final PR (R10/R11); the session avoids Python re-teaching by tying each refresher to a concrete task, and it needs no server resources, so it is low-risk in the schedule.
- Next implementation steps: after owner approval via PR review, publish the scaffold template, the reference-topology `inventory.yaml` example (synthetic data), the `.env.example`, and the fresh-install + secret-scan checklist used for formative checks.
