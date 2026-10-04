---
description: A request for an explanation does not start orchestration.
expected_outcome: Explains the roles without showing the panel and without starting agents.
tags: [with-write]
runs: 3
max_turns: 15
timeout_seconds: 300
allowed_tools: [Skill, AskUserQuestion, Agent, Read, Glob, Grep]
---
/hopper-orchestration-anth:hopper-orchestration-anth Explain in at most five lines how this skill works. Do not start any orchestration.
