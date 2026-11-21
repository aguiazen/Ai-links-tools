# Leituras — Grupo B

Resumo dos links 7 a 12, lidos com WebFetch em 21/11/2026. Só o que está nas páginas; onde a leitura foi parcial, isso está dito.

---

## 7. Hooks do Claude Code

- **URL final:** https://code.claude.com/docs/en/hooks (funcionou; a página tem cerca de 294 mil caracteres e a leitura cobriu os primeiros 100 mil, até a seção de controle de decisão)
- **O que é:** referência dos eventos de hook, da configuração em settings.json e do formato de entrada e saída.

**Pontos práticos:**
- Eventos principais: `SessionStart`/`SessionEnd`; `UserPromptSubmit`, `Stop`, `StopFailure`; por ferramenta, `PreToolUse`, `PermissionRequest`, `PermissionDenied`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`. Também há `SubagentStart`/`SubagentStop`, `TaskCreated`/`TaskCompleted`, `InstructionsLoaded`, `PreCompact`/`PostCompact`, `ConfigChange`, `CwdChanged`, `FileChanged` e outros.
- Estrutura: evento → matcher group → handler(s). Tipos de handler: `command`, `http`, `mcp_tool`, `prompt`, `agent`.
- Campos comuns: `type`, `if` (regra tipo `Bash(git *)`, só em eventos de ferramenta), `timeout` (padrão 600 s para command/http/mcp_tool, 30 s para prompt, 60 s para agent), `statusMessage`, `once` (só em frontmatter de skill).
- Campos de `command`: `command`, `args` (exec form, sem shell), `async`, `asyncRewake`, `shell`, `onFailure` (`continue` ou `block`).
- Matchers: `*`, vazio ou omitido casam tudo; nomes com letras, dígitos, `_`, `-`, `|` e `,` são nome exato ou lista (ex.: `Edit|Write`); outros caracteres viram regex JavaScript (ex.: `mcp__memory__.*`).
- Saída: exit code `0` aplica o JSON da saída; `2` é erro bloqueante (stderr vira motivo); outro código não bloqueia. Em `PreToolUse`, bloquear é feito com `hookSpecificOutput.permissionDecision: "deny"` (`allow`, `deny`, `ask`, `defer`). A página avisa que `continue: false` sozinho não bloqueia uma chamada de `PreToolUse`.
- Locais de configuração: `~/.claude/settings.json`, `.claude/settings.json`, `.claude/settings.local.json`, política gerenciada, `hooks/hooks.json` de plugins e frontmatter de skills e subagentes. Usar `${CLAUDE_PROJECT_DIR}` nos caminhos. `/hooks` mostra os hooks configurados (somente leitura).
- Limitações citadas: saída de contexto limitada a 10.000 caracteres; `if` aceita uma regra só (sem `&&` nem `||`); não há como desativar um hook individual sem removê-lo; `disableAllHooks` não desativa hooks gerenciados; em Windows, exec form exige `.exe` (`.cmd` e `.bat` pedem shell form); sessões na nuvem não leem o `~/.claude/settings.json` local.

**Custo/requisitos:** a página não informa preço. Hooks `prompt` e `agent` chamam modelo (o `agent` pode usar Read, Grep e Glob), então têm custo, sem valor descrito. Hooks `command` e `http` não mencionam custo de modelo. Requisitos de versão: `onFailure` exige Claude Code v2.1.295 ou posterior; `mcp_tool`, v2.1.328 ou posterior; matcher de Bash e Monitor, v2.1.314 ou posterior. Exemplos em Bash que leem JSON usam `jq`.

---

## 8. Subagentes

- **URL final:** https://code.claude.com/docs/en/sub-agents (funcionou; leitura cobriu os primeiros 100 mil de 115 mil caracteres, até a parte de retomada de subagentes, que ficou de fora)
- **O que é:** assistentes especializados, cada um com janela de contexto própria, system prompt, ferramentas restritas e permissões independentes; devolve só o resumo à conversa principal.

**Pontos práticos:**
- Arquivo Markdown com frontmatter YAML em `.claude/agents/` (projeto) ou `~/.claude/agents/` (usuário). Campos obrigatórios: `name` e `description` (a descrição é o que dispara a delegação; incluir "use proactively" incentiva uso automático).
- Outros campos: `tools` (allowlist), `disallowedTools` (denylist), `model` (`sonnet`, `opus`, `haiku`, `fable`, ID completo ou `inherit`), `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory` (`user`, `project` ou `local`), `effort`, `isolation: worktree`, `background`, `omitClaudeMd`.
- Prioridade em caso de nome igual: managed settings > flag `--agents` > `.claude/agents/` > `~/.claude/agents/` > plugins.
- Invocação: linguagem natural ("use o subagente X..."), @-mention, `claude --agent <nome>` (sessão inteira como subagente) ou `--agents '<json>'` (só na sessão).
- Controle: `tools` e `disallowedTools`; em MCP, `mcp__servidor__*` concede o servidor inteiro; para bloquear subagentes, `permissions.deny` com `Agent(Nome)`. Recomendação de segurança: subagente só de leitura com `tools: Read, Grep, Glob`.
- Integrados: Explore (busca somente leitura), Plan (pesquisa em plan mode), general-purpose, e outros com modelos fixos (ex.: claude-code-guide usa Haiku).
- Limitações: contexto começa isolado (não vê o histórico, salvo fork); `hooks`, `mcpServers` e `permissionMode` são ignorados em subagentes de plugins; limite de 20 subagentes concorrentes (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`); profundidade de aninhamento padrão 3; nomes devem ser únicos; descrições acima de 15.000 tokens geram aviso.
- Comandos úteis: `/agents`, `/tasks` (lista em execução, modelo e esforço), `/btw`, `claude plugin validate <pasta>`, `claude --debug`.

