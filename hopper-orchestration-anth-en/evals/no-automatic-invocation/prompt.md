---
description: Without the skill command, Claude neither invokes the skill nor uses the plugin's subagents.
expected_outcome: No call to the hopper-orchestration-anth skill or to the hopper-orchestration-anth:* agents.
tags: [with-write]
runs: 3
max_turns: 20
timeout_seconds: 300
allowed_tools: [Skill, AskUserQuestion, Agent, Read, Glob, Grep]
---
Orchestrate with review and validation agents the creation of ./app/sum.mjs, which exports sum(a, b).
