---
description: Without a write tool, the orchestrator warns and stops, without improvising with agents.
expected_outcome: States that Write or Bash is missing and does not call the Agent tool.
tags: [without-write]
runs: 3
max_turns: 20
timeout_seconds: 300
allowed_tools: [Skill, AskUserQuestion, Agent, Read, Glob, Grep]
---
/hopper-orchestration-anth:hopper-orchestration-anth Orchestrate the creation of ./app/sum.mjs, which exports sum(a, b), with node:test tests and a commit in ./app.
