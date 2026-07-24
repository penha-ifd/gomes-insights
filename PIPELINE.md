# Pipeline Agêntico do Gomes

Documentação completa do sistema multi-agente do Gomo.

## Arquitetura

```
                         ┌──────────────────────┐
                         │    Slack / @menção    │
                         └──────────┬───────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
          ▼                         ▼                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ 🐛 Gomes Triage │     │ 🔭 Gomes Radar  │     │ 🔬 Gomes Deep   │
│ (bug-triage)    │     │ (insights-agent)│     │ Dive (deep-dive)│
│ Triagem de bugs │     │ Radar diário    │     │ Análise ad-hoc  │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                        │
    LOW/MEDIUM              ⏰ 7h BRT              ❌ On-demand
         │                       │
         ▼                       ▼
┌─────────────────┐     ┌─────────────────┐
│ 🐜 Tamanduá Fix │     │ 📦 Release Notes│
│ (code-fixer)    │     │ (release-notes) │
│ Implementa fix  │     │ Changelog semanal│
└────────┬────────┘     └────────┬────────┘
         │                  ⏰ Sex 18h
    PR aberto                    │
         │                       │
         ▼                       │
┌─────────────────┐              │
│ 🔍 PR Reviewer  │              │
│ (pr-reviewer)   │              │
│ Revisa + merge  │              │
└────────┬────────┘              │
    ⚡ Webhook                    │
         │                       │
         └───────────┬───────────┘
                     │
                     ▼
            ┌─────────────────┐
            │    #gomo-insights│
            │    #gomes-code   │
            └─────────────────┘

┌─────────────────┐     ┌─────────────────┐
│ 🛰️ Competitive  │     │ 🚨 OnCall       │
│ Alert           │     │ (oncall)        │
│ (competitive)   │     │ Plantão 24/7    │
│ Mercado diário  │     │ Health checks   │
└────────┬────────┘     └────────┬────────┘
    ⏰ 9h BRT               ⏰ 30min
         │                       │
         └───────────┬───────────┘
                     ▼
            ┌─────────────────┐
            │  Alertas c/      │
            │  @channel se     │
            │  CRÍTICO         │
            └─────────────────┘
```

## Skills

### 1. gomes-insights-agent (v1.5)
**Radar diário de produto e mercado.**

- ⏰ Cron: `gomes-daily-radar` — diário às 7h BRT (10:00 UTC)
- 📍 Canal: `#gomo-insights` (C0BKAVAV8KE)
- 📊 Fontes: PostHog (DAU, WAU, retenção, reviews), GitHub (commits, PRs), Jira (sprint), Web (mercado)
- 📁 Arquivo: `skills/ifood/gomes-insights-agent/SKILL.md`

### 2. gomes-bug-triage (v1.0)
**Triagem autônoma de bugs.**

- ❌ On-demand: quando alguém reporta bug no Slack
- 📍 Canal: `#gomes-code` (C0BKGC9P62W)
- 🔧 Fluxo: análise → classificação (LOW/MEDIUM/HIGH) → Jira ticket → handoff para Tamanduá
- 📁 Arquivo: `skills/ifood/gomes-bug-triage/SKILL.md`

### 3. gomes-code-fixer — Tamanduá (v1.1)
**Agente autônomo de correção de bugs.**

- ⛓️ Encadeado: disparado pela triagem do bug-triage (LOW/MEDIUM)
- 📍 Canal: `#gomes-code` (C0BKGC9P62W)
- 🔧 Fluxo: clone repo → analisar código → implementar fix → validar (lint/testes) → PR
- ⚠️ Precisa de token GitHub com scope `repo`
- 📁 Arquivo: `skills/ifood/gomes-code-fixer/SKILL.md`

### 4. gomes-pr-reviewer (v1.0)
**Revisor automático de PRs.**

- ⚡ Webhook primário + cron fallback: `gomes-pr-reviewer` — 2x/dia (7h e 15h BRT)
- 📍 Canal: `#gomes-code` (C0BKGC9P62W)
- ✅ Aprova: diff <100 linhas, lint limpo, segue padrão
- ❌ Rejeita: diff >200, console.log, altera config
- ⏸️ Encaminha: lógica complexa, auth, pagamento
- 📁 Arquivo: `skills/ifood/gomes-pr-reviewer/SKILL.md`

### 5. gomes-release-notes (v1.0)
**Changelog semanal.**

- ⏰ Cron: `gomes-release-notes` — sexta 18h BRT (21:00 UTC)
- 📍 Canal: `#gomo-insights` (C0BKAVAV8KE)
- 📦 Compila: PRs mergeados + bugs Jira concluídos + métricas da semana
- 📁 Arquivo: `skills/ifood/gomes-release-notes/SKILL.md`

### 6. gomes-competitive-alert (v1.0)
**Monitoramento de concorrentes.**

- ⏰ Cron: `gomes-competitive-alert` — diário às 9h BRT (12:00 UTC)
- 📍 Canal: `#gomo-insights` (C0BKAVAV8KE)
- 🛰️ Monitora: Beli, Corner, Yelp, Espaces, TripAdvisor
- 🔴 Alerta crítico vai para `#gomes-code` com @channel
- 📁 Arquivo: `skills/ifood/gomes-competitive-alert/SKILL.md`

### 7. gomes-deep-dive (v1.0)
**Análise aprofundada de métricas sob demanda.**

