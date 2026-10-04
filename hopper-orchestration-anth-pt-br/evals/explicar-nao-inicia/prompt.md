---
description: Pedido de explicação não inicia a orquestração.
expected_outcome: Explica os papéis sem mostrar o painel e sem iniciar agentes.
tags: [com-escrita]
runs: 3
max_turns: 15
timeout_seconds: 300
allowed_tools: [Skill, AskUserQuestion, Agent, Read, Glob, Grep]
---
/hopper-orchestration-anth:hopper-orchestration-anth Explique em até cinco linhas como esta skill funciona. Não inicie nenhuma orquestração.
