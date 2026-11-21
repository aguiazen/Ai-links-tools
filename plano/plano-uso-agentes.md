# Plano de uso — links sobre agentes Claude (post de 21/11/2026)

Planejamento (Opus) para decisão do Felipe, em 21/11/2026. Fontes: `leituras/grupo-A-docs-1-6.md` (A), `leituras/grupo-B-docs-7-12.md` (B), `leituras/grupo-C-cursos.md` (C) e os repositórios citados em cada linha. Hooks, subagentes e memória foram lidos só em parte (B §7–9), e a página do curso no Skilljar deu 403 (B §10).

---

## 1. Veredito do post

- **Útil:** os 12 links de documentação oficial (1–12). Quase tudo já está em uso na FELK. O que falta aplicar está em poucos itens: harness de longa duração (A §4), hook de verificação no `Stop` (A §2), GitHub Action (B §12), Agent SDK (A §5) e avaliação de skills (A §6).
- **Marketing:** a frase do Eric Schmidt não foi verificada, e os 15 cursos são links de afiliado Impact para o Coursera (`links-agentes-claude.md`). Só 4 cursos são da Anthropic (16, 17, 18, 26; C). Os outros são de terceiros, e vários têm certificado pago.
- **Conclusão:** o post não traz nada que mude a estratégia. Ele serve como checklist para conferir o que já temos e achar quatro ou cinco lacunas baratas.

---

## 2. Já temos

| Recurso dos links | Onde já usamos | Maturidade |
|---|---|---|
| CLAUDE.md por projeto (A §1–2, B §9) | `felk-ferramentas/CLAUDE.md` (77 linhas), `Felk-Autopilot/CLAUDE.md` (35), `.claude/` do `Business-AI-Felktech` | Alta. Abaixo do limite de 200 linhas, mas com contradições (ver §3, item L3) |
| Skills `.claude/skills/<nome>/SKILL.md` (A §6) | 14 skills em `felk-ferramentas/.claude/skills/` (proposta, qsms, book, agente-email-propostas…), 2 no Autopilot (`propostas`, `secretaria`), `/edital` no Business-AI | Alta. Todas com menos de 130 linhas, bem abaixo do teto de 500 |
| Subagentes `.claude/agents/` (B §8) | `felk-elaborador-proposta` + `felk-verificador-proposta` + `felk-arquiteto-api-rdo` (felk-ferramentas); `verificador-edital` + `auditor-final` (Business-AI) | Alta no desenho. `tools` restrito em todos; `model` só no `auditor-final` |
| Padrão evaluator-optimizer (A §3) | `/agente-email-propostas`: elaborador → verificador, até 3 ciclos (`SKILL.md`, Etapa D); `verificar_tr.py` + `verificador-edital` no Business-AI | Alta. Teste de regressão real em `felk-ferramentas/docs/teste-regressao-1170.md` |
| Hooks PreToolUse para regra sem exceção (A §2, B §7) | `guarda-autopilot.mjs` (e-mail, pagamento, Drive) e `bloquear-comando-perigoso.mjs` em `felk-ferramentas/.claude/settings.json`; `Felk-Autopilot/hooks/` com teste `node --test` | Alta. **Mas são duas versões diferentes do guarda** (ver L1) |
| Hooks de ciclo de vida (B §7) | Radar: `radar-hook.mjs` em SessionStart/UserPromptSubmit/PostToolUse/Stop/SessionEnd, fail-open, sem gravar o texto do prompt | Alta |
| Tarefas agendadas na nuvem (A §1) | `Felk-Autopilot/rotinas/propostas.md` (07:30) e `rotinas/secretaria-ceo.md` (06:47), com prompt versionado e id | Média. Funciona, mas a "memória" da secretaria depende do contexto da sessão persistente (ver L2) |
| MCP próprio (A §1) | `felk-mcp`: 27 consultas, perfis, auditoria, 260+ testes (`felk-ferramentas/docs/felk-mcp/projeto.md`) | Alta no código. Ainda não publicado (faltam credenciais, a API v1 do RDO e o deploy) |
| API direta com teto de gasto (A §3) | `felk-intel/app/services/claude_agent.py` com `MAX_RESEARCH_RUNS_PER_DAY` (`felk-intel/README.md`) | Média-alta |
| CI com testes (A §2: "dar um check") | `felk-ferramentas/.github/workflows/` (felk-mcp, ferramentas-testes); `Business-AI-Felktech/.github/workflows/testes.yml` (lições, mutações, hash das verificações) | Alta. O CI do Business-AI é o mais rigoroso do grupo |
| Plugin / manifesto (A §6) | `Felk-Autopilot/.claude-plugin/` + `hooks/hooks.json` | Média |
| Memória entre sessões (B §9) | Auto memory + Cérebro FELK no Obsidian (`/vault`) + Radar (`/checkpoint`) | Alta, até redundante |

