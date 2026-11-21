# Leituras — Grupo A

Resumo das páginas 1 a 6 (documentação oficial do Claude Code, Anthropic e Claude Agent SDK). Tudo abaixo vem do conteúdo das páginas lidas em 21/11/2026; o que a página não trazia está marcado como "não consta".

---

## 1. Visão geral do Claude Code

- **URL final que funcionou:** https://code.claude.com/docs/en/overview
- **O que é:** ferramenta de programação agentica que lê a base de código, edita arquivos, executa comandos e se integra às ferramentas de desenvolvimento. Disponível no terminal, IDE, app desktop e navegador.

**Pontos práticos:**
- Instalação nativa recomendada: `curl -fsSL https://claude.ai/install.sh | bash` (macOS, Linux, WSL); no Windows, `irm https://claude.ai/install.ps1 | iex` (PowerShell) ou um comando `curl` para CMD. Também há Homebrew (`claude-code` e `claude-code@latest`), WinGet e pacotes apt, dnf ou apk. A instalação nativa atualiza sozinha; Homebrew e WinGet não, e precisam de upgrade manual.
- Uso básico: `cd seu-projeto` e depois `claude`. No primeiro uso pede login; se `ANTHROPIC_API_KEY` estiver definida e aprovada, o login é pulado.
- Modo não interativo: `claude -p "..."`, com pipes, por exemplo `tail -200 app.log | claude -p "..."`.
- `CLAUDE.md` na raiz do projeto é lido no início de toda sessão. Se existir `AGENTS.md`, o Claude Code pode lê-lo no lugar.
- Memória automática: o Claude registra aprendizados entre sessões sem o usuário escrever nada.
- Skills (`/review-pr`, `/deploy-staging`) empacotam fluxos repetíveis; hooks rodam comandos antes ou depois das ações do Claude (por exemplo, formatar após cada edição).
- Subagentes para trabalho em paralelo; o Agent SDK permite agentes próprios com controle de orquestração, ferramentas e permissões.
- Agendamento: tarefas agendadas na nuvem (funcionam com o computador desligado), tarefas no desktop, `/loop` para repetir um prompt dentro da sessão CLI.
- Integrações: MCP, GitHub Actions, GitLab CI/CD, Slack (menção a `@Claude` gera um PR), Chrome para depurar aplicações web, Remote Control para continuar a sessão pelo celular.
- Superfícies (terminal, VS Code, JetBrains, desktop, web) usam o mesmo motor, então CLAUDE.md, configurações e servidores MCP valem em todas.

**Custo/requisitos:**
- A maioria das superfícies exige assinatura Claude ou conta no Anthropic Console. Terminal, VS Code e JetBrains também aceitam provedores terceiros.
- App desktop exige assinatura paga. Recursos como `claude --teleport` e `/desktop` exigem assinatura claude.ai.
- JetBrains exige o CLI instalado separadamente.
- Academy da Anthropic: cursos gratuitos (Claude Code 101 e Claude Code in Action), segundo a página.

---

## 2. Boas práticas do Claude Code

- **URL final que funcionou:** https://code.claude.com/docs/en/best-practices
- **O que é:** guia sobre o que fazer quando os padrões do Claude Code não bastam: informar o que ele não consegue ver, dar um jeito de verificar o trabalho e impor regras sem exceção.

