# Module research playbook

Use this workflow to collect the evidence needed to decide whether a module
belongs in the course and, if approved, to implement it. Complete one
[`module brief`](../templates/module-brief.md) per candidate module. Research
first; do not jump directly to slides or code.

## 1. Confirm the need

1. Read `README.md`, `AGENTS.md`, `docs/course-outline.md`, and
   `docs/research/legacy-review.md`.
2. Identify the agreed course outcome this module supports.
3. State why the topic is not already adequately covered by Python Essentials
   1/2 or another planned module.
4. Identify prerequisites, especially networking fundamentals, Python skills,
   GNS3 familiarity, and earlier course modules.
5. If the owner has not settled a decision that materially affects the module,
   record the uncertainty rather than silently choosing a platform or tool.

## 2. Investigate authoritative sources

1. Start with current official documentation from Cisco, GNS3, Python, or the
   relevant project/vendor. Use standards documents when protocol behavior
   needs to be established.
2. Verify supported platforms, device images, software versions, licensing,
   API availability, authentication, and known constraints.
3. Determine whether a topic labeled “RPC” means NETCONF, gRPC/gNMI, XML-RPC,
   a Cisco-specific API, or something else. Do not conflate these.
4. Record source title, URL, publisher, version or last-updated information,
   and access date. Distinguish official docs from secondary tutorials.
5. Verify security-sensitive operational assumptions for an isolated
   student-owned lab; use non-production examples and synthetic credentials.
6. Mark unavailable, stale, contradictory, or unverified information as a
   risk. Never claim a lab is compatible without evidence or a test.

## 3. Examine the prior attempts selectively

1. Use `docs/research/legacy-review.md` to identify likely prior examples.
2. Search the relevant module docs and code only to discover coverage,
   exercises, prerequisites, and potential pitfalls.
3. Do not copy student submissions, secrets, device configurations, or code.
4. Note what to retain conceptually, what to modernize, and what to omit for
   the new audience. Re-test any technical claim against current sources.

## 4. Design for a three-hour session

1. Write 2–4 measurable learning outcomes using observable verbs.
2. Plan an achievable sequence: setup/check-in, concise explanation,
   instructor demonstration, guided lab, independent application, verification,
   and debrief.
3. Assign realistic minutes to activities; the planned in-session total must
   equal 180 minutes, including breaks and transition time.
4. Identify what students will actually configure, automate, inspect, or test.
5. Include expected outputs, verification criteria, cleanup/reset steps, and
   recovery paths for common failures.
6. Separate prerequisites and optional homework from the three-hour session.
7. Keep lecture and coding volume realistic for learners with mixed equipment
   and connectivity.

## 5. Specify lab and assessment requirements

For each hands-on activity, collect:

- GNS3 project/topology requirements and whether topology is created through
  the GNS3 API.
- Device types, exact image names/versions, resources, licensing, and access
  requirements.
- Connection details and required services/protocols (for example SSH, HTTP
  APIs, or pyATS connectivity).
- Dependencies and installation approach, with known OS/Python constraints.
- Starter files and student deliverables, without embedding secrets.
- Safe failure modes, resets, validation checks, expected time, and
  instructor support needs.
- Accessibility, lab resource, and offline/fallback requirements.

Design assessment to show individual understanding even when students use AI
or work in teams. Specify what can be AI-assisted, required disclosure, and
how students demonstrate that they understand and tested the result.

## 6. Check course fit

1. Map outcomes to the course outcomes and the final project.
2. Add prerequisites and dependency edges to neighboring modules.
3. Identify overlap; merge or remove redundant topics rather than expanding the
   schedule by default.
4. Count only scheduled class time toward the course total. The whole course
   must contain 7–13 three-hour modules (21–39 hours).
5. Consider whether a topic is core, optional, or better left as an enrichment
   exercise.
6. List unresolved owner decisions and delivery risks.

## 7. Review before implementation

A module is ready for implementation only when:

- Its need and learning outcomes are approved.
- Official sources, versions, platform support, and citations are recorded.
- The 180-minute plan and prerequisites are credible.
- Lab requirements and expected results can be reproduced or have explicit
  validation tasks assigned.
- Assessment, AI-use expectations, and safety/reset behavior are clear.
- Dependencies and the total course length still fit the agreed outline.

After approval, implement materials in the module's directory under
`curriculum/modules/`. Keep the research brief and lesson/lab implementation
separate so that technical research can be reviewed and updated independently.
