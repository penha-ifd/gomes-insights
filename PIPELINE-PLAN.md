# Gomes Agentic Bug Pipeline — Plano de Implementação

> **Para:** Engineering Manager do Gomo  
> **Autor:** Tiago Penha · AI Labs  
> **Data:** Julho 2026  
> **Diagrama:** `/Users/tiago.penha/Desktop/gomes-agentic-pipeline.html`

---

## Visão Geral

Pipeline autônomo onde o **Gomes** (agente Hermes + DeepSeek V4 Pro) detecta, analisa e classifica bugs, e o **Tamanduá** (workflow engine, 23 workflows instalados) executa correções — com gates de aprovação por criticidade.

```
Bug (Jira/Slack) → Gomes analisa → Classifica criticidade → 
  ├─ Baixa: Tamanduá abre PR + auto-merge
  ├─ Média: Tamanduá abre PR + code review
  └─ Alta: Gomes reporta → EM aprova → Tamanduá age
```

---

## Arquitetura

| Camada | Ferramenta | Função |
|---|---|---|
| **Gatilho** | Slack / Jira / Cron | Bug reportado ou scan programado |
| **Cérebro** | Gomes (Hermes + DeepSeek) | Analisa, cruza dados, classifica, decide |
| **Executor** | Tamanduá (23 workflows) | Cria branch, aplica fix, CI, PR |
| **Aprovação** | Slack (DM do EM) | Gate manual para bugs críticos |
| **Dados** | PostHog, Gomo DB, GitHub, Confluence, Jira | Contexto para análise |

---

## O que já existe

- [x] Gomes rodando no Slack (perfil `gomo`, gateway launchd)
- [x] Cron diário de radar (`gomes-daily-radar`, 7h BRT)
- [x] MCPs: PostHog, GitHub, Jira, Confluence, Slack
- [x] Tamanduá instalado (`~/tamandua`, 23 workflows, `--hermes-as-harness`)
- [x] `GATEWAY_ALLOW_ALL_USERS=true` + scopes Slack completos
- [x] `cron_mode: approve` — agentes podem propor crons
- [x] Vision tool habilitada para análise de screenshots

## O que precisa ser construído

### Fase 1 — Pipeline de Análise (Gomes)

| # | O que | Como |
|---|---|---|
| 1.1 | **Skill de triagem de bugs** | Nova skill `gomes-bug-triage` que ensina o Gomes a: carregar bug do Jira, cruzar com métricas do PostHog, estimar impacto (# usuários afetados), classificar criticidade |
| 1.2 | **Template de análise** | Formato padronizado que o Gomes produz: severidade, usuários afetados, código suspeito (GitHub), regressão?, recomendação |
| 1.3 | **SOUL.md update** | Adicionar regra: "Quando alguém reportar bug, NÃO responda só com texto. Analise, classifique e ofereça criar o fix." |
| 1.4 | **Cron de scan** | Job diário que varre bugs novos no Jira (SWNPGMO) e faz triagem automática |

### Fase 2 — Handoff Gomes → Tamanduá

| # | O que | Como |
|---|---|---|
| 2.1 | **Workflow Tamanduá: `gomo-bug-fix`** | Workflow que recebe: repo, arquivo, descrição do fix. Faz: `git checkout -b fix/XXX` → aplica patch → `git push` → abre PR |
| 2.2 | **Workflow Tamanduá: `gomo-bug-report`** | Workflow que recebe análise do Gomes e posta no canal ou DM do EM com botões de aprovar/rejeitar |
| 2.3 | **Formato de handoff** | JSON/Markdown padronizado que o Gomes produz e o Tamanduá consome. Ex.: `{"repo": "restu-mobile", "file": "src/feed.ts", "fix": "...", "criticality": "medium"}` |
| 2.4 | **Integração Gomes→Tamanduá** | Gomes chama Tamanduá via `terminal` com `tamandua run gomo-bug-fix --data analysis.json`. Ou via `--hermes-as-harness` se o workflow rodar como subagente Hermes. |

### Fase 3 — Gates de Aprovação

| # | O que | Como |
|---|---|---|
| 3.1 | **Classificador de criticidade** | Regras no skill: LOW (typo, UI menor, <10 usuários), MEDIUM (bug funcional, 10-100 usuários, sem regressão), HIGH (regressão, >100 usuários, quebra fluxo core) |
| 3.2 | **Fluxo LOW** | Tamanduá aplica fix → abre PR → se CI passar, merge automático |
| 3.3 | **Fluxo MEDIUM** | Tamanduá aplica fix → abre PR → notifica canal #gomo-insights → aguarda review humano |
| 3.4 | **Fluxo HIGH** | Gomes envia análise detalhada pro EM (DM) → EM aprova → Tamanduá age |
| 3.5 | **Mecanismo de aprovação** | Gomes posta no Slack com botões/blocos interativos (precisa habilitar `interactivity` no app Slack) OU EM responde "aprova" no DM e Gomes dispara o Tamanduá |

### Fase 4 — Qualidade e Segurança

| # | O que | Como |
|---|---|---|
| 4.1 | **Validação pré-merge** | Tamanduá roda testes existentes + verifica se o fix não quebra nada (lint, typecheck, testes) |
| 4.2 | **Log de decisões** | Todo bug triado → registro em `~/gomes-insights/decisions/bugs/YYYY-MM-DD.md` com: decisão, criticidade, ação tomada |
| 4.3 | **Métricas do pipeline** | Track: bugs triados/dia, tempo médio triagem→fix, taxa de auto-merge, falsos positivos |
| 4.4 | **Rollback** | Se auto-merge quebrar algo → rollback automático (Tamanduá reverte PR) + alerta no Slack |

---

## Cronograma Sugerido

| Semana | Fase | Entregável |
|---|---|---|
| **1** | Fase 1 | Skill de triagem + template + SOUL.md + cron de scan |
| **2** | Fase 2 | Workflows Tamanduá + formato de handoff + integração |
| **3** | Fase 3 | Gates de criticidade + fluxos LOW/MEDIUM/HIGH |
| **4** | Fase 4 | Validação, logs, métricas, rollback |

---

## Riscos

| Risco | Mitigação |
|---|---|
| Token GenPlat expira | Monitorar, renovar a cada ~3 semanas |
| Tamanduá não cobre todos os cenários | Começar com fluxo LOW (simples), expandir |
| Auto-merge quebra produção | Rollback automático + alerta imediato |
| Falsos positivos na classificação | Métricas + ajuste iterativo do classificador |
| EM sobrecarregado com aprovações HIGH | Threshold alto pra HIGH (>100 usuários OU regressão) |

---

## Primeiro Passo (hoje)

Criar a skill `gomes-bug-triage` com o template de análise e instruções de classificação. É o coração do pipeline.

```bash
# Estrutura da skill
~/.hermes/profiles/gomo/skills/ifood/gomes-bug-triage/SKILL.md
```

Conteúdo mínimo:
- Como carregar bug do Jira
- Como cruzar com PostHog (usuários afetados)
- Como buscar código relacionado no GitHub
- Matriz de criticidade (LOW/MEDIUM/HIGH)
- Template de output padronizado
- Como disparar o Tamanduá

---

## Referências

- **Diagrama:** `gomes-agentic-pipeline.html` no Desktop
- **Repo:** `github.com/penha-ifd/gomes-insights`
- **Skill atual:** `~/.hermes/profiles/gomo/skills/ifood/gomes-insights-agent/SKILL.md`
- **SOUL.md:** `~/.hermes/profiles/gomo/SOUL.md`
- **Tamanduá:** `~/tamandua/` (23 workflows)
- **Gateway:** `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway *`