Resumo: a FELK já pratica o núcleo do que os links ensinam (skills, subagentes, hooks, rotinas, MCP, avaliador independente). Não há recurso básico faltando.

---

## 3. Lacunas e oportunidades

Ordem de prioridade: ganho alto com custo baixo primeiro. Esforço: P = horas, M = dias, G = semanas.

| # | O que é (fonte) | Onde aplicar | Ganho esperado | Esforço | Custo | Risco |
|---|---|---|---|---|---|---|
| **L1** | **Uma fonte só para o guarda.** Regra sem exceção precisa ser uma regra só (A §2) | `felk-ferramentas/.claude/hooks/guarda-autopilot.mjs` sempre bloqueia o envio. `Felk-Autopilot/hooks/guarda-autopilot.mjs` libera com `APROVADO FELK-AA-NNNN`. A rotina de propostas roda na sessão de `felk-ferramentas` (`rotinas/propostas.md`), onde vale a versão estrita, então o fluxo de aprovação do Autopilot não funciona lá | Fim da divergência silenciosa. Mudança no guarda passa a valer nos dois repositórios | P | Nenhum | Baixo. O risco está em continuar como está |
| **L2** | **Harness de rotina longa:** estado em arquivo, não no contexto. Compactar o contexto não basta, e o agente declara "pronto" sem verificar (A §4) | `rotinas/secretaria-ceo.md` pede para "aproveitar o que já sabe das rodadas anteriores", o que depende da memória da conversa. A secretaria passa a gravar um JSON de estado (alertas dados, rascunhos criados, data). `monitorar-caixa.mjs` já marca as mensagens processadas e serve de modelo | A rotina para de repetir alerta e de esquecer o que fez depois de compactar o contexto | P | Nenhum | Baixo. Falta definir onde o JSON mora (container, VPS ou Radar) |
| **L3** | **Auditoria do CLAUDE.md:** `/doctor prompt-audit` acha instruções contraditórias e não altera nada (B §9). `.claude/rules/` com `paths:` carrega regra só quando o trabalho é naquela pasta (A §2) | `felk-ferramentas/CLAUDE.md`. Contradições já visíveis: numeração `FELK-AA-NNNN-001` × `FELK-<número>-<ano>`; "export via Word COM" × "na nuvem o PDF sai pelo LibreOffice". A seção Felk Tech poderia ir para uma regra com `paths: Felk Tech/**` | Menos instrução ignorada e menos tokens por sessão | P | Poucos tokens | Baixo. As correções passam pelo Felipe |
| **L4** | **Hook de verificação:** `Stop` com `exit 2` impede o fim do turno enquanto o teste falha. Pedir evidência, não "os testes passam" (A §2) | `Felk-Autopilot`: a regra 6 do CLAUDE.md ("Mudou o guarda? Rode `node --test`…") vira hook. Mesmo padrão para `felk-ferramentas/.claude/hooks/*.test.mjs` | Uma regra que hoje é só instrução passa a ser garantida | P | Nenhum. Hook `command` não chama modelo (B §7) | Baixo. Hook lento atrasa o fim do turno; usar só quando arquivos de hook mudaram |
| **L5** | **Skills com efeito colateral só por comando:** `disable-model-invocation: true` (A §2, A §6). Hoje nenhuma skill FELK usa | `/registro-proposta` (grava CSV), `/book` e `/qsms` (gravam no Drive), `/pos-venda` | O Claude não dispara sozinho uma baixa de proposta ou uma gravação no Drive | P | Nenhum | Baixo. Exige digitar `/nome`, que já é o uso normal |
| **L6** | **Padrão de projeto multi-sessão:** arquivo de progresso + lista de funcionalidades em JSON (itens começam reprovados, teste não se apaga) + rotina de início (git log, progresso, teste de ponta a ponta) (A §4) | Projetos de programação longos: `felk-mcp` (hoje usa `handover-2026-11-17.md` avulso), API v1 do `rdo-simples` e app do edital. Complementa o `/checkpoint`: o Radar fica com o estado para o Felipe, o repositório com o estado para o agente | Menos retrabalho entre sessões e menos "declarou pronto sem testar". Alinha com `docs/seguranca-ia/checklist-engenharia-sistemas.md` (prova executada) | P para criar o modelo, depois rotina | Nenhum | Baixo. Risco de virar mais um documento sem manutenção; mitigar com teste no CI que confere o JSON |
| **L7** | **Avaliação de skill:** o `skill-creator` gera `evals/evals.json`, roda cada caso em subagente isolado e compara versões A/B (A §6) | `/proposta` e `/agente-email-propostas`. O caso FELK-26-1170 de `teste-regressao-1170.md` vira o primeiro eval; somar 3 a 5 propostas reais anonimizadas (`tools/anonimizar.py`) | Mudança na skill passa a ter nota antes/depois, em vez de teste manual | M | Tokens de subagente a cada rodada de eval. Rodar só quando a skill muda | Médio. Dados de cliente nos casos exigem anonimização antes |
| **L8** | **Claude Code GitHub Action:** revisão de PR por `@claude` ou por prompt fixo no runner do GitHub (B §12) | `aguiazen/rdo-simples` (SaaS em produção, multi-tenant). Primeira passada automática em todo PR, focada em isolamento entre tenants e migrations. **Não substitui** a verificação independente exigida no checklist de engenharia | Bug de isolamento ou migration pego antes do `deploy.sh` | P (instalar por `/install-github-app`) | **API key da Anthropic, cobrada por uso.** A página não diz que a assinatura Max cobre a action (B §12). Precisa de admin do repositório | Médio. Custo variável por PR e chave guardada no GitHub. Começar com gatilho manual (`@claude`), não em todo push |
| **L9** | **Modelo por subagente:** `model: haiku` em tarefa simples; `/usage` mostra onde o consumo foi (A §2, B §8) | Tarefas mecânicas: cópias do `/book`, conferências de formato. O verificador de proposta continua no modelo forte | Menos cota do Max gasta. Regra "custo primeiro" do CLAUDE.md | P | Economia | Médio. Modelo menor erra mais; medir com o eval do L7 antes de trocar |
| **L10** | **Agent SDK para produto:** `ClaudeSDKClient` com ferramentas próprias (`@tool` + `create_sdk_mcp_server`), hooks em código e `disallowed_tools`. Atenção: `allowed_tools` só auto-aprova, não restringe (A §5a). Uso regido pelos Commercial Terms, inclusive para produto vendido a cliente final (A §5b) | Camada de LLM do agente de TR (`Business-AI-Felktech/agente/README.md`, etapa 3): hoje roda no "Claude Code Max", que não serve para produto vendido. O SDK chama `verificar_tr.py`/`verificar_etp.py` como ferramentas | É o caminho técnico para transformar o agente interno em produto da Felk Tech | G | **API key e cobrança por token**, a embutir no preço por prefeitura | Alto. Depende de decisões abertas (ver §4): parecer jurídico, sigilo e SoberanIA |
| **L11** | **Consolidar o fluxo de propostas** (A §3: comece pelo mais simples) | Hoje há dois fluxos: `/agente-email-propostas` (felk-ferramentas, com verificador) e a skill `propostas` (Autopilot, só com elaborador em `.claude/agents/`). `rotinas/propostas.md` diz que apontar a sessão para o Autopilot é decisão pendente do Felipe | Um procedimento, uma rotina, um guarda | M | Nenhum | Médio. Mexe na rotina que está em produção; fazer em janela com execução manual conferida |

