# Gomes — Agente de Insights do Gomo

Gomes é o agente AI de produto e mercado do **Gomo** (app social de descoberta e avaliação de restaurantes do iFood). Ele roda como bot no Slack, publica um radar diário e responde perguntas sob demanda.

## O que ele faz

- **Radar diário** às 7h BRT no canal `#gomo-insights` com métricas de produto, execução e mercado
- **Responde perguntas** de qualquer pessoa no canal ou DM com dados concretos
- **Coleta feedback** de usuários e registra no repo
- **Monitora concorrentes** (Beli, Corner, Yelp, TripAdvisor, Espaces)

## Fontes de dados

| Fonte | O que extrai |
|---|---|
| **PostHog** (project 351731) | DAU, WAU, MAU, retenção, reviews, funis, sessões |
| **GitHub** (mmarqueti/restu-mobile, restu-web) | Commits, PRs, linhas de código, issues |
| **Jira** (SWNPGMO) | Sprint progress, bugs, velocity |
| **Google News / Web** | Movimentos de concorrentes, tendências de mercado |

## Como funciona

```
┌──────────────┐     ┌─────────────────┐     ┌──────────┐
│  Slack DM /   │────▶│  Hermes Gateway  │────▶│  DeepSeek │
│  @menção      │     │  (launchd, Mac)  │     │  V4 Pro   │
└──────────────┘     └─────────────────┘     └──────────┘
                            │
                    ┌───────┴───────┐
                    │  Ferramentas   │
                    │  PostHog/GitHub│
                    │  Jira/Web      │
                    └───────────────┘
```

- **Motor:** Hermes Agent com perfil `gomo`
- **Modelo:** DeepSeek V4 Pro via GenPlat (iFood)
- **Gateway:** Serviço launchd (`ai.hermes.gateway-gomo`) rodando no Mac do Penha
- **Cron:** `gomes-daily-radar` dispara diariamente às 7h BRT
- **Slack:** Socket Mode, app `@Gomes` no workspace iFood Global

## Estrutura do repo

```
gomes-insights/
├── README.md              ← este arquivo
├── AGENT-RATIONALE.md     ← racional completo da criação do agente
├── radar/                 ← radares diários (YYYY-MM-DD.md)
├── trends/                ← séries temporais (metrics.jsonl)
└── decisions/             ← feedbacks e decisões do canal
```

## Como recriar o agente do zero

Ver `AGENT-RATIONALE.md` para o passo a passo completo.

TL;DR:
1. Instalar Hermes Agent
2. Criar perfil `gomo` (`hermes profile create gomo`)
3. Criar app Slack com os scopes do `slack-app-manifest.yaml`
4. Configurar `.env` com tokens e `GATEWAY_ALLOW_ALL_USERS=true`
5. Instalar skill `gomes-insights-agent`
6. Criar cron `gomes-daily-radar`
7. Iniciar gateway: `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway start`

## Manutenção

- **Token OAuth expira ~3 semanas:** reautenticar com `hermes mcp test slack`
- **Gateway parou:** `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway restart`
- **Logs:** `~/.hermes/profiles/gomo/logs/gateway.log`
- **Mac precisa ficar ligado** (o gateway roda localmente)
