---
description: Sem ferramenta de escrita, o orquestrador avisa e para, sem improvisar com agentes.
expected_outcome: Informa que faltam Write ou Bash e não chama a ferramenta Agent.
tags: [sem-escrita]
runs: 3
max_turns: 20
timeout_seconds: 300
allowed_tools: [Skill, AskUserQuestion, Agent, Read, Glob, Grep]
---
/hopper-orchestration-anth:hopper-orchestration-anth Orquestre a criação de ./app/soma.mjs, que exporta soma(a, b), com testes em node:test e um commit em ./app.
