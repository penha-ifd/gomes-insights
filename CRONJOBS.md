# Cron Jobs do Gomes

Jobs agendados no Hermes Agent (perfil `gomo`). Criados em 2026-07-24.

## Jobs Ativos

### gomes-daily-radar
- **ID:** `0d56fdc9feb7`
- **Skill:** `gomes-insights-agent`
- **Schedule:** `0 10 * * *` (7h BRT)
- **Frequência:** Diário
- **Canal:** #gomo-insights (C0BKAVAV8KE)
- **Faz:** Radar completo — PostHog, GitHub, Jira, mercado

### gomes-pr-reviewer
- **ID:** `431ce4d5ecde`
- **Skill:** `gomes-pr-reviewer`
- **Schedule:** `0 10,18 * * *` (7h e 15h BRT)
- **Frequência:** 2x/dia
- **Canal:** #gomes-code (C0BKGC9P62W)
- **Faz:** Revisa PRs abertos sem review, aprova/rejeita/encaminha

### gomes-competitive-alert
- **ID:** `450b1eb2154e`
- **Skill:** `gomes-competitive-alert`
- **Schedule:** `0 12 * * *` (9h BRT)
- **Frequência:** Diário
- **Canal:** #gomo-insights (C0BKAVAV8KE)
- **Faz:** Monitora concorrentes (Beli, Corner, Yelp, etc.)

### gomes-oncall
- **ID:** `8f05e9a48cf9`
- **Skill:** `gomes-oncall`
- **Schedule:** `*/30 * * * *` (a cada 30 min)
- **Frequência:** 48x/dia
- **Canal:** #gomes-code (C0BKGC9P62W) — só se anomalia
- **Faz:** Health checks (DAU, cadastros, reviews, erros, bugs)

### gomes-release-notes
- **ID:** `d6e0a40b26d5`
- **Skill:** `gomes-release-notes`
- **Schedule:** `0 21 * * 5` (sexta 18h BRT)
- **Frequência:** Semanal
- **Canal:** #gomo-insights (C0BKAVAV8KE)
- **Faz:** Changelog semanal (PRs + Jira + métricas)

## Timeline Diária (BRT)

```
00:00 ┤ oncall (silencioso)
00:30 ┤ oncall (silencioso)
01:00 ┤ oncall (silencioso)
...
06:30 ┤ oncall (silencioso)
07:00 ┤ 🔭 Radar diário ─────────── #gomo-insights
07:00 ┤ 🔍 PR Reviewer (manhã) ──── #gomes-code
07:30 ┤ oncall (silencioso)
08:00 ┤ oncall (silencioso)
08:30 ┤ oncall (silencioso)
09:00 ┤ 🛰️ Competitive Alert ────── #gomo-insights
09:30 ┤ oncall (silencioso)
...
15:00 ┤ 🔍 PR Reviewer (tarde) ──── #gomes-code
...
18:00 ┤ 📦 Release Notes (sexta) ── #gomo-insights
...
23:30 ┤ oncall (silencioso)
```

## Comandos

```bash
# Listar todos
HERMES_HOME=~/.hermes/profiles/gomo hermes cron list

# Rodar manualmente (teste)
HERMES_HOME=~/.hermes/profiles/gomo hermes cron run <job_id>

# Pausar
HERMES_HOME=~/.hermes/profiles/gomo hermes cron pause <job_id>

# Retomar
HERMES_HOME=~/.hermes/profiles/gomo hermes cron resume <job_id>

# Remover
HERMES_HOME=~/.hermes/profiles/gomo hermes cron remove <job_id>
```