**Pontos práticos:**
- Separe instruções não determinísticas (prompt, CLAUDE.md, skills, que o modelo lê e decide seguir) de determinísticas (hooks, regras de deny e configurações gerenciadas, que o harness aplica sempre). Regra que não pode ter exceção vai para hook ou deny rule. Se uma linha do CLAUDE.md é ignorada mesmo com ênfase, ela provavelmente deveria virar hook.
- **CLAUDE.md curto:** inclua só o que o Claude não descobre lendo o repositório: comandos de teste, typecheck e lint; armadilhas do código; decisões fora do repositório; correções já feitas duas vezes. `/init` gera um ponto de partida, `/doctor` sugere cortes, e a memória recomenda ficar abaixo de 200 linhas. Preferências pessoais vão para `CLAUDE.local.md`, que entra no `.gitignore`.
- **Regras por parte do código:** `.claude/rules/` com `paths:` carrega a regra só ao trabalhar nos arquivos indicados.
- **Procedimentos do time em skills:** `.claude/skills/<nome>/SKILL.md`. Para ações com efeito colateral (ex.: publicar release), use `disable-model-invocation: true` para o Claude não disparar sozinho.
- **Dar um check para o Claude:** algo que devolva passou ou falhou (testes, build, linter, screenshot). `/goal` mantém o Claude trabalhando até uma condição ser atendida; um hook `Stop` com `exit 2` impede o fim do turno enquanto o typecheck falhar. Peça evidência (comando e saída), não apenas "todos os testes passam".
- **Hooks de exemplo:** `PostToolUse` com `Edit|Write` para formatar; `PreToolUse` para bloquear edições em `migrations/`.
- **Permissões:** regras `deny` valem em todos os modos, inclusive auto e bypassPermissions (ex.: `Read(./.env)`). O sandbox (`/sandbox`) limita acesso ao sistema de arquivos e à rede em macOS, Linux e WSL2. A página alerta que regras casam com chamadas de ferramenta, não com a intenção: um script que abre `.env` não é barrado por `Read`.
- **Escopos de configuração:** `settings.local.json` e `~/.claude/settings.json` são pessoais; `.claude/settings.json` vale para o repositório; configurações gerenciadas são da organização. Mantenha o conjunto de hooks e deny rules pequeno.
- **Trabalho sem supervisão:** antes de deixar rodar, defina o que é "pronto", como verificar, o que pode tocar, um limite (`--max-turns`, `--max-budget-usd`) e quem aprova o resultado. Para scripts, use `claude -p` com `--allowedTools`, `--permission-mode dontAsk` e `--max-turns`; `--bare` pula hooks, skills, MCP e CLAUDE.md.
- **Paralelismo:** `claude -w <nome>` abre sessão em worktree própria; `.worktreeinclude` copia arquivos ignorados pelo git (como `.env`) para cada worktree; `/batch` divide mudanças grandes em 5 a 30 unidades, mostra o plano e abre PRs.
- **Controles de sessão e custo:** `Esc` interrompe; `/rewind` volta código e/ou conversa; `/clear` entre tarefas não relacionadas; `/compact` com foco; `/usage` mostra onde o consumo foi; `/effort` ajusta o esforço (padrão `medium`); subagentes `model: haiku` para tarefas simples; `/rename` e `claude --resume <nome>` retomam sessões.

**Custo/requisitos:** a página não trata de preços. Menciona modo auto para sessões de terminal e VS Code, `--max-budget-usd` para execuções `-p` e limites de gasto configurados pelo usuário. Cache de prompt dura cerca de 1 hora em assinatura e 5 minutos em API key, provedor de nuvem ou créditos de uso.

---

## 3. Building effective agents (Anthropic)

- **URL final que funcionou:** https://www.anthropic.com/research/building-effective-agents
- **O que é:** artigo de pesquisa da Anthropic sobre padrões para construir agentes de IA, separando workflows de agentes e descrevendo padrões de composição.

**Pontos práticos:**
- **Workflow** = LLMs e ferramentas orquestrados por caminhos de código definidos. **Agente** = o próprio LLM decide os passos e o uso de ferramentas.
- Comece pela solução mais simples. Muitas vezes, otimizar uma chamada com retrieval e exemplos no contexto já basta. Aumente a complexidade só com ganho comprovado.
- **Prompt chaining:** etapas sequenciais com verificações programáticas entre elas (troca latência por precisão).
- **Routing:** classifica a entrada e encaminha para um fluxo especializado; perguntas simples podem ir a modelos menores.
- **Parallelization:** sectioning (subtarefas independentes em paralelo) ou voting (mesma tarefa várias vezes).
- **Orchestrator-workers:** um LLM central define subtarefas dinamicamente, delega e sintetiza.
- **Evaluator-optimizer:** um LLM gera, outro avalia em loop de refinamento; útil quando há critérios claros.
- Agentes precisam de condições de parada (ex.: máximo de iterações), guardrails e testes em ambiente sandbox.
- Cuidado com frameworks: podem esconder prompts e respostas e dificultar a depuração. A página sugere começar pela API direta.
- Dedique esforço ao design das ferramentas (ACI): descrições claras, exemplos, casos de borda e limites entre ferramentas. Exemplo citado: exigir caminhos absolutos eliminou erros com caminhos relativos.
- Os casos mais promissores citados são suporte ao cliente e agentes de código, onde o sucesso é verificável.

**Custo/requisitos:** a página não menciona preço nem requisitos de conta. Cita o Claude Agent SDK, o Strands Agents SDK, Rivet e Vellum como frameworks.

---

## 4. Harnesses para agentes de longa duração (Anthropic)

- **URL final que funcionou:** https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **O que é:** post de engenharia sobre como fazer agentes de codificação trabalharem por muitas sessões, com um harness que mantém o progresso entre elas.