**O que não vale agora:**
- **Ferramentas do awesome-claude-code** (B §11): sons, abas de tmux, notificações. Não resolvem problema da FELK. Também não instalar skills de terceiros: o próprio plano do Autopilot registra o caso de skills maliciosas no marketplace do OpenClaw (`Felk-Autopilot/docs/plano-felk-autopilot.md` §2.3).
- **`agentcairn`** (memória com Obsidian, B §11): já temos `/vault` e o Cérebro FELK.
- **Hooks `prompt`/`agent`** (B §7): chamam modelo a cada evento e têm custo. Os hooks `command` atuais bastam.
- **`/batch` e worktrees em paralelo** (A §2): úteis em migração grande de código, que não está no horizonte. Reavaliar no `rdo-simples` se aparecer refatoração ampla.
- **Claude Managed Agents** (plano do Autopilot §3): é anterior a este post e não aparece nas leituras. Fica fora deste plano.

---

## 4. Tese "empresa agêntica" aplicada à Felk Tech

A frase do post ("construa um agente melhor que o dos outros e seja dono da empresa") é chamariz. O critério útil vem da própria Anthropic: os casos mais promissores são aqueles em que **o sucesso é verificável** (A §3).

**Critérios de escolha, em ordem:**
1. **Verificável:** existe uma régua objetiva de certo e errado (lei, checklist, contrato assinado)?
2. **Dado próprio:** a Felk Tech tem acervo ou experiência de campo que um concorrente genérico não tem?
3. **Canal:** já existe cliente ou porta aberta?
4. **Margem:** o custo de API por cliente cabe no preço?
5. **Risco:** jurídico, de sigilo e de dependência de terceiro.

