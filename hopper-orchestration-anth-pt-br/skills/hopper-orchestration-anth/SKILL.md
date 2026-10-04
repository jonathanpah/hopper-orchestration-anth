---
name: hopper-orchestration-anth
description: Coordena uma etapa de trabalho com três subagentes Claude (executor, revisor e validador), cada um com o modelo e o esforço escolhidos pelo usuário, até que todos os critérios estejam comprovados por evidências. Use quando o usuário acionar esta skill para orquestrar uma entrega com agentes.
compatibility: Requer Claude Code com o plugin hopper-orchestration-anth, que traz os subagentes, e as ferramentas Agent, SendMessage e AskUserQuestion.
disable-model-invocation: true
---

# hopper-orchestration-anth

Quem lê este arquivo é o **orquestrador**: a sessão do Claude em que o usuário acionou a skill. Os agentes não leem este arquivo. Cada agente lê só a própria missão, que o orquestrador escreve. Os termos e as regras que vão nas missões estão em `${CLAUDE_SKILL_DIR}/regras-dos-agentes.md`; o orquestrador lê esse arquivo inteiro antes da preparação.

## 1. Quem participa

| Participante | Quem é | O que faz | Nome nos arquivos |
| --- | --- | --- | --- |
| Usuário | A pessoa que acionou a skill | Autoriza a etapa, escolhe modelos e esforços e toma as decisões que fogem da autorização | — |
| Orquestrador | A sessão do Claude que lê esta skill | Define a etapa, escreve as missões, chama os agentes, decide devoluções, pausas e aceite e fala com o usuário. Não produz, não revisa e não valida | `orquestrador` |
| Executor | Subagente | Produz a entrega, verifica e corrige | `executor`, `executor-2`… |
| Revisor | Subagente | Confere a entrega nos arquivos: requisitos, erros e omissões | `revisor` |
| Validador | Subagente | Confere o tipo de cada critério e comprova os critérios de destino | `validador` |

Os agentes não falam com o usuário nem entre si. Tudo passa pelo orquestrador.

## 2. Marcadores e arquivos

Um texto entre `< >` é um marcador: o orquestrador o troca pelo valor real.

| Marcador | Significa | Exemplo |
| --- | --- | --- |
| `<papel>` | Nome do agente nos arquivos | `revisor` |
| `<esforço>` | `high`, `xhigh` ou `max` | `xhigh` |
| `<modelo>` | `opus`, `sonnet` ou `fable` | `sonnet` |
| `<rodada>` | Número da rodada | `2` |
| `<tema>` | Tema da etapa, em até três palavras separadas por hífen | `ajuste-login` |
| `<pasta-da-etapa>` | `docs-by-hopper-orchestration/<AAAAMMDD-HHMMSS>-<tema>/`, na raiz do projeto, com a data e a hora do início da etapa | — |
| `<descrição>` | Descrição curta de uma evidência | `testes` |
| `<id-do-agente>` | Identificador que a ferramenta Agent devolve ao iniciar o agente | — |
| `<projeto>` e `<sessão>` | Pasta do projeto e sessão do orquestrador em `~/.claude/projects/` | — |

Arquivos de cada etapa, todos dentro de `<pasta-da-etapa>`:

| Arquivo | Quem escreve | Quem lê | Conteúdo |
| --- | --- | --- | --- |
| `missao-<papel>.md` | Orquestrador | O agente daquele papel | A missão do agente (seção 6) |
| `r<rodada>-<papel>.md` | O agente | Orquestrador e os agentes seguintes | O relatório da rodada, que começa pela passagem |
| `evidencias/r<rodada>-<papel>-<descrição>` | O agente | Orquestrador e os agentes seguintes | Saídas de comandos e outras provas |
| `eventos.md` | Orquestrador | Orquestrador e usuário | Uma linha por evento da etapa (seção 10) |
| `resumo.md` | Orquestrador | Usuário e a próxima etapa | O resultado da etapa (seção 10) |

## 3. Quando o orquestrador usa esta skill

Só quando o usuário aciona a skill. Explicar, revisar ou planejar com a skill não inicia a orquestração.

## 4. Preparação: o que o orquestrador faz antes de iniciar agentes

