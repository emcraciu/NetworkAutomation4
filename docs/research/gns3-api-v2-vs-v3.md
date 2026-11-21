# GNS3 server API v2 (2.2.x) vs v3 (3.0.x) — what actually differs

> Research note, accessed 2026-11-21. Question from the course owner: how big is
> the v2→v3 difference, and should the course maintain two versions (both could be
> installed; with a VM one could switch)?

## Short answer

For the ~10 API calls this course needs (version, authenticate, list templates,
create project, create nodes, create links, start/stop, read console port,
close/delete project), the difference is **small**:

| Aspect | v2 (2.2.x, maintenance line) | v3 (3.0.x, active line) |
|---|---|---|
| Base path | `/v2/...` | `/v3/...` |
| Port | 3080 | 3080 (same) |
| Auth | HTTP Basic header on every request (no token endpoint) | `POST /v3/access/users/authenticate` → access token used as Bearer; `/v3/access/users/login` → JWT pair + `/refresh` |
| Access control | none (or basic auth only) | full RBAC: `/v3/access/users`, `/v3/access/groups`, `/v3/access/roles` |
| Projects/nodes/links/templates | `/v2/projects`, `/v2/projects/{id}/nodes`, `/links`, `/v2/templates` | same shapes under `/v3/...` (field-level renames possible — validate on the installed server) |
| Compute endpoints | `/v2/compute/...` proxy endpoints | reorganized; controller proxies to computes |
| New in v3 | — | `/v3/settings`, `/v3/statistics`, `/v3/iou_license`, `/v3/reload`; websocket notifications authenticate with JWT in the query string |
| Client libraries | `gns3fy` targets v2 — works out of the box | gns3fy v3 support unverified → teach plain `requests` |
| Server OS | Linux native; GNS3 VM for Win/macOS | **Linux only**; Win/macOS via remote server or web UI |
| Project files | — | converted to the new format on first open in 3.x, **no rollback** (explicit warning in the 3.0 release notes) |
| License | GPLv3, free | GPLv3, free |

Sources (accessed 2026-11-21): api.gns3.net (v2 endpoints + general docs),
apiv3.gns3.net (v3 OpenAPI/Swagger), gns3-server.readthedocs.io endpoint lists,
GNS3/gns3-server GitHub releases + LICENSE, gns3.com community blogs "What's new in
GNS3 version 3.0" and "GNS3 3.0 Released!", gns3fy docs/PyPI, RBAC discussion
GNS3/gns3-server#1949.

## Can both run "on the server" at once?

Not on the same host: both versions ship the same `gns3server` binary, systemd
service, port 3080, and config directories, and both are the same PyPI/apt package
name (`gns3-server`), so they overwrite each other. You can run one version at a
time, or two separate VMs/containers. And because 3.x **irreversibly converts
project files** on first open, switching a VM back and forth mid-course corrupts
the "switch back" path for existing projects. So "switching" is an operational
risk, not a feature.

## Recommendation

**Standardize the course on one version — 3.0.x (API v3).** Reasons:

1. 2.2 is in maintenance mode (2.2.61 was a security/bugfix release); 3.x is the
   active line. No licensing difference (both GPLv3).
2. The calls the course uses are ~95 % identical; the real diff is the `/v3`
   prefix + token auth. That is a **~30-line compatibility shim** in the Module 2
   starter code (`gns3_client.py` with an `API_VERSION` switch), not two course
   versions.
3. Maintaining two full material tracks would double owner review and lab
   validation for zero pedagogical gain, and the irreversible project conversion
   makes version switching mid-course unsafe anyway.

The shim gives the insurance the owner wants: if the server is ever switched
between 2.2.x and 3.0.x, the same student scripts keep working by changing one
constant, instead of maintaining two sets of briefs/labs.

## Open validation items (on the installed server)

- Exact v3 payload/response field names for project/node/link create + start
  (v2→v3 renames possible).
- Whether any gns3fy release works against a v3 controller (default: `requests`).
- RBAC role needed for a student account on the 3.0.x controller.