| Candidato | 1. Verificável | 2. Dado próprio | 3. Canal | 4. Margem | 5. Risco | Veredito |
|---|---|---|---|---|---|---|
| **A. Agente de ETP/TR para prefeituras** (já existe: `Business-AI-Felktech/agente/`, skill `/edital`) | Alto. Alíneas do art. 6º, XXIII, verificadores com teste de mutação, gabarito contra o contrato do PNCP (`agente/README.md`) | Alto. Corpus PNCP + experiência de execução da FELK (`estrategia/03_modelo_de_negocio.md`) | Médio. SMICT Niterói candidata, Firjan/Conecta Lab ainda sem confirmação (`README.md`, alertas) | Mensalidade por município. Custo por token "a embutir" (03) | **Alto.** Parecer do advogado sobre conflito de interesses (arts. 9º e 14) é condição antes de qualquer venda. Sigilo ainda em decisão (`produto/sigilo-dos-dados.md`). Acesso de empresa privada à API da SoberanIA não confirmado | **Nº 1, já em curso.** Não abrir frente nova antes de resolver o risco jurídico. O Agent SDK (L10) só entra depois do parecer e da escolha do modelo |
| **B. Agente de RDO dentro do RDO Simples** (hipótese: transformar anotação, foto e áudio do encarregado em RDO e resumo para o cliente) | Médio. Dá para conferir contra o efetivo e as fotos, mas "bom relatório" é subjetivo | Alto. A FELK usa o RDO Simples como cliente founder | **Alto.** SaaS em produção com clientes; vira módulo pago, não empresa nova | Viável como adicional por tenant | Médio. Dado de cliente passa pelo provedor do modelo e o isolamento multi-tenant precisa ser provado. **Não verifiquei o que o RDO Simples já tem de IA**: o repositório `rdo-simples` não está nesta sessão | **Nº 2.** Melhor canal e menor risco jurídico. Próximo passo é uma especificação (só leitura do repositório), não código |
| **C. Agente de proposta técnica para empreiteiras** (vender o fluxo `/agente-email-propostas`) | Alto. Checklist e verificador já existem | Baixo para o cliente. O valor está no banco de composições e nos preços da FELK, que são sigilosos e não se vendem | Baixo. Empreiteiras também são concorrentes e clientes da FELK | Incerta | Médio-alto. Cada empresa tem template e custos próprios, ou seja, customização pesada. O README do agente já decidiu: "a proposta técnica de licitante ficou como ferramenta interna da FELK, fora do produto" (`Business-AI-Felktech/agente/README.md`) | **Não vale.** Mantém como vantagem interna da FELK |

**Também não vale:** transformar o `felk-intel` (prospecção de vencedores no PNCP) em produto. É ferramenta comercial da própria FELK para buscar subcontratação (`felk-intel/README.md`), e vendê-la arma o concorrente.

**Leitura crítica:** a Felk Tech já é, na prática, uma "empresa agêntica" pequena: um SaaS no ar e um agente vertical em piloto. O risco não é faltar ideia de agente. É abrir terceira e quarta frentes antes de fechar A (jurídico e piloto) e B (especificação).

---

## 5. Cursos

Preferir sempre o endereço direto da Anthropic ou do Coursera, **nunca o link de afiliado do post**.