1. **Lê a etapa anterior.** Se houver, o orquestrador lê o `resumo.md` dela (em etapas antigas, `summary.md`) e não refaz trabalho já aceito.
2. **Define a etapa:** entrega, critérios com o tipo de cada um, limites e o responsável por cada item alterável.
3. **Confere as próprias ferramentas:** Write, Bash, Agent e SendMessage, carregando as adiadas. Se faltar alguma, avisa o usuário e para.
4. **Confere o modo de permissão da sessão.** Os agentes herdam esse modo. O orquestrador recomenda o modo auto, que revisa as ações dos agentes, ou o sandbox. Se a sessão estiver sem permissões (bypass), avisa o usuário antes de iniciar agentes e segue a decisão dele. Se a etapa incluir ações que o modo auto costuma negar, como instalar em produção, apagar algo sem volta ou ler credenciais, o orquestrador avisa antes de iniciar agentes e pede, de uma vez, a troca do modo da sessão ou uma regra de permissão que libere só aquela ação.
5. **Dimensiona a equipe.** Revisor e validador só saem da equipe se o usuário os dispensar. Se a equipe parecer grande ou pequena demais para a entrega, o orquestrador recomenda outra ao usuário, por exemplo sem validador quando não houver critério de destino. Mais de um executor, só para partes independentes. Em tudo isso, vale a decisão do usuário.
6. **Mostra o painel.** Antes de qualquer chamada à ferramenta Agent, o orquestrador pergunta a equipe ao usuário numa única pergunta da ferramenta AskUserQuestion, com até quatro opções clicáveis. Cada opção é uma equipe completa, com o modelo e o esforço de cada papel na ordem executor, revisor e validador, por exemplo "Sonnet high · Opus high · Opus high". A escolha anterior do usuário vem primeiro; as outras são combinações comuns. Para outra equipe, o usuário a escreve em "Outro". Sem essa ferramenta, o orquestrador mostra as mesmas opções numeradas e pede o número ou a equipe por escrito. A escolha do executor vale para todos os executores.
   - Modelos: Opus (`opus`), Sonnet (`sonnet`) e Fable (`fable`).
   - Esforços: High (`high`), xHigh (`xhigh`) e Max (`max`).
7. **Cria a pasta da etapa e escreve uma missão para cada agente** (seção 6).
8. **Inicia o executor** (seção 5).

Preparar não autoriza executar. Nenhum plano ou passagem amplia o que o usuário autorizou.

## 5. Como o orquestrador inicia e retoma agentes

O plugin traz uma definição de subagente para cada papel e esforço, com o tipo `hopper-orchestration-anth:<papel>-<esforço>` (exemplo: `hopper-orchestration-anth:revisor-xhigh`). A definição fixa o esforço e as ferramentas:

- executor: Read, Write, Edit, Bash, Grep e Glob;
- revisor e validador: Read, Write, Bash, Grep e Glob;
- nenhum agente tem Agent nem Skill: agentes não criam agentes nem usam skills.

**Para iniciar um agente,** o orquestrador:

1. chama a ferramenta Agent com o tipo do agente, `model` igual ao `<modelo>` escolhido e execução em segundo plano;
2. envia só este pedido: "Leia e cumpra `<pasta-da-etapa>/missao-<papel>.md`. Rodada `<rodada>`.", mais o caminho do último relatório, quando houver. Nada da conversa com o usuário vai no pedido;
3. espera o aviso de término, sem consultar o agente enquanto ele trabalha;
4. confere o modelo e o esforço usados na transcrição do agente, `~/.claude/projects/<projeto>/<sessão>/subagents/agent-<id-do-agente>.jsonl`, nos campos `"model"` e `"effort"`. Se não conseguir confirmar, registra isso e avisa o usuário.

**Para retomar o mesmo agente** numa correção, o orquestrador chama a ferramenta SendMessage com o `<id-do-agente>` e informa a nova rodada e o caminho do relatório que motivou a retomada. O agente retomado mantém o histórico e o modelo. Se a retomada falhar, o orquestrador inicia outro agente do mesmo papel, com a missão e o último relatório.

**O orquestrador não:**

- usa agentes no próprio trabalho: ele mesmo escreve a pasta da etapa, as missões, `eventos.md` e `resumo.md`;
- inicia agentes pelo terminal nem cria programas para controlá-los;
- troca modelo ou esforço por conta própria.

## 6. Como o orquestrador escreve a missão

A missão é o único texto que o agente recebe; ele não vê a conversa com o usuário, para que revisão e validação não herdem conclusões. O orquestrador grava cada missão em `<pasta-da-etapa>/missao-<papel>.md` com estes itens, nesta ordem:

1. o papel do agente e o nome dele nos arquivos (`executor`, `executor-2`, `revisor` ou `validador`);
2. objetivo, entrega e critérios, cada critério com o seu tipo;
3. a pasta da etapa, a especificação, os papéis da equipe e as decisões já tomadas, com os motivos;
4. as instruções do usuário e do projeto que valem para a etapa, inclusive o idioma;
5. os acessos, os limites e os itens alteráveis sob responsabilidade do agente; com mais de um executor, qual deles junta as partes;
6. copiadas sem mudança de `regras-dos-agentes.md`: as partes "Termos", "Regras de todos os agentes", a parte do papel do agente, "Passagem" e "Comunicação".

Se uma decisão mudar critérios, o tipo de um critério ou os limites, o orquestrador atualiza as missões afetadas e registra a decisão, com o motivo, no item 3 de cada uma.

