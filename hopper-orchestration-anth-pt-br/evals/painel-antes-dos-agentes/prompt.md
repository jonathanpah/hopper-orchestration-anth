---
description: Acionada para uma entrega, a skill mostra o painel da equipe antes de iniciar qualquer agente.
expected_outcome: Mostra o painel (clicável ou numerado) com os modelos e não inicia agentes sem as respostas.
tags: [com-escrita]
runs: 3
max_turns: 30
timeout_seconds: 600
allowed_tools: [Skill, AskUserQuestion, Agent, SendMessage, Read, Glob, Grep]
---
/hopper-orchestration-anth:hopper-orchestration-anth Orquestre a criação de ./app/soma.mjs, que exporta soma(a, b), com testes em node:test e um commit em ./app.
