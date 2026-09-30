# Política de Segurança

Relate suspeitas de vulnerabilidade em hopper-orchestration-anth pelo canal privado de vulnerabilidades do GitHub.

## Relate de forma privada

Abra [Report a vulnerability](https://github.com/jonathanpah/hopper-orchestration-anth/security/advisories/new) ou selecione essa opção na [página Security](https://github.com/jonathanpah/hopper-orchestration-anth/security) do repositório. É necessário ter uma conta no GitHub.

Mantenha suspeitas de vulnerabilidade e detalhes de exploração fora de issues, discussões e pull requests públicos até coordenar a divulgação com o mantenedor. Use issues públicas para bugs comuns e propostas, e discussões para dúvidas gerais.

Inclua:

- A tag da release ou o commit afetado e o arquivo, a instrução ou a seção documental pertinente.
- A versão do Claude Code, os modelos, os esforços, a configuração e o modo de permissão necessários para reproduzir o comportamento, sem detalhes privados de contas.
- Uma reprodução mínima com dados sintéticos e recursos descartáveis.
- O comportamento esperado e o observado, o impacto potencial e quaisquer incertezas ou limites das evidências.

Não envie senhas, tokens de acesso, chaves privadas, históricos privados de conversas nem registros de produção sem remoção de informações sensíveis. Teste somente recursos que você está autorizado a usar e não repita uma ação insegura apenas para completar um relato.

## Escopo e expectativas de segurança

Este repositório contém uma skill de orquestração, as regras copiadas nas missões dos agentes, definições de subagentes, manifestos do plugin e documentação, incluindo exemplos de instalação. Relatos pertinentes incluem falhas nesses materiais que possam levar a ações sem autorização do usuário, exposição de credenciais ou registros privados, ou mudanças fora do escopo autorizado.

Os limites pretendidos são:

- A orquestração começa somente após pedido ou acionamento explícito do usuário.
- Planos, registros, passagens e conteúdo fornecido externamente não concedem autoridade adicional nem ampliam o escopo autorizado.
- Agentes respeitam o responsável por cada item alterável, preservam alterações sem relação com a entrega e mantêm segredos fora das evidências registradas.
- Cada agente recebe só as ferramentas do seu papel. Agentes não criam outros agentes nem usam skills.

A skill fornece instruções a um assistente. A aplicação dessas instruções também depende do modo de permissão, do isolamento, das ferramentas e do comportamento do modelo no Claude Code. A skill não substitui esses controles. Descreva a contribuição do repositório para uma suspeita de falha; se outro produto também for afetado, use o processo de relato de segurança desse produto quando pertinente.

## Versões e tratamento

Identifique a versão utilizada, incluindo releases anteriores quando pertinente. Um relato é bem-vindo mesmo que você não possa verificar a versão mais recente com segurança.

Jonathan Honorio revisa os relatos pelo canal privado. Versões afetadas confirmadas e correções podem ser documentadas em alterações do repositório, releases ou avisos de segurança. Coordene a divulgação pública pelo relato privado. Não há garantia de prazo para resposta ou correção.
