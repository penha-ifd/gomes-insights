# Handover do Gomes — Guia de Transferência

Este guia assume que você está entregando o Gomes para outra pessoa operar. Siga na ordem.

## Pré-requisitos para o novo operador

- [ ] Mac com Hermes Agent instalado
- [ ] Acesso à rede iFood (VPN ou office — necessário para MCPs do gateway)
- [ ] Conta Slack no workspace iFood Global
- [ ] Acesso ao repo `penha-ifd/gomes-insights` (GitHub)

## O que transferir

### 1. Repositório (já está pronto)

```bash
git clone git@github.com:penha-ifd/gomes-insights.git
```

Toda a documentação está em:
- `README.md` — visão geral
- `SETUP.md` — passo a passo de instalação
- `INDEX.md` — índice completo
- `docs/skills/` — 9 skills em markdown
- `docs/memoria-spec.md` — spec da base de memória
- `memoria/` — base de memória povoada

### 2. Tokens e secrets (transfira por canal seguro)

| Secret | Onde está hoje | Como transferir |
|---|---|---|
| `SLACK_BOT_TOKEN` | `~/.hermes/profiles/gomo/.env` | Copie o valor. É o token OAuth do app @Gomes no Slack |
| `SLACK_APP_TOKEN` | `~/.hermes/profiles/gomo/.env` | Token de Socket Mode (começa com `xapp-`) |
| `POSTHOG_API_KEY` | `~/.hermes/profiles/gomo/.env` | Project API key do PostHog project 351731 |
| `MCP_GOMO_API_KEY` | `~/.hermes/profiles/gomo/.env` | Bearer token do MCP Gomo |
| GitHub token (scope `repo`) | `~/.hermes/profiles/gomo/home/.github_token` | Precisa de scope `repo` para Tamanduá e PR Reviewer |
| MCP OAuth tokens | `~/.hermes/mcp-tokens/` | **Não copie arquivos.** Re-autentique com `hermes mcp test <nome>` na nova máquina |

### 3. Configuração do Hermes

```bash
# 1. Criar perfil gomo na nova máquina
hermes profile create gomo

# 2. Copiar config.yaml
# Transfira ~/.hermes/profiles/gomo/config.yaml para a nova máquina
# (substituindo o gerado automaticamente)

# 3. Criar .env com os tokens
# ~/.hermes/profiles/gomo/.env
POSTHOG_API_KEY=phx_...
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...
MCP_GOMO_API_KEY=...
GATEWAY_ALLOW_ALL_USERS=true

# 4. Instalar skills
mkdir -p ~/.hermes/profiles/gomo/skills/ifood/
cp gomes-insights/docs/skills/*.md ~/.hermes/profiles/gomo/skills/ifood/
# Renomear e criar diretórios (ver SETUP.md)

# 5. Autenticar MCPs (UM por vez)
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test databricks
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test slack
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test atlassian
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test google
HERMES_HOME=~/.hermes/profiles/gomo hermes mcp test gitlab

# 6. Configurar MCP Gomo (certificado auto-assinado)
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain \
  <(openssl s_client -connect mcp.gomoapp.com.br:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM)
```

### 4. Cron jobs

```bash
HERMES_HOME=~/.hermes/profiles/gomo

# Listar jobs existentes para referência
hermes cron list

# Recriar na nova máquina (ver CRONJOBS.md para a lista completa):
hermes cron create gomes-daily-radar --schedule "0 10 * * *" --skill gomes-insights-agent --prompt "..." --deliver "slack:C0BKAVAV8KE"
# ... (repetir para cada job)
```

### 5. Gateway (substitui o launchd do Penha)

```bash
# Iniciar
HERMES_HOME=~/.hermes/profiles/gomo hermes gateway start

# Verificar
HERMES_HOME=~/.hermes/profiles/gomo hermes gateway status

# Dashboard
HERMES_HOME=~/.hermes/profiles/gomo hermes dashboard
```

### 6. Verificação final

- [ ] `hermes mcp list` mostra 6 MCPs ativos
- [ ] `hermes cron list` mostra os jobs scheduled
- [ ] Mensagem "@Gomes teste" no #gomo-insights recebe resposta
- [ ] Dashboard em localhost:9119 mostra crons e MCPs
- [ ] Radar diário aparece no canal às 7h BRT

## O que NÃO transfere automaticamente

| Item | Por que | Ação |
|---|---|---|
| MCP OAuth tokens | Vinculados à máquina que autenticou | Re-autenticar com `hermes mcp test` |
| Certificado SSL do MCP Gomo | Keychain do macOS é local | Re-adicionar com `security add-trusted-cert` |
| Gateway como serviço launchd | Plist é local | Criar novo `~/Library/LaunchAgents/ai.hermes.gateway-gomo.plist` |

## Contatos para suporte

| Pessoa | Responsabilidade |
|---|---|
| Tiago Penha | Criador do Gomes, owner do repo e da spec |
| Marcelo Marchetti | Admin do MCP Gomo (token, Railway) |
| Bruno Parizotto | Produto, cron jobs de performance |
| Vitor Batista | Repos restu-mobile e restu-web |
