---
description: Invoked for a deliverable, the skill shows the team panel before starting any agent.
expected_outcome: Shows the panel (clickable or numbered) with the models and does not start agents without the answers.
tags: [with-write]
runs: 3
max_turns: 30
timeout_seconds: 600
allowed_tools: [Skill, AskUserQuestion, Agent, SendMessage, Read, Glob, Grep]
---
/hopper-orchestration-anth:hopper-orchestration-anth Orchestrate the creation of ./app/sum.mjs, which exports sum(a, b), with node:test tests and a commit in ./app.
