# Índice — Repositório gomes-insights

## Documentação

| Arquivo | Conteúdo |
|---|---|
| `README.md` | Visão geral do sistema multi-agente, pipeline, skills, fontes de dados |
| `SETUP.md` | Guia passo a passo para instalar do zero |
| `PIPELINE.md` | Arquitetura completa do pipeline agêntico (8 skills, fluxos, canais) |
| `CRONJOBS.md` | Cron jobs ativos, schedule, timeline diária, comandos |
| `AGENT-RATIONALE.md` | Racional da criação do agente (contexto de produto) |
| `docs/memoria-spec.md` | Spec da base de memória agêntica (`gomo-memoria`) |

## Skills

| Arquivo | Skill | O que faz |
|---|---|---|
| `docs/skills/gomes-insights-agent.md` | `gomes-insights-agent` | Radar diário: PostHog + GitHub + Jira + MCP Gomo + mercado |
| `docs/skills/gomes-bug-triage.md` | `gomes-bug-triage` | Triagem de bugs: classificação, Jira, handoff |
| `docs/skills/gomes-code-fixer.md` | `gomes-code-fixer` | Tamanduá: implementa fix para bugs LOW/MEDIUM |
| `docs/skills/gomes-pr-reviewer.md` | `gomes-pr-reviewer` | Revisão automática de PRs: aprova/rejeita/encaminha |
| `docs/skills/gomes-release-notes.md` | `gomes-release-notes` | Changelog semanal: PRs + Jira + métricas |
| `docs/skills/gomes-competitive-alert.md` | `gomes-competitive-alert` | Monitoramento de concorrentes (Beli, Corner, etc.) |
| `docs/skills/gomes-deep-dive.md` | `gomes-deep-dive` | Análise aprofundada de métricas sob demanda |
| `docs/skills/gomes-oncall.md` | `gomes-oncall` | Plantão 24/7: health checks a cada 30min |

## Canais Slack

| Canal | ID | Uso |
|---|---|---|
| `#gomo-insights` | `C0BKAVAV8KE` | Radar diário, release notes, deep dive, competitive |
| `#gomes-code` | `C0BKGC9P62W` | Triagens, PRs, fixes, oncall alerts |

## Cron Jobs

| Job | Skill | Schedule (BRT) | Canal |
|---|---|---|---|
| `gomes-daily-radar` | insights-agent | 7h | #gomo-insights |
| `bruno-daily-performance` | insights-agent | 9h | DM Bruno |
| `Gomo Radar Diário` | insights-agent | 12h (seg-sex) | C0BJV7Y1KDF |
| `gomes-competitive-alert` | competitive-alert | 9h | #gomo-insights |
| `gomes-pr-reviewer` | pr-reviewer | 7h, 15h | #gomes-code |
| `gomes-release-notes` | release-notes | Sex 18h | #gomo-insights |

## Fontes de Dados

| Fonte | Acesso | Skills |
|---|---|---|
| PostHog | API key (`$POSTHOG_API_KEY`) | insights, deep-dive, oncall |
| MCP Gomo | Bearer token (`$MCP_GOMO_API_KEY`) | insights, deep-dive, oncall |
| GitHub | Token (`~/.hermes/profiles/gomo/home/.github_token`) | insights, code-fixer, pr-reviewer, release-notes |
| Jira SWNPGMO | MCP atlassian (OAuth) | bug-triage, insights, release-notes |
| Google News | Browser navigate | insights, competitive-alert |

## Infraestrutura

- **Motor:** Hermes Agent com perfil `gomo`
- **Gateway:** launchd `ai.hermes.gateway-gomo` no macOS
- **MCPs ativos:** databricks, slack, gitlab, atlassian, google, gomo
- **Dashboard:** `http://localhost:9119` (`hermes dashboard`)
- **Repo irmão:** `gomo-memoria` (memória agêntica — spec em `docs/memoria-spec.md`)