| Curso | Para quem | Por quê |
|---|---|---|
| **26. Claude Code in Action** (Anthropic, 1 h) | Felipe | Cobre exatamente o que ele opera: plan mode, `/compact`, rewind, hooks, cloud routines, headless, GitHub Action e plugin (C §26). A Anthropic Academy oferece de graça, segundo a página de visão geral (A §1). A página do Skilljar deu 403 nesta leitura, então confira no navegador |
| **16. Building with the Claude API** (Anthropic, grátis, sem certificado) | Felipe, **só** os módulos de tool use, avaliação e fluxos agênticos (cerca de 4 h de 13) | Base para decidir o L10 (Agent SDK, custo por token, eval) com conhecimento próprio, sem depender só do que o Claude explica (C §16) |
| **13. AI Fundamentals with Claude** (AWIT + Anthropic, 4 h, opcional) | Equipe administrativa, **só se** ela passar a usar o Claude no dia a dia | Prompt, contexto e Artifacts para quem não programa (C §13). Sem uso real, não vale as 4 h |

**Descartados:**
- **14, 15, 19:** o 19 é a soma de 13, 14 e 15, com certificado pago (C, "duplicados").
- **17 (MCP Advanced):** o `felk-mcp` já está construído. Transportes e sampling são detalhe de implementação, que o Claude resolve.
- **18 (Teaching AI Fluency):** feito para docentes.
- **20, 25, 27:** do mesmo instrutor terceiro (SkillsBooster), sobre temas que já dominamos (marketing, setup do Claude Code, Cowork).
- **21:** depende de Rust/pmat, fora do nosso stack.
- **22:** voltado a engenheiro de software e exige assinatura paga.
- **23:** pago, com 24 h de conteúdo que repete o 16.
- **24:** ensina OpenAI Agents SDK, que foge do Claude (C §24).

---

## 6. Plano em fases

### 0–30 dias: higiene e regras em código (sem custo novo)

| Ação | Responsável | Entregável | Pronto quando |
|---|---|---|---|
| Decidir a versão canônica do guarda (estrita ou com `APROVADO`) | Felipe | Decisão registrada no Radar (`/checkpoint`) | Decisão dada |
| Unificar o guarda nos dois repositórios, com teste (L1) | Claude | PR em `Felk-Autopilot` e em `felk-ferramentas` | `node --test hooks/guarda-autopilot.test.mjs propostas/*.test.mjs` passando nos dois, com a saída colada no PR |
| Hook `Stop`/`PostToolUse` que roda os testes do guarda quando um `hooks/*.mjs` muda (L4) | Claude | Entrada em `.claude/settings.json` + teste | Mudança proposital que quebra o guarda impede o fim do turno (prova executada) |
| `/doctor prompt-audit` nos CLAUDE.md + lista de contradições (L3) | Claude roda, Felipe aprova | PR com as correções aprovadas | As duas contradições citadas em L3 resolvidas, sem linha nova sem dono |
| `disable-model-invocation: true` nas skills com efeito colateral (L5) | Claude, com a lista confirmada por Felipe | PR em `felk-ferramentas` | `claude plugin validate` sem erro; as skills só disparam por `/nome` |
| Estado da secretaria em JSON (L2) | Claude; Felipe escolhe onde o arquivo mora | Ajuste em `.claude/skills/secretaria/SKILL.md` e `rotinas/secretaria-ceo.md` | Duas execuções seguidas sem alerta repetido nem rascunho duplicado |
| Fazer o curso 26 | Felipe | Curso concluído | 1 h feita |

### 31–60 dias: medir e revisar automaticamente

| Ação | Responsável | Entregável | Pronto quando |
|---|---|---|---|
| Modelo de projeto multi-sessão (progresso + `funcionalidades.json` + rotina de início) aplicado ao `felk-mcp` (L6) | Claude | Arquivos no repositório + verificação no CI | Uma sessão nova retoma o `felk-mcp` lendo só esses arquivos, e o CI reprova o JSON se um teste sumir |
| Evals do `/proposta` e do `/agente-email-propostas` com `skill-creator` (L7) | Claude | `evals/evals.json` com o caso 1170 + 3 casos anonimizados | `benchmark.json` gerado; a próxima mudança de skill é comparada A/B |
| Piloto da GitHub Action no `rdo-simples`, só por `@claude` (L8) | Felipe cria a API key e define o teto; Claude configura | Workflow em PR | 30 dias de uso com custo medido e ao menos um achado real ou a decisão de desligar |
| Trocar o modelo de subagentes mecânicos, medindo com o eval (L9) | Claude | Campo `model` nos agentes escolhidos | Nota do eval sem queda e consumo menor em `/usage` |
| Consolidar o fluxo de propostas em um só (L11) | Felipe decide o repositório; Claude migra | Rotina única + `rotinas/propostas.md` atualizado | Uma execução manual conferida antes de religar a agenda |

