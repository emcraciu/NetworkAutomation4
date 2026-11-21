# Module 6: NETCONF and YANG

> Status: ready for review  
> Scheduled duration: 3 hours (180 minutes)  
> Owner approval: pending — course owner PR review (owner decisions of 2026-11-21)

## Purpose and course alignment

- Course outcome supported: final-project requirement R4 (NETCONF against a
  NETCONF-capable device — CSR1000v 17.3.8a — for a config or operational task, plus a
  short YANG-basics explanation in the README); the outline makes NETCONF mandatory.
- Why students need this module: NETCONF is the only RPC-style, model-driven management
  protocol in the confirmed image set (only the CSR1000v supports it), so students must
  read YANG models and drive `<hello>`, `get-config`, `edit-config`, `get` with
  ncclient; Module 5 showed the same datastores over HTTP.
- What this module will not cover: RESTCONF details (Module 5), gNMI/gRPC, writing YANG
  models (students read them), pyATS (Module 7).
- Core / optional / enrichment: core.

## Audience and prerequisites

- Assumed Python Essentials 1/2 knowledge: XML basics (stdlib `xml.etree.ElementTree`
  is enough), dicts, exceptions, env-var config from Module 3.
- Networking prerequisites: loopback interfaces and descriptions at CCNA level; the
  concept of configuration vs operational (state) datastores.
- GNS3 and prior-module prerequisites: Module 2 (API v3 topology), Module 3 (project
  structure, env-var secrets), Module 4 (bootstrapped CSR1000v with `netconf-yang` +
  working SSH credentials); Module 5 is a useful contrast, not a hard prerequisite.
- Just-in-time refresher needed (if any), and why: a short XML/ElementTree recap —
  PE1/PE2 do not cover XML and NETCONF payloads are XML.

## Learning outcomes

By the end of the session, students can:

1. Explain the NETCONF session model (RFC 6241: SSH on TCP 830, `<hello>`, datastores such as `running`, operations `get-config`, `edit-config`, `get`) and how RESTCONF (RFC 8040) maps onto the same YANG datastores over HTTP.
2. Read a YANG model's structure — module, container, leaf, leaf-list, tree — using
   the public Cisco models (YangModels/yang `vendor/cisco/xe`, incl.
   `Cisco-IOS-XE-native`).
3. Use ncclient against a CSR1000v 17.3.8a (`manager.connect(host, port=830,
   username=..., password=..., hostkey_verify=False, allow_agent=False)`) to print
   `server_capabilities`, run filtered `get-config` on running, and `get` for state.
4. Push a config change with `edit-config` (loopback + description), read it back,
   verify it on the device CLI over SSH, and recover cleanly from a rejected edit.

## Research findings

| Topic or claim | Finding | Source (title and URL) | Version/access date | Confidence or open issue |
|---|---|---|---|---|
| NETCONF (RFC 6241): SSH subsystem transport, default TCP 830, `<hello>` capability exchange, datastores, `<get-config>`/`<edit-config>`/`<get>` | Protocol model for this module | RFC 6241 — https://www.rfc-editor.org/rfc/rfc6241.html | Standards Track, June 2011; accessed 2026-11-21 | High |
| YANG (RFC 6020; current revision RFC 7950 "YANG 1.1"): modules, containers, leaves, leaf-lists, trees, config vs state; RESTCONF (RFC 8040) is the HTTP binding of the same NETCONF datastores | YANG basics + relation to Module 5 | RFC 6020 — https://www.rfc-editor.org/rfc/rfc6020.html ; RFC 7950 — https://www.rfc-editor.org/rfc/rfc7950.html ; RFC 8040 — https://www.rfc-editor.org/rfc/rfc8040.html | accessed 2026-11-21 | High |
| IOS XE 17.3 NETCONF: `netconf-yang`, SSH port 830 default (configurable), capability discovery via ietf-netconf-monitoring | Enablement + port for the lab | "Programmability Configuration Guide, Cisco IOS XE Amsterdam 17.3.x — NETCONF Protocol" — https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg/configuring_yang_datamodel.html | accessed 2026-11-21 | High; on-device `<hello>` check via ncclient pending |
| Public Cisco YANG models: YangModels/yang, `vendor/cisco/xe/...` (incl. `Cisco-IOS-XE-native`) | Where students read real models | GitHub YangModels/yang — https://github.com/YangModels/yang/tree/main/vendor/cisco | accessed 2026-11-21 | High |
| ncclient API: `manager.connect(host, port=830, username=..., password=..., hostkey_verify=False, allow_agent=False)`, then `server_capabilities`, `get_config(source='running', filter=...)`, `edit_config(target='running', config=...)`, `get(filter=...)` | Client library for the lab | ncclient PyPI — https://pypi.org/project/ncclient/ ; docs — https://ncclient.readthedocs.io/en/latest/ | accessed 2026-11-21 | High; confirm kwargs against the pinned installed version |
| NETCONF only on CSR1000v among confirmed images (ASAv/FTDv REST-only, IOSv neither) | Device selection: CSR1000v only | DevNet "Standard Device Interfaces & Models" — https://developer.cisco.com/site/standard-network-devices/ ; repo note `docs/research/device-protocol-support.md` (owner-confirmed 2026-11-21) | accessed 2026-11-21 | High (owner-confirmed) |
| IOS XE candidate datastore / confirmed-commit exists per the 17.3 guide; `hostkey_verify=False` acceptable only in the isolated lab | Rollback story + lab security posture | Same 17.3 guide NETCONF chapter (URL above); ncclient/paramiko docs — https://ncclient.readthedocs.io/en/latest/ , https://pypi.org/project/paramiko/ | accessed 2026-11-21 | Medium — verify candidate behavior on device; teach `edit-config` to `running` by default; never commit keys |
### Platform and version constraints