## 7. O que o orquestrador faz quando recebe uma passagem

O orquestrador confere se a passagem tem todos os campos da parte "Passagem" de `regras-dos-agentes.md`. Se faltar algum, pede ao mesmo agente que complete; isso não conta como devolução. Depois, segue a tabela abaixo. Nela, "chama" quer dizer iniciar o agente na rodada 1 e retomar o mesmo agente nas rodadas seguintes.

| Situação | O que o orquestrador faz |
| --- | --- |
| Um executor terminou a sua parte, e outro executor junta as partes | Chama o executor que junta as partes |
| O executor terminou a entrega | Chama o revisor |
| O revisor aprovou, e a equipe tem validador | Chama o validador |
| O revisor aprovou, e a equipe não tem validador | Faz o aceite (seção 9) |
| O validador aprovou | Faz o aceite (seção 9) |
| O revisor ou o validador devolveu a entrega | Trata a devolução (seção 8) |
| O validador discordou do tipo de um critério | Decide o tipo, registra, atualiza as missões e retoma o validador só com os critérios de destino ainda não conferidos |
| O agente ficou bloqueado, precisa de contexto ou concluiu com ressalvas | Resolve, se estiver dentro da autorização; senão, pausa e leva ao usuário (seção 9) |

Com uma parte pausada ou bloqueada, o orquestrador espera a passagem final do executor antes de chamar o revisor. Só executores de partes independentes trabalham ao mesmo tempo.

## 8. Como o orquestrador trata as devoluções

O orquestrador conta as devoluções da etapa e age assim:

- **Primeira devolução:** retoma o executor e pede que corrija os achados do relatório que devolveu a entrega.
- **Segunda devolução:** antes de retomar o executor, define o critério exato que a próxima rodada precisa cumprir e o inclui no pedido.
- **Terceira devolução em diante:** compara o total de achados abertos, do revisor e do validador somados, com o da devolução anterior. Se diminuiu, registra a decisão de seguir, define o critério exato e retoma o executor. Se não diminuiu, pausa e leva ao usuário o histórico, a causa provável e uma recomendação.
- **O executor entrega de novo, sem mudança, a versão já devolvida:** o orquestrador não chama o revisor, porque não há mudança a conferir. Conta uma nova devolução, com os mesmos achados abertos, e segue as regras acima.
- **O mesmo impedimento outra vez, sem nada ter mudado:** pausa e leva ao usuário.

O orquestrador registra cada decisão em `eventos.md`, com o motivo. Não impõe prazo. Não aceita nem cancela a etapa só porque as falhas se repetem.

## 9. Aceite, pausa e encerramento

- **Aceite:** o orquestrador aceita a etapa quando cada critério tem evidência da versão entregue e não há achado aberto. Sugestões não impedem o aceite.
- **Pausa:** quando a etapa depende de uma decisão do usuário, o orquestrador para e entrega ao usuário o resultado parcial, o impedimento e a próxima ação recomendada. Os itens alteráveis continuam com os seus responsáveis. Com a resposta do usuário, o orquestrador retoma a etapa com os mesmos agentes.
- **Encerramento:** depois do aceite, ou quando o usuário decide encerrar, o orquestrador escreve o `resumo.md`, libera os itens alteráveis e apaga os temporários que ele mesmo criou. Se o pedido do usuário incluir algum commit, o orquestrador faz também um commit só da pasta da etapa, no repositório Git que a contém; se o pedido não incluir commit, ou se a pasta não estiver num repositório Git, registra no `resumo.md` que ela ficou fora do Git. O orquestrador não pergunta ao usuário sobre esse commit.

## 10. Como o orquestrador registra a etapa

**`eventos.md`:** o orquestrador escreve uma linha por evento, com data, hora e fuso lidos no momento do registro. Eventos: etapa definida; equipe escolhida; agente iniciado ou retomado, com tipo, modelo e `<id-do-agente>`; agente terminado, com modelo e esforço confirmados; passagem recebida; devolução; decisão, com o motivo; pausa; aceite. Em cada linha, o orquestrador diz se a ação foi decidida, tentada ou realizada, e só registra como realizada o que confirmou. Registra também as falhas.

**`resumo.md`:** ao encerrar a etapa, o orquestrador escreve: resultado; versão entregue; cada critério com o caminho da evidência; equipe, com papel, modelo e esforço; decisões do usuário, com as palavras dele; limitações; pendências; próximo passo.

Depois de interrupção ou compactação, o orquestrador relê esta skill e continua do último evento confirmado em `eventos.md`.

## 11. Como o orquestrador fala com o usuário

O orquestrador segue a parte "Comunicação" de `regras-dos-agentes.md`. Além disso:

- informa andamento só quando muda o resultado, o impedimento ou a decisão necessária;
- não pede de novo o que o usuário já confirmou, exceto o painel.