**Pontos práticos:**
- **Problema:** cada sessão nova começa sem memória da anterior; compactação de contexto não basta. Dois modos de falha: o agente tenta fazer tudo de uma vez e deixa pela metade; um agente posterior vê progresso parcial e declara o projeto concluído.
- **Initializer agent (primeira sessão):** cria `init.sh`, um arquivo de progresso (`claude-progress.txt`) e um commit inicial no git.
- **Coding agent (sessões seguintes):** avança em uma funcionalidade por sessão e deixa o ambiente limpo, pronto para merge.
- **Lista de funcionalidades em JSON:** itens começam como não aprovados; o JSON foi preferido ao Markdown porque o modelo altera menos esse formato. Testes não devem ser removidos nem editados.
- **Rotina de início de sessão:** `pwd`, ler o log do git e o arquivo de progresso, escolher a funcionalidade prioritária, rodar `init.sh` e um teste básico de ponta a ponta antes de implementar algo novo.
- **Git como rede de segurança:** commits descritivos permitem reverter e voltar a estados funcionais.
- **Teste de ponta a ponta com automação de navegador** (Puppeteer MCP), feito como um usuário faria; melhorou muito a detecção de bugs que só aparecem na interface.
- **Verificar antes de marcar como concluído:** o agente tendia a declarar sucesso sem validar o fluxo completo.
- **Limitações citadas:** alertas nativos do navegador não aparecem para o Puppeteer MCP; não está claro se um agente único é melhor que multiagente; agentes especializados (teste, QA, limpeza) não foram validados; a demonstração foi feita só em desenvolvimento web full-stack.
- Os dois agentes usam o mesmo system prompt e as mesmas ferramentas; só mudam os prompts iniciais.

**Custo/requisitos:** a página não trata de custo nem de requisitos de acesso.

---

## 5. Claude Agent SDK — Python e TypeScript

### 5a. Python

- **URL final que funcionou:** https://github.com/anthropics/claude-agent-sdk-python
- **O que é:** SDK em Python para o Claude Agent, que usa o mesmo núcleo do Claude Code.

**Pontos práticos:**
- Instalação: `pip install claude-agent-sdk`.
- `query()` é uma função assíncrona que devolve um iterador de mensagens.
- `ClaudeSDKClient` permite conversas bidirecionais e é o que habilita **ferramentas customizadas** e **hooks**; `query()` não os suporta.
- Ferramentas próprias: funções Python com `@tool`, empacotadas por `create_sdk_mcp_server` (servidor MCP in-process, sem subprocesso).
- Hooks: funções Python chamadas em pontos do loop; exemplo citado: bloquear `Bash` em `PreToolUse` com `permissionDecision: "deny"`.
- `ClaudeAgentOptions` configura `system_prompt`, `max_turns`, `allowed_tools`, `permission_mode`, `cwd`, `mcp_servers`, `hooks`.
- Por padrão o Claude tem todas as ferramentas do Claude Code (Read, Write, Edit, Bash etc.).
- Atenção: `allowed_tools` é lista de auto-aprovação, não restrição; para bloquear use `disallowed_tools`.
- Cache do system prompt: por padrão é gravado na primeira requisição; `"snapshot": False` reconstrói a cada requisição (exige Claude Code CLI 2.1.257 ou superior).
- Exceções: `CLINotFoundError`, `CLIConnectionError`, `ProcessError`, `ResultError`, `CLIJSONDecodeError`.
- Migração do Claude Code SDK (< 0.1.0): renomear `ClaudeCodeOptions` para `ClaudeAgentOptions`.

**Custo/requisitos:** Python 3.10+. O Claude Code CLI vem incluído no pacote; pode-se apontar outro binário com `cli_path`. A página não menciona API key nem outros provedores.

### 5b. TypeScript

- **URL final que funcionou:** https://github.com/anthropics/claude-agent-sdk-typescript (README)
- **O que é:** SDK em TypeScript para construir agentes autônomos programaticamente, com as capacidades do Claude Code.

**Pontos práticos:**
- Instalação: `npm install @anthropic-ai/claude-agent-sdk`.
- Documentação oficial indicada na página: docs.claude.com/en/api/agent-sdk/overview.
- Antigo Claude Code SDK foi renomeado para Claude Agent SDK; há guia de migração com breaking changes (não detalhadas no README).
- Uso regido pelos Commercial Terms of Service da Anthropic, inclusive para produtos oferecidos a clientes finais.
- Coleta de feedback (uso, dados de conversa associados, comando `/bug`); a página afirma limites de retenção e política contra treino com esse feedback.
- Bugs e pedidos vão para as issues do GitHub; há servidor no Discord.
- O README não traz exemplos de `query`, opções, ferramentas, hooks nem MCP, nem limitações técnicas.

