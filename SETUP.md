# Setup do Gomes — passo a passo

Este guia assume que você tem acesso ao Mac que roda o gateway e às credenciais listadas abaixo.

## Pré-requisitos

- macOS com Hermes Agent instalado
- Token Slack Bot (`SLACK_BOT_TOKEN`) do app `@Gomes`
- Token Slack App (`SLACK_APP_TOKEN`) para Socket Mode
- PostHog API key (project 351731)
- Token GitHub com scope `repo` (para Tamanduá e PR Reviewer)
- MCP Gomo token (Bearer, para `executar_sql` no banco de produção)
- Acesso à rede iFood (VPN ou office) para MCPs do gateway

## 1. Instalar skills

```bash
# Copiar todas as skills para o perfil gomo
mkdir -p ~/.hermes/profiles/gomo/skills/ifood/
cp docs/skills/*.md ~/.hermes/profiles/gomo/skills/ifood/

# Renomear de volta pra SKILL.md
cd ~/.hermes/profiles/gomo/skills/ifood/
for f in gomes-*.md; do
  dir="${f%.md}"
  mkdir -p "$dir"
  mv "$f" "$dir/SKILL.md"
done
```

## 2. Configurar variáveis de ambiente

Edite `~/.hermes/profiles/gomo/.env`:

```bash
POSTHOG_API_KEY=phx_...
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...
GITHUB_TOKEN=ghp_...
MCP_GOMO_API_KEY=90593064-8faa-46f3-94fe-706c0916a6be
GATEWAY_ALLOW_ALL_USERS=true
```

## 3. Configurar MCPs

Os MCPs do gateway iFood usam OAuth. Para autenticar:

```bash
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test databricks
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test slack
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test atlassian
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test google
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test gitlab
```

O MCP Gomo usa SSL auto-assinado. Configure manualmente no `config.yaml`:

```yaml
mcp_servers:
  gomo:
    url: https://mcp.gomoapp.com.br/mcp
    auth: header
    token: "${MCP_GOMO_API_KEY}"
    enabled: true
    transport_options:
      verify: false
```

Depois adicione o certificado ao sistema:

```bash
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain \
  <(openssl s_client -connect mcp.gomoapp.com.br:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM)
```

## 4. Configurar canais Slack

No `config.yaml`, adicione os canais de resposta livre:

```yaml
slack:
  require_mention: true
  free_response_channels: C0BKAVAV8KE,C0BKGC9P62W
  channel_prompts:
    C0BKAVAV8KE: "Você é o Gomes. Carregue a skill gomes-insights-agent antes de responder."
    C0BKGC9P62W: "Você é o Gomes, analista de produto e engenharia do Gomo. NUNCA poste como Tiago Penha."
```

## 5. Criar cron jobs

```bash
HERMES_HOME=~/.hermes/profiles/gomo

# Radar diário — 7h BRT
hermes cron create gomes-daily-radar \
  --schedule "0 10 * * *" \
  --skill gomes-insights-agent \
  --prompt "Gere o radar diário completo do Gomo e poste neste canal." \
  --deliver "slack:C0BKAVAV8KE"

# Competitive alert — 9h BRT
hermes cron create gomes-competitive-alert \
  --schedule "0 12 * * *" \
  --skill gomes-competitive-alert \
  --prompt "Verifique concorrentes e poste apenas se houver alertas relevantes." \
  --deliver "slack:C0BKAVAV8KE"

# Release notes — sexta 18h BRT
hermes cron create gomes-release-notes \
  --schedule "0 21 * * 5" \
  --skill gomes-release-notes \
  --prompt "Compile o changelog semanal e poste neste canal." \
  --deliver "slack:C0BKAVAV8KE"

# PR reviewer — 7h e 15h BRT
hermes cron create gomes-pr-reviewer \
  --schedule "0 10,18 * * *" \
  --skill gomes-pr-reviewer \
  --prompt "Revise PRs abertos sem review nos repos restu-mobile e restu-web." \
  --deliver "slack:C0BKGC9P62W"
```

## 6. Iniciar o gateway

```bash
HERMES_HOME=~/.hermes/profiles/gomo hermes gateway start
```

O gateway roda como serviço launchd e reinicia automaticamente se o Mac reiniciar.

## 7. Verificar que tudo funciona

```bash
# Listar MCPs
hermes mcp list

# Listar cron jobs
hermes cron list

# Dashboard web
HERMES_HOME=~/.hermes/profiles/gomo hermes dashboard

# Testar no Slack: enviar "@Gomes teste" no #gomo-insights
```

## Manutenção

| Problema | Comando |
|---|---|
| Token OAuth expirado (~3 semanas idle) | `HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test <nome>` |
| Gateway parou | `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway restart` |
| Logs | `~/.hermes/profiles/gomo/logs/gateway.log` |
| Dashboard não mostra crons | Conferir `HERMES_HOME` — dashboard padrão usa perfil `default` |
| Mensagens saem como "Tiago Penha" | Verificar `free_response_channels` e `channel_prompts` no config.yaml |

## Arquitetura de arquivos

```
~/.hermes/profiles/gomo/
├── config.yaml          # config principal (MCPs, gateway, Slack)
├── .env                 # tokens e secrets
├── skills/ifood/        # skills do Gomes (8 skills)
│   ├── gomes-insights-agent/SKILL.md
│   ├── gomes-bug-triage/SKILL.md
│   ├── gomes-code-fixer/SKILL.md
│   ├── gomes-pr-reviewer/SKILL.md
│   ├── gomes-release-notes/SKILL.md
│   ├── gomes-competitive-alert/SKILL.md
│   ├── gomes-deep-dive/SKILL.md
│   └── gomes-oncall/SKILL.md
├── logs/
│   └── gateway.log
└── home/
    └── .github_token    # token GitHub para API
```