- Cisco device types/images: CSR1000v IOS XE Amsterdam 17.3.8a only — the sole
  NETCONF-capable image (`netconf-yang`, TCP 830); ASAv/FTDv/IOSv are excluded.
- GNS3 version/API requirements: GNS3 server 3.0.x, REST API v3; one or two CSR1000v
  nodes created via the API (≤5-node limit); Module 4 bootstrap already enables
  `netconf-yang`.
- Python version and dependencies: Python 3.11+; `ncclient` (pulls in `paramiko`,
  `lxml`), stdlib `xml.etree.ElementTree` for pretty-printing; `netmiko` from Module 4
  for CLI verification; credentials via env vars only.
- Licensing, resource, and OS assumptions: `netconf-yang` is an IOS XE software feature
  (no separate license expected — verify on device); CSR1000v's unlicensed throughput
  cap is irrelevant for NETCONF control traffic; any OS over WireGuard VPN.
- Protocol/API interpretation (especially if “RPC” is involved): in this course “RPC”
  means NETCONF — XML-encoded RPCs over SSH on TCP 830 (RFC 6241), not gNMI/gRPC or
  XML-RPC; RESTCONF (Module 5, RFC 8040) is the HTTP binding of the same YANG datastores.

## Three-hour session plan

| Time | Activity | Student task / evidence of learning |
|---|---|---|
| 0–10 (10 min) | Check-in + recap of Modules 4–5 (SSH + RESTCONF working); what “RPC” means here | Student states the NETCONF port and which image supports it |
| 10–35 (25 min) | Standards: RFC 6241 session model (SSH 830, `<hello>`, datastores, operations); RFC 6020/7950 YANG basics (modules, containers, leaves, trees) | Student reads a small YANG tree and names config vs state nodes |
| 35–55 (20 min) | Cisco models in YangModels/yang `vendor/cisco/xe`; ncclient intro: connect kwargs, capabilities, get-config/edit-config/get | Student finds the interface module in the repo and matches it to CLI config |
| 55–70 (15 min) | Break | — |
| 70–80 (10 min) | Instructor demo: ncclient connect to CSR1000v, print `server_capabilities`, `get-config` running | Students see `<hello>` capabilities and a filtered config reply |
| 80–130 (50 min) | Guided lab: `get-config` running (filtered to interfaces), `edit-config` loopback + description, read-back, `get` operational state, CLI verification over SSH | Working script: capabilities printed; loopback in read-back and `show running-config` |
| 130–155 (25 min) | YANG-tree practice: map sent/received XML to the model tree; `get` vs `get-config` (state vs config) | Student annotates one `Cisco-IOS-XE-native` subtree with the matching XML element |
| 155–170 (15 min) | Independent practice + failure drill: rejected edit, wrong filter, port closed (`netconf-yang` missing) | Student recovers each case; explains why running config was not corrupted |
| 170–180 (10 min) | Wrap-up and exit check | Exit check: explain `<hello>` and one operation; bridge to Module 7 (pyATS) |

**Planned total:** 180 minutes (includes the 15-minute break and the wrap-up)

## Lab specification

- GNS3 topology and API steps: create the project via API v3, add one CSR1000v (two
  for parallel reads if Module 1 is in place), bootstrap via the Module 4 console
  script, read management IP + status from `GET /v3/projects/{project_id}/nodes`.
- Devices, images, interfaces, and services: CSR1000v IOS XE 17.3.8a; SSH (TCP 22, CLI
  verification) and NETCONF (TCP 830, `netconf-yang`); synthetic credentials via env
  vars only.
- Starter materials: `netconf_client.py` skeleton (`manager.connect` with the lab
  kwargs, capability print, filtered `get_config`, `edit_config`, `get`, ElementTree
  pretty-print), `.env.example` (names only), a YANG tree excerpt from YangModels/yang.
