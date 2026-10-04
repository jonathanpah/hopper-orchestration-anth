# hopper-orchestration-anth

[English](../hopper-orchestration-anth-en/README.md) | **Português (Brasil)**

Uma skill do Claude Code para coordenar uma etapa de trabalho com três subagentes Claude (executor, revisor e validador) até que todos os critérios estejam comprovados por evidências.

O orquestrador é a sessão do Claude em que o usuário aciona a skill. Ele define a etapa, escreve as missões, chama os agentes e decide devoluções, pausas e aceite. O executor produz a entrega, o revisor a confere nos arquivos e o validador comprova os critérios que dependem de algo fora da entrega, como uma instalação ou um serviço em execução. O fluxo começa somente quando o usuário aciona a skill.

O plugin tem três partes:

- o [SKILL.md](skills/hopper-orchestration-anth/SKILL.md), lido só pelo orquestrador;
- as [regras dos agentes](skills/hopper-orchestration-anth/regras-dos-agentes.md), que o orquestrador copia sem mudança para cada missão;
- nove [definições de subagentes](agents/), uma por papel (executor, revisor e validador) e esforço (High, xHigh e Max).

Consulte o [índice documental](index.md) ou veja [como contribuir](CONTRIBUTING.md).

Esta pasta contém a versão em português. Os comandos abaixo instalam esta versão, com o mesmo nome `hopper-orchestration-anth` usado em inglês.

## O que o fluxo exige

- Um painel da equipe, sempre mostrado antes do primeiro agente. Para executor, revisor e validador, o usuário clica no modelo (Opus, Sonnet ou Fable) e no esforço (High, xHigh ou Max). A escolha anterior aparece primeiro.
- Missões escritas pelo orquestrador que bastam sozinhas. Os agentes não veem a conversa com o usuário.
- Executor, revisor e validador em toda etapa. Revisor e validador só saem se o usuário os dispensar. Mais de um executor, só para partes independentes.
- Critérios observáveis, classificados como locais ou de destino, versão entregue identificada e um único responsável por item alterável.
- Relatórios que começam por uma passagem com campos fixos, evidências em arquivos e um registro de eventos com a hora real.
- Devoluções graduadas: correção, depois critério exato e, sem redução dos achados, pausa para decisão do usuário.

O orquestrador mantém o modelo e o esforço da conversa principal. Ele confere, na transcrição de cada agente, o modelo e o esforço usados. Agentes não criam outros agentes nem usam skills. Explicar ou revisar a skill não aciona o fluxo.

## Compatibilidade e limites

A skill é feita para o Claude Code, na CLI e no aplicativo, em macOS e Linux. Ela depende das ferramentas Agent, SendMessage e AskUserQuestion e dos subagentes que o plugin traz. Fora do Claude Code, o fluxo não está disponível.

