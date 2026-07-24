# Gomes — Agente de Insights do Gomo

Gomes é o sistema multi-agente de produto e engenharia do **Gomo** (app social de descoberta e avaliação de restaurantes do iFood). Ele roda como bot no Slack com múltiplas skills especializadas.

## Pipeline Agêntico

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

| # | Skill | O que faz | Gatilho | Canal |
|---|---|---|---|---|
| 1 | `gomes-insights-agent` | Radar diário: PostHog + GitHub + Jira + mercado | ⏰ Cron 7h BRT | #gomo-insights |
| 2 | `gomes-bug-triage` | Triagem de bugs: classificação, Jira, causa provável | On-demand (Slack) | #gomes-code |
| 3 | `gomes-code-fixer` | Tamanduá: implementa fix para bugs LOW/MEDIUM | Após triagem | #gomes-code |
| 4 | `gomes-pr-reviewer` | Revisão automática de PRs: aprova/rejeita/encaminha | ⚡ Webhook + cron 2x/dia | #gomes-code |
| 5 | `gomes-release-notes` | Changelog semanal: PRs + Jira + métricas | ⏰ Cron sex 18h | #gomo-insights |
| 6 | `gomes-competitive-alert` | Monitoramento de concorrentes (Beli, Corner, etc.) | ⏰ Cron 9h BRT | #gomo-insights |
| 7 | `gomes-deep-dive` | Análise aprofundada de métricas sob demanda | ❌ On-demand | #gomo-insights |
| 8 | `gomes-oncall` | Plantão 24/7: health checks a cada 30min | ⏰ Cron 30min | #gomes-code |

## Cron Jobs Necessários

| Cron Job | Skill | Schedule | Canal |
|---|---|---|---|
| `gomes-daily-radar` | insights-agent | `0 10 * * *` (7h BRT) | #gomo-insights |
| `gomes-competitive-alert` | competitive-alert | `0 12 * * *` (9h BRT) | #gomo-insights |
| `gomes-oncall` | oncall | `*/30 * * * *` | #gomes-code (só se anomalia) |
| `gomes-release-notes` | release-notes | `0 21 * * 5` (sex 18h BRT) | #gomo-insights |
| `gomes-pr-reviewer` | pr-reviewer | Webhook primário, cron fallback `0 10,18 * * *` | #gomes-code |

## Canais Slack

| Canal | ID | Uso |
|---|---|---|
| #gomo-insights | `C0BKAVAV8KE` | Radar diário, release notes, deep dive, competitive |
| #gomes-code | `C0BKGC9P62W` | Triagens, PRs, fixes, oncall alerts |

## Fontes de Dados

| Fonte | O que extrai | Skills |
|---|---|---|
| **PostHog** (project 351731) | DAU, WAU, MAU, retenção, reviews, funis, sessões | insights, deep-dive, oncall |
| **GitHub** (mmarqueti/restu-mobile, restu-web) | Commits, PRs, linhas de código | insights, release-notes, pr-reviewer |
| **Jira** (SWNPGMO) | Bugs, tasks, sprint progress | bug-triage, insights, release-notes, oncall |
| **Google News / Web** | Movimentos de concorrentes, tendências | insights, competitive-alert |
| **PostgreSQL** (restu-web backend) | SIR, WAC cross-table | deep-dive (quando disponível) |

## Infraestrutura

- **Motor:** Hermes Agent com perfil `gomo`
- **Modelo:** DeepSeek V4 Pro via GenPlat (iFood)
- **Gateway:** Serviço launchd (`ai.hermes.gateway-gomo`) no Mac do Penha
- **MCPs:** databricks, slack, gitlab, atlassian, google
- **Slack:** Socket Mode, app `@Gomes` no workspace iFood Global

## Estrutura do Repo

```
gomes-insights/
├── README.md              ← este arquivo
├── AGENT-RATIONALE.md     ← racional completo da criação do agente
├── PIPELINE.md            ← documentação do pipeline agêntico
├── radar/                 ← radares diários (YYYY-MM-DD.md)
├── trends/                ← séries temporais (metrics.jsonl)
├── decisions/             ← feedbacks e decisões do canal
│   └── bugs/              ← triagens de bugs
└── skills/                ← referência das skills (symlink ou cópia)
```

## Como recriar do zero

Ver `AGENT-RATIONALE.md` para o passo a passo completo.

TL;DR:
1. Instalar Hermes Agent
2. Criar perfil `gomo` (`hermes profile create gomo`)
3. Criar app Slack com os scopes do `slack-app-manifest.yaml`
4. Configurar `.env` com tokens e `GATEWAY_ALLOW_ALL_USERS=true`
5. Instalar todas as skills em `~/.hermes/profiles/gomo/skills/ifood/`
6. Criar os 5 cron jobs listados acima
7. Iniciar gateway: `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway start`

## Manutenção

- **Token OAuth expira ~3 semanas:** reautenticar com `hermes mcp test <nome>`
- **Gateway parou:** `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway restart`
- **Logs:** `~/.hermes/profiles/gomo/logs/gateway.log`
- **Mac precisa ficar ligado** (o gateway roda localmente)
- **Token GitHub precisa scope `repo`** para Tamanduá e PR Reviewer fazerem push/merge
