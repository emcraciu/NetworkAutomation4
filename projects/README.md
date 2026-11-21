# Student projects

Each individual or announced team should submit its project in its own
subdirectory:

```text
projects/<project-name>/
├── README.md
├── scripts/
└── ...
```

The project README must explain the topology, prerequisites, setup/run
instructions, expected results, and how to clean up. Include the valid pyATS
testbed and the scripts/data needed to create the GNS3 topology, bootstrap
devices, configure routing/firewall policy, and verify expected traffic from
the Ubuntu endpoint. Do not include credentials, tokens, private keys, or
other secrets.

Document AI use in the project materials as an `AI-USE.md` file (or a README
section) with one table:

| Date | Tool (name + version) | Task / prompt summary | What was accepted or changed, and why |
|---|---|---|---|
| | | | |

The purpose is to understand how students use AI, not to prohibit it. Log
honestly and briefly; undocumented AI use is itself a finding. See
[`docs/final-project-requirements.md`](../docs/final-project-requirements.md)
for the full project checklist, the 50 % review threshold, team rules, and
presentation logistics.

Submit a pull request against this repository with student deliverables only
under `projects/`. For team projects, announce the team early and ensure the
pull request contains commits from every team member. Review checks that at
least 50% of the final-project requirements are present; once the reviewed
pull request is merged, the project is considered accepted. Students also
present their project, slides, and running scripts to the 3PG team.