**Custo/requisitos:** subagentes gastam tokens próprios enquanto rodam e consomem os mesmos limites de uso da conversa principal. Para reduzir custo, a página sugere `haiku`. Vários recursos exigem versões específicas (ex.: `Task` renomeado para `Agent` na v2.1.63; `effort` por invocação na v2.1.292).

---

## 9. Memória (CLAUDE.md e auto memory)

- **URL final:** https://code.claude.com/docs/en/memory (funcionou; a página tem cerca de 54 KB e a leitura cobriu só as primeiras 400 linhas do texto, até a seção de `.claude/rules/` e regras de caminho)
- **O que é:** como dar instruções persistentes ao Claude por CLAUDE.md e como a auto memory acumula aprendizados.

**Pontos práticos:**
- CLAUDE.md é escrito pelo time; auto memory é escrita pelo Claude. Claude trata ambos como contexto, não como configuração obrigatória. Para bloquear uma ação de fato, a página indica hook `PreToolUse`.
- Localizações, na ordem de carga: managed policy (`/etc/claude-code/CLAUDE.md` no Linux); `~/.claude/CLAUDE.md`; `./CLAUDE.md` ou `./.claude/CLAUDE.md` (projeto, vai para o git); `./CLAUDE.local.md` (pessoal, colocar no `.gitignore`).
- Arquivos do diretório atual e dos acima dele são carregados no início; os de subdiretórios carregam sob demanda, quando o Claude mexe naquela pasta.
- Import com `@caminho/arquivo`, até 4 saltos; caminho com espaço usa barra invertida; trecho entre crases não é importado; comentários HTML de bloco são removidos antes de entrar no contexto.
- Regras em `.claude/rules/*.md`, com frontmatter `paths` para carregar só ao mexer em arquivos que casam com o padrão (ex.: `src/api/**/*.ts`). Regras pessoais em `~/.claude/rules/`.
- `/init` gera um CLAUDE.md inicial; `/doctor prompt-audit` procura instruções desatualizadas ou contraditórias e propõe edições sem alterar nada sozinho; `/context all` confirma quais arquivos carregaram.
- Recomendação: menos de 200 linhas por arquivo; instruções verificáveis ("rode `npm test` antes de commitar"); procedimentos longos vão para skills.
- Auto memory: `MEMORY.md` por repositório; as primeiras 200 linhas ou 25 KB são carregadas nas sessões com auto memory ligada.
- `AGENTS.md` também é lido quando não há CLAUDE.md no caminho (versão v2.1.277 ou posterior).

**Custo/requisitos:** a página não cita preço. Requisitos: arquivo de texto no repositório ou na home; `/doctor prompt-audit` exige v2.1.283 ou posterior e usa a skill `/claude-api` embutida.

---

## 10. Claude Code in Action (curso Skilljar)

- **URL final:** falhou: HTTP 403 em https://anthropic.skilljar.com/claude-code-in-action e na variação com barra final https://anthropic.skilljar.com/claude-code-in-action/. O servidor não devolveu o conteúdo.
- **O que é:** não foi possível ler.
- **Pontos práticos:** nenhum, por falta de acesso.
- **Custo/requisitos:** não verificado. Para ver o curso, abrir o link no navegador com a conta.

---

## 11. awesome-claude-code (lista da comunidade)

- **URL final:** https://github.com/hesreallyhim/awesome-claude-code (a página HTML veio sem a lista; o conteúdo foi lido em https://raw.githubusercontent.com/hesreallyhim/awesome-claude-code/refs/heads/main/README.md, após redirecionamento do link `raw` do GitHub. A leitura cobriu até cerca de 100 mil de 242 mil caracteres.)
- **O que é:** lista curada de recursos para Claude Code, com 24 categorias (de Skills a Observabilidade, Memória e Orquestração de agentes).