O SKILL.md segue o [formato Agent Skills](https://agentskills.io/specification) e tem `disable-model-invocation: true`: o Claude não aciona a skill por conta própria. Cada agente recebe só as ferramentas do seu papel e herda o modo de permissão da sessão. A skill recomenda o modo auto ou o sandbox.

O reconhecimento da skill não comprova que todo o fluxo funciona no seu ambiente. Antes de iniciar agentes, o orquestrador confere as próprias ferramentas e o modo de permissão. A falta de algo essencial exige uma decisão do usuário. A skill não alega economia de custo medida.

## Instalação

O plugin e a skill se chamam `hopper-orchestration-anth` nos dois idiomas. Escolha um idioma só. A pasta `hopper-orchestration-anth-pt-br/` contém o plugin completo em português: a skill em `skills/hopper-orchestration-anth/` e os subagentes em `agents/`.

Clone o repositório uma vez:

```sh
mkdir -p "$HOME/.local/share"
git clone https://github.com/jonathanpah/hopper-orchestration-anth.git "$HOME/.local/share/hopper-orchestration-anth"
```

Se o destino já existir, confira sua origem e alterações locais antes de atualizá-lo.

### Claude Code

```sh
claude plugin marketplace add "$HOME/.local/share/hopper-orchestration-anth/hopper-orchestration-anth-pt-br"
claude plugin install hopper-orchestration-anth@hopper-orchestration-anth
```

Numa sessão já aberta, envie `/reload-plugins` como mensagem separada para carregar a skill e os subagentes. Não é preciso configurar `skillOverrides`: o próprio SKILL.md impede o acionamento automático.

### Aplicativo e nomes

O identificador completo é `hopper-orchestration-anth:hopper-orchestration-anth`: o primeiro trecho identifica o plugin, e o segundo identifica a skill. O produto acrescenta esse prefixo. Acione a skill com `/hopper-orchestration-anth:hopper-orchestration-anth`. Os subagentes aparecem como `hopper-orchestration-anth:executor-high`, `hopper-orchestration-anth:revisor-xhigh` e assim por diante; só o orquestrador da skill os usa.

A instalação pela CLI não comprova presença no catálogo do aplicativo. No aplicativo do Claude, confira **Personalização → Plugins → Meus** e **Habilidades → Meus**. Quando o plugin não estiver nesse catálogo, adicione o pacote pela opção de upload de plugin do Claude e confira as duas listas. O ZIP deve conter os arquivos da pasta do idioma escolhido, incluindo `.claude-plugin/plugin.json`, `skills/` e `agents/`. Uma instalação da conta e uma instalação local são registros distintos; evite manter duas versões concorrentes.

### Verificação

Confira nome, versão e conteúdo com `claude plugin list` e `claude plugin details hopper-orchestration-anth@hopper-orchestration-anth`: a skill e os nove agentes devem aparecer. Verifique o aplicativo separadamente. A instalação não executa agentes; testar o fluxo exige acionar a skill com uma tarefa delimitada. A pasta [`evals/`](evals/) traz os casos da suíte oficial; para repeti-los, rode nesta pasta `claude plugin eval . --tag com-escrita --allow-tools Write Bash SendMessage` e `claude plugin eval . --tag sem-escrita`.

## Uso

Acione `/hopper-orchestration-anth:hopper-orchestration-anth` e descreva a entrega autorizada. Por exemplo:

> Orquestre a correção do cálculo de frete neste projeto. A entrega é o código corrigido, com testes, num commit; não publique nada.

Antes de iniciar os agentes, o orquestrador mostra o painel da equipe. Para executor, revisor e validador, clique no modelo e no esforço.

Esta instalação usa a skill em português. Os agentes se comunicam no idioma do usuário. Um plano de tarefa ou uma passagem, por si só, não autoriza iniciar a orquestração nem ampliar seu escopo.

## Registros

Cada etapa cria uma subpasta `docs-by-hopper-orchestration/AAAAMMDD-HHMMSS-<tema>` na raiz do projeto em que a orquestração foi acionada. Ela contém as missões (`missao-<papel>.md`), os relatórios das rodadas (`r<rodada>-<papel>.md`), as evidências (`evidencias/`), o `eventos.md` e o `resumo.md` final.

Datas e horas usam o fuso do sistema, lido no momento de cada registro, sempre com o deslocamento em relação a UTC.

Mantenha os registros de execução fora deste repositório quando possível. Não envie credenciais, históricos privados de conversas, caminhos pessoais ou evidências operacionais sem remoção de informações sensíveis em uma contribuição.

## Atualizações

Confira as alterações locais e atualize o clone com `git pull --ff-only`. Depois, execute `claude plugin marketplace update hopper-orchestration-anth` e `claude plugin update hopper-orchestration-anth@hopper-orchestration-anth`, e envie `/reload-plugins` nas sessões abertas. Se usou upload no aplicativo, atualize também o pacote da conta. Confira o conteúdo e a descoberta novamente.

Para remover, use `claude plugin uninstall hopper-orchestration-anth@hopper-orchestration-anth` e remova a instalação da conta no aplicativo, se existir. Os registros dos projetos permanecem preservados.

## Contribuição

Para suspeitas de vulnerabilidade, siga a [Política de Segurança](SECURITY.md) e relate de forma privada antes de compartilhar detalhes publicamente.

Use [Issues](https://github.com/jonathanpah/hopper-orchestration-anth/issues) para problemas reproduzíveis e propostas concretas. Use [Discussions](https://github.com/jonathanpah/hopper-orchestration-anth/discussions) para dúvidas, experiências e ideias que ainda não sejam propostas de alteração.

Para propor uma edição, crie um fork do repositório, uma branch e um pull request. Colaboradores podem comentar e sugerir mudanças; o mantenedor decide o que será integrado. Consulte [CONTRIBUTING.md](CONTRIBUTING.md) para as expectativas de escopo, evidências e revisão, e o [Código de Conduta](CODE_OF_CONDUCT.md) para os padrões de participação.

## Licença e mantenedor

Copyright (c) 2026 Jonathan Honorio. Publicado sob a [Licença MIT](LICENSE).

Mantido por [Jonathan Honorio (@jonathanpah)](https://github.com/jonathanpah). Este é um projeto independente, sem alegação de vínculo ou endosso da Anthropic.