### 61–90 dias: produto Felk Tech (só com o jurídico resolvido)

| Ação | Responsável | Entregável | Pronto quando |
|---|---|---|---|
| Parecer do advogado sobre conflito de interesses no agente de TR | Felipe | Parecer | Recebido. **Sem ele, nada desta fase anda** |
| Escolher a camada de modelo do agente de TR (Claude via API ou Soberano 1) | Felipe | Decisão | Dada, com o custo por TR estimado |
| Protótipo com Agent SDK: `ClaudeSDKClient` + verificadores como `@tool` + hook `PreToolUse` + `disallowed_tools` (L10) | Claude | Protótipo em `Business-AI-Felktech/agente/` com testes no CI atual | Um caso de `agente/exemplos/` gerado pelo SDK passa em `verificar_tr.py` e no `verificador-edital`, com custo em tokens medido |
| Especificação do agente de RDO (candidato B), só leitura do `rdo-simples` | Claude | `spec` em `Business-AI-Felktech/produto/` | Diz o que o RDO Simples já tem, o que falta, o custo por tenant e o teste de isolamento necessário |

---

## 7. Decisões para o Felipe

1. **Qual guarda vale nos dois repositórios: o estrito (nunca envia) ou o que libera com `APROVADO FELK-AA-NNNN`?** Recomendo **o que libera com a frase**, como versão única. É o desenho mais recente e tem teste. Só que hoje ele não vale na sessão que roda a rotina de propostas.
2. **Liberar uma API key da Anthropic, com teto mensal, para a GitHub Action no `rdo-simples`?** Recomendo **sim, como piloto de 30 dias, só por `@claude`** e com teto baixo definido por você. A assinatura Max não cobre a action (B §12).
3. **Onde mora o estado das rotinas (JSON da secretaria): no container da sessão, na VPS ou no Radar?** Recomendo **o Radar**, que já existe, é externo à sessão e você já consulta.
4. **Consolidar o fluxo de propostas em qual repositório?** Recomendo **`Felk-Autopilot`**, levando para lá o verificador que hoje só existe em `felk-ferramentas`. O CLAUDE.md do Autopilot já o define como casa dos agentes.
5. **Agente de RDO (candidato B): autoriza uma especificação de leitura no `rdo-simples` agora, em paralelo ao TR?** Recomendo **sim, só a especificação**. É barata, e o canal de venda já existe. Código, só depois.
6. **Curso:** recomendo **você fazer só o 26 (1 h)** agora e o 16 parcial antes da decisão do Agent SDK. Equipe, só se ela passar a usar o Claude.
7. **Proposta técnica para empreiteiras como produto?** Recomendo **não**. Continua como vantagem interna da FELK, como o próprio `Business-AI-Felktech/agente/README.md` já registra.

## 8. Decisões do Felipe (21/11/2026)

| # | Decisão | Execução |
|---|---------|----------|
| 1 | Guarda único: o que libera com `APROVADO FELK-AA-NNNN` | Feito, reforçado: a frase precisa estar numa mensagem digitada pelo Felipe (o hook lê o histórico da sessão). Hook de testes + CI. PRs Felk-Autopilot #12 e felk-ferramentas #65 |
| 2 | API key para GitHub Action no rdo-simples | **Não aprovada** — fica fora |
| 3 | Estado das rotinas no Radar | Feito: rota `/api/hook/estado/:rotina` (radar-felk #8) + Secretaria lê/grava. Depende de deploy |
| 4 | Fluxo de propostas consolidado no Felk-Autopilot | Feito no repositório (verificador + normas + Etapa 5b). Corte da tarefa agendada pendente (`rotinas/propostas.md`) |
| 5 | Spec do agente de RDO (Opus, esforço extra) | `Business-AI-Felktech/produto/spec-agente-rdo.md` |
| 6 | Cursos: só o 26 (Claude Code in Action, grátis na Anthropic Academy) | Registrado. Demais descartados |
| 7 | Proposta técnica para empreiteiras não vira produto | Registrado. Fica como vantagem interna da FELK |