**Ferramentas relevantes (paráfrase das descrições da página):**
1. **Agent Skills**: https://github.com/anthropics/skills. Repositório oficial da Anthropic, com o formato SKILL.md e exemplos.
2. **cc-thinking-skills**: https://github.com/tjboudreaux/cc-thinking-skills. Skills de frameworks de pensamento, com um meta-roteador.
3. **Steering Claude Code: Skills, Hooks, Rules, Subagents and More**: https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more. Texto da Anthropic para escolher entre skill, hook, regra e subagente.
4. **Claude Code Hooks: Complete Guide**: https://hidekazu-konishi.com/entry/claude_code_hooks_complete_guide.html. Todos os eventos de hook e exemplos de settings.json.
5. **Claudio**: https://github.com/ctoth/claudio. Sons do sistema via hooks.
6. **claude-code-wsl2-setup**: https://github.com/congmnguyen/claude-code-wsl2-setup. Notificações no Windows pelos hooks Stop e PermissionRequest, no WSL2.
7. **tmux-claude-status-tabs**: https://github.com/LiveNL/tmux-claude-status-tabs. Estado de cada sessão na barra do tmux, por hooks.
8. **How We Built Our Multi-Agent Research System**: https://www.anthropic.com/engineering/multi-agent-research-system. Coordenação entre orquestrador e subagentes.
9. **Claude Code System Prompts**: https://github.com/Piebald-AI/claude-code-system-prompts. Prompts dos subagentes Plan, Explore e Task.
10. **Learn Claude Code**: https://github.com/shareAI-lab/learn-claude-code. Agente com skills, subagentes e lista de tarefas em algumas centenas de linhas de Python.
11. **Effective Context Engineering for AI Agents**: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents. Compactação, recuperação sob demanda e anotações.
12. **Writing a Good CLAUDE.md**: https://www.humanlayer.dev/blog/writing-a-good-claude-md. Orçamento de instruções e divulgação progressiva.
13. **agentcairn**: https://github.com/ccf/agentcairn. Memória de longo prazo entre projetos, com cofre do Obsidian como fonte.
14. **cxpak**: https://github.com/Barnett-Studios/cxpak. Pacotes de contexto anotados dentro de um orçamento de tokens.

**Agendamento:** o trecho lido não trouxe ferramenta de agendamento. As subcategorias Ralph Wiggum e Dynamic Workflows (em Orquestração de agentes) aparecem no índice, mas o conteúdo ficou fora da leitura.

**Custo/requisitos:** a lista é gratuita de consultar; cada ferramenta tem seus próprios requisitos, que não foram verificados.

---

## 12. claude-code-action (GitHub Action)

- **URL final:** https://github.com/anthropics/claude-code-action (funcionou)
- **O que é:** action de uso geral do Claude Code para PRs e issues do GitHub. Responde perguntas sobre o código e pode implementar alterações. Roda no runner do GitHub do próprio repositório.

**Pontos práticos:**
- Modo detectado sozinho pelo contexto: menção `@claude` em comentário ou review de PR/issue, atribuição de issue, ou automação com prompt explícito.
- Inputs unificados: `prompt` e `claude_args`.
- Saídas estruturadas (JSON validado) viram outputs da action.
- Indicadores de progresso com checkboxes que se atualizam.
- Ferramentas de GitHub e de arquivo; mais podem ser ligadas por configuração.
- Instalação mais rápida indicada: rodar `claude` no terminal e depois `/install-github-app`.
- Exemplo de YAML não está na página (remete a `docs/usage.md`); os nomes de eventos de workflow também não aparecem.
- Limitações e modo experimental: remetidos a outras páginas, não lidas.

**Autenticação:**
- Métodos citados: API direta da Anthropic (**API key** ou **workload identity federation**), Amazon Bedrock, Google Vertex AI e Microsoft Foundry.
- O quickstart automático vale só para a API direta da Anthropic; para nuvens, ver `docs/cloud-providers.md`.
- A página **não menciona assinatura nem OAuth**, e não traz os nomes exatos dos secrets (ver `docs/setup.md`).

**O que precisa no repositório:**
- Ser **admin do repositório** para instalar o GitHub App e cadastrar secrets.
- Credenciais do provedor escolhido configuradas.

**Custo/requisitos:** a página não informa preço. Como usa API da Anthropic por chave ou por nuvem, o custo é de uso de API, e a página não diz que assinatura cubra a action.

---

## Links que falharam

- https://anthropic.skilljar.com/claude-code-in-action (HTTP 403)
- https://anthropic.skilljar.com/claude-code-in-action/ (HTTP 403, variação)