- ❌ On-demand: quando alguém pergunta "por que WAU caiu?" etc.
- 📍 Canal: `#gomo-insights` (C0BKAVAV8KE)
- 🔬 Testa hipóteses com queries HogQL, cruza com Jira e eventos externos
- 📁 Arquivo: `skills/ifood/gomes-deep-dive/SKILL.md`

### 8. gomes-oncall (v1.0)
**Plantão 24/7 de health checks.**

- ⏰ Cron: `gomes-oncall` — a cada 30 minutos
- 📍 Canal: `#gomes-code` (C0BKGC9P62W) — só se anomalia
- 🚨 Checks: DAU, cadastros, reviews, erros, bugs críticos
- 🔴 CRÍTICO → @channel. Silêncio = tudo ok
- 📁 Arquivo: `skills/ifood/gomes-oncall/SKILL.md`

## Cron Jobs

| Job ID | Nome | Skill | Schedule (UTC) | Schedule (BRT) | Próxima |
|---|---|---|---|---|---|
| `0d56fdc9feb7` | gomes-daily-radar | insights-agent | `0 10 * * *` | 7h | 2026-07-24 10:00 |
| `431ce4d5ecde` | gomes-pr-reviewer | pr-reviewer | `0 10,18 * * *` | 7h, 15h | 2026-07-24 10:00 |
| `450b1eb2154e` | gomes-competitive-alert | competitive-alert | `0 12 * * *` | 9h | 2026-07-24 12:00 |
| `8f05e9a48cf9` | gomes-oncall | oncall | `*/30 * * * *` | 30/30min | 2026-07-24 01:00 |
| `d6e0a40b26d5` | gomes-release-notes | release-notes | `0 21 * * 5` | Sexta 18h | 2026-07-24 21:00 |

### Comandos úteis

```bash
# Listar todos os cron jobs
HERMES_HOME=~/.hermes/profiles/gomo hermes cron list

# Ver status de um job específico
hermes cron status <job_id>

# Rodar um job manualmente (teste)
hermes cron run <job_id>

# Pausar/retomar
hermes cron pause <job_id>
hermes cron resume <job_id>
```

## Canais Slack

| Canal | ID | Uso | Skills |
|---|---|---|---|
| #gomo-insights | `C0BKAVAV8KE` | Radar, release, competitive, deep dive | insights, release-notes, competitive, deep-dive |
| #gomes-code | `C0BKGC9P62W` | Triagens, fixes, PRs, oncall alerts | bug-triage, code-fixer, pr-reviewer, oncall |

## Infraestrutura

| Componente | Detalhe |
|---|---|
| **Motor** | Hermes Agent com perfil `gomo` |
| **Modelo** | DeepSeek V4 Pro via GenPlat (iFood) |
| **Gateway** | Serviço launchd `ai.hermes.gateway-gomo` no macOS |
| **MCPs** | databricks, slack, gitlab, atlassian, google (5 ativos) |
| **Dashboard** | `http://localhost:9119` (`hermes dashboard`) |
| **Repo** | `github.com/penha-ifd/gomes-insights` |

## Fontes de Dados

| Fonte | Acesso | Skills |
|---|---|---|
| PostHog | API (project 351731) via `$POSTHOG_API_KEY` | insights, deep-dive, oncall |
| GitHub | API via token `~/.hermes/profiles/gomo/home/.github_token` | insights, code-fixer, pr-reviewer, release-notes |
| Jira SWNPGMO | MCP atlassian (cloudId: `21371cca-8433-4602-8c86-afa266485cce`) | bug-triage, insights, release-notes, oncall |
| Google News | `browser_navigate` → `news.google.com` | insights, competitive-alert |
| PostgreSQL | restu-web backend (não disponível via MCP) | deep-dive (limitado) |

## Setup do Zero

```bash
# 1. Instalar skills
mkdir -p ~/.hermes/profiles/gomo/skills/ifood/
cp -r skills/ifood/* ~/.hermes/profiles/gomo/skills/ifood/

# 2. Verificar MCPs
hermes mcp test databricks
hermes mcp test slack
hermes mcp test gitlab
hermes mcp test atlassian
hermes mcp test google

# 3. Criar cron jobs (via chat com Gomes ou hermes cron create)
# Jobs: gomes-daily-radar, gomes-pr-reviewer, gomes-competitive-alert,
#        gomes-oncall, gomes-release-notes

# 4. Iniciar gateway
HERMES_HOME=~/.hermes/profiles/gomo hermes gateway start

# 5. (Opcional) Dashboard para monitoramento
HERMES_HOME=~/.hermes/profiles/gomo hermes dashboard
```

## Manutenção

| Problema | Solução |
|---|---|
| Token OAuth expirado (~3 semanas idle) | `hermes mcp test <nome>` |
| Gateway parou | `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway restart` |
| Token GitHub read-only (push 403) | Gerar novo token com scope `repo` |
| Dashboard não sobe | Verificar porta 9119 (`lsof -i :9119`) |
| Logs | `~/.hermes/profiles/gomo/logs/gateway.log` |
| Mac desligou | Gateway roda local, precisa estar ligado |

## Changelog

### 2026-07-24
- Pipeline completo com 8 skills documentado
- 5 cron jobs criados e ativos
- README, PIPELINE.md atualizados
- Dashboard web em localhost:9119
- Tamanduá (code-fixer) testado com SWNPGMO-344, atualizado para v1.1
- Figma e Granola removidos dos MCPs