**Custo/requisitos:** Node.js 18+ (badge da página). Não menciona API key nem outros requisitos.

---

## 6. Skills no Claude Code

- **URL final que funcionou:** https://code.claude.com/docs/en/skills
- **O que é:** uma skill é um `SKILL.md` com instruções que o Claude pode usar quando relevante ou que o usuário invoca com `/nome`.

**Pontos práticos:**
- Locais: pessoal `~/.claude/skills/<nome>/SKILL.md`; projeto `.claude/skills/<nome>/SKILL.md` (pode ser commitado). Também há níveis empresarial, plugin e conta claude.ai. Em nomes iguais, a precedência é empresarial > pessoal > projeto.
- O `SKILL.md` começa com frontmatter YAML entre `---` na primeira linha. Campos são opcionais; `description` é o recomendado, porque é ela que o Claude usa para decidir quando aplicar. Campos com nome errado são ignorados sem erro.
- Campos citados: `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `user-invocable`, `allowed-tools`, `disallowed-tools`, `disable-model-invocation`, `model`, `effort`, `context`, `agent`, `paths`, `hooks`, `shell`.
- Argumentos: `$ARGUMENTS`, posicionais (`$0`, `$ARGUMENTS[1]`) ou nomeados.
- Arquivos de apoio (`reference.md`, `examples.md`, `scripts/`): referencie no SKILL.md; mantenha o SKILL.md com menos de 500 linhas; use `${CLAUDE_SKILL_DIR}` para caminhos de scripts.
- `` !`comando` `` executa um comando antes de enviar o conteúdo ao Claude; se falhar, a invocação inteira é abortada. `disableSkillShellExecution` desliga isso para skills de usuário, projeto e plugin.
- Permissões: `allowed-tools` pré-aprova ferramentas só durante o turno que invoca a skill (não restringe o resto); `disallowed-tools` remove ferramentas enquanto a skill está ativa; regras `Skill(nome)` em `/permissions` permitem ou negam skills.
- `context: fork` roda a skill em subagente sem histórico da conversa, então precisa de instruções explícitas.
- Compactação: após compactar, cada skill volta com até 5.000 tokens, com total de 25.000 tokens reanexados. Se a skill sumir do comportamento, invoque de novo e coloque as instruções mais importantes no início.
- Listagem de descrições: o orçamento é 1% da janela de contexto (`skillListingBudgetFraction` ajusta); cada entrada tem limite de 1.536 caracteres (`skillListingMaxDescChars`). Descrições de skills menos usadas são cortadas primeiro.
- Evitar `<` e `>` na descrição (aparecem como `&lt;`); use colchetes ou `argument-hint`.
- Validação: `claude plugin validate <pasta>` (Claude Code 2.1.233 ou superior) acusa frontmatter inválido.
- Avaliação: o plugin `skill-creator` cria casos de teste em `evals/evals.json`, roda cada caso em subagente isolado, gera `grading.json` e `benchmark.json` e compara versões com teste A/B.
- Compartilhamento: `.claude/skills/` no repositório, plugins com diretório `skills/`, ou deploy gerenciado por configurações da organização.
- Limitações: skills em `~/.claude/skills/` não são carregadas no Cowork nem em sessões na nuvem, que usam as skills habilitadas na conta claude.ai. Ao enviar para claude.ai ou API, só valem `name`, `description`, `license`, `compatibility`, `metadata` e `allowed-tools`; campos extras causam erro. Com `allowManagedPermissionRulesOnly`, o `allowed-tools` de skills de projeto e pessoais é ignorado.
- Se o frontmatter for inválido, a skill carrega sem metadados: `/nome` funciona, mas o Claude não a encontra pela descrição (use `--debug` para ver o erro).

**Custo/requisitos:** a página não trata de preço. Requer Claude Code; a disponibilidade em sessões na nuvem depende da conta claude.ai.

---

## Links que falharam

Nenhum. As seis páginas (com as duas do SDK em TypeScript e Python) foram lidas. Observação: a página de skills tem cerca de 120 mil caracteres; o WebFetch leu em duas partes (0 a 100.000 e a partir de 100.000), e a leitura foi completa. A página de boas práticas foi lida integralmente a partir do arquivo de resultado salvo pela ferramenta.
