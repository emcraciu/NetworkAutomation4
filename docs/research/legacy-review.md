# Review of previous course attempts

This review uses the local folders `NetworkAutomation` and
`NetworkAutomation3` as curriculum evidence. It does not import their code,
student submissions, credentials, or configuration into this project.

## First attempt: `NetworkAutomation`

The repository is arranged as `modul1` through `modul8` and `module9`, alongside
shared connector examples, testbeds, projects, and homework. The early modules
include a Python fundamentals notebook, data structures and control flow,
functions/imports, classes, files, and related language features. Later
material includes asynchronous execution, Paramiko/Netmiko SSH, REST/RESTCONF,
pyATS/testbeds, NAPALM, Cisco FTD REST configuration, and unit tests/mocking.

The project brief in `examen_proiect.md` emphasizes a multi-device testbed,
initial and interface configuration, DHCP, static routes, an FTD access rule,
connectivity checks through pyATS, unit tests/mocking, and optional OSPF and
parallel configuration. Homework docs include Python-level exercises such as
collecting switch/VLAN information and building configuration objects.

## Second attempt: `NetworkAutomation3`

This repository has 18 numbered module folders. The first seven are largely
Python language topics: built-ins and data types, flow control, functions,
imports/files/bytes, classes, context managers and generators, inheritance,
and map/filter. Later modules include subprocess, asyncio, multiprocessing,
threading and Paramiko, pyATS, REST, Genie, FTD API automation, testing/mocking,
Ansible, and GNS3 API usage. Its `lib/gns3_api/setup_env_gns3.py` and
`modul18/gns3_create.py` are evidence of GNS3 API exploration, not validated
course material.

The second attempt is more extensive than the first, but still spends many
modules on general Python content before network-automation work. Given the
new audience prerequisite (Python Essentials 1 and 2), most such sessions
should be reconsidered. The course may still need small, applied refreshers
for classes, exceptions, packaging, testing, data formats, and concurrency
where these directly support a lab.

## Reusable curriculum signals

- A multi-vendor/device testbed and a repeatable GNS3 lab can anchor the
  course and the final project.
- SSH/CLI, REST, and pyATS/Genie appeared in both attempts and are strong
  candidates for investigation.
- Earlier work additionally explored RPC-adjacent management mechanisms,
  NETCONF/RESTCONF, NAPALM, FTD APIs, Ansible, testing, and concurrency.
  Clarify whether each is in scope and how much depth is useful.
- Device configuration should be paired with verification, failure handling,
  and testing rather than taught as command delivery alone.
- The final project can integrate GNS3 API topology setup with device
  automation and automated verification, subject to confirmed images,
  licensing, and available lab resources.

## Limitations and follow-up

This is a structural and document-level review, not a technical validation of
the old code, dependencies, APIs, or lab instructions. The prior folders also
contain student work and machine-specific testbeds; do not copy them into the
new course. Any technical topic selected for the new outline must be
researched against current official documentation and tested against the
confirmed platform versions.

The requested second folder was found as `NetworkAutomation3`