- Student deliverables: working `netconf_client.py` that (a) prints server
  capabilities, (b) `get-config` running filtered to interfaces, (c) pushes a loopback
  + description via `edit-config`, (d) reads it back, (e) `get` operational state;
  CLI verification via netmiko (`show running-config | include Loopback`); a short
  README with the YANG-basics explanation required by R4; AI-USE.md entries.
- Expected results and verification: `server_capabilities` lists `ietf-netconf` (and
  Cisco capabilities); `edit-config` returns `<ok/>`; read-back contains the loopback
  and description; `get` returns operational interface state; CLI shows the loopback
  up/up.
- Reset/cleanup: remove the loopback via `edit-config` (delete) or delete the whole
  project via `DELETE /v3/projects/{project_id}`.
- Common failures and recovery: connection refused on 830 (check `netconf-yang` —
  re-run Module 4 bootstrap); auth failure (env-var creds); host-key mismatch
  (lab-only `hostkey_verify=False`); XML validation error on `edit-config` (fix
  payload; `running` rejects invalid config atomically — verify candidate/commit on
  device first); bad filters (keep subtree filters small); oversized unfiltered replies.
- Offline or low-resource alternative: pair on a single CSR1000v node; the YANG-tree
  and XML-mapping exercises work fully offline against the YangModels/yang repo; a
  recorded ncclient transcript can dry-run parsing/filter logic if the server is down.

## Assessment and AI use

- Formative checks: instructor reviews each student's capability printout and filter
  during the guided lab; exit check at 170–180.
- Assessed evidence: working `netconf_client.py` with captured `<ok/>` and read-back,
  CLI verification, the README's YANG-basics explanation, and the exit check; feeds
  final-project R4 directly.
- How individual understanding is demonstrated: each student maps one YANG tree to the
  XML their script sent and explains, unprompted, what `<hello>` negotiates and why
  `get` and `get-config` differ.
- Permitted AI support and required disclosure: AI may be used for drafting, debugging
  (e.g. XML namespace errors), and explaining RFC/YANG concepts, but every use must be
  logged in `AI-USE.md` (date, tool + version, task/prompt summary, what was
  accepted/changed and why); undocumented AI use is a finding. Never paste credentials
  or configs into AI tools.

## Dependencies and course fit

- Requires: Module 2 (API v3 topology, VPN), Module 3 (project structure, env-var
  secrets), Module 4 (bootstrapped CSR1000v with `netconf-yang` + SSH credentials);
  Module 5 useful as a contrast, not a hard prerequisite.
- Enables: final-project R4 (NETCONF + YANG basics); supports Module 7 (pyATS testbeds
  can use NETCONF/SSH to the same devices) and Modules 8–10 verification; reinforces
  Module 5's RESTCONF model.
- Overlap with other candidate modules: deliberate, bounded contrast with Module 5 —
  RESTCONF vs NETCONF on the same datastores; this module teaches the protocol and model
  reading, Module 5 taught the HTTP mapping. No other module covers NETCONF.
- Connection to the final project: direct implementation of R4 — ncclient (or pyATS
  NETCONF) doing a config/operational task on the CSR1000v plus the README's
  YANG-basics explanation.
- Scheduled course hours after inclusion: modules 2–11 = 10 × 3 h = 30 h (33 h with
  Module 1, 36 h with optional Module 12) — one 3-hour session inside that envelope,
  total within 21–39 h.

## Risks and open questions

- [ ] Technical verification needed: CSR1000v 17.3.8a `<hello>` via ncclient on the
      server image; candidate/commit behavior on 17.3.8a (teach `edit-config` to `running` until verified).
- [ ] Lab validation needed: pinned ncclient `connect()` kwargs against the Module 3
      venv; server RAM/CPU sizing for the later 5-node topology (FTDv ~8 GB RAM, 4
      vCPUs); exact IOSv 15.x build TBD (no NETCONF on IOSv anyway).
- [ ] Owner decision needed: WireGuard provisioning + GNS3 account procedure document
      still pending (assumed done after Module 2).
- [ ] Risk: CSR1000v unlicensed throughput cap — irrelevant for NETCONF control traffic.
- [ ] Risk: single NETCONF-capable device — if the CSR1000v fails to boot there is no
      fallback; mitigate via the Module 4 status/reboot path and offline YANG/XML work.
- [ ] Risk: `hostkey_verify=False` must stay lab-only and commented; the final
      project's secret scan and review reject committed keys/secrets.

## Researcher recommendation

- Recommendation: include
- Rationale: NETCONF is mandatory and the CSR1000v 17.3.8a is the only confirmed
  NETCONF-capable image, so this module is required and self-contained: one device, one
  client library (ncclient), three RFCs, and the public Cisco YANG models. It maps
  directly to final-project R4 and sets up the pyATS module.
- Next implementation steps: after owner approval via PR review, implement the starter
  `netconf_client.py` and lab README here; run the on-device `<hello>`/capabilities and
  candidate-datastore checks on the remote server and record results in
  `docs/research/device-protocol-support.md`; pin the ncclient version in Module 3.
