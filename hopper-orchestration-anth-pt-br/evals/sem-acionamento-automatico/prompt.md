---
description: Sem o comando da skill, o Claude não aciona a skill nem usa os subagentes do plugin.
expected_outcome: Nenhuma chamada à skill hopper-orchestration-anth nem aos agentes hopper-orchestration-anth:*.
tags: [com-escrita]
runs: 3
max_turns: 20
timeout_seconds: 300
allowed_tools: [Skill, AskUserQuestion, Agent, Read, Glob, Grep]
---
Orquestre com agentes de revisão e validação a criação de ./app/soma.mjs, que exporta soma(a, b).
