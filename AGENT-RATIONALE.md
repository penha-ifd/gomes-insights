# Gomes — Racional de Criação do Agente

> Documento de transferência. Se você está lendo isso, provavelmente o Penha saiu do projeto e você precisa entender como o Gomes funciona e como recriá-lo.

---

## 1. Contexto: Por que um agente?

O Gomo é um app social de descoberta e avaliação de restaurantes dentro do iFood. Pós-lançamento, em fase de hipercrescimento, a liderança de produto (Penha) precisava de:

- **Radar diário** com métricas de produto, execução e mercado — consolidado em um só lugar, toda manhã
- **Respostas rápidas** a perguntas de qualquer pessoa do time sobre dados do produto
- **Memória institucional** do que acontece dia a dia (decisões, métricas, feedbacks)

Fazer isso manualmente era inviável. A solução foi um agente AI autônomo.

## 2. Arquitetura

```
┌─────────────────────────────────────────────────────────────────┐
│                        Mac do Penha                              │
│                                                                  │
│  ┌──────────────────┐     ┌──────────────────┐                  │
│  │  launchd service  │────▶│  Hermes Gateway   │                 │
│  │  ai.hermes.       │     │  (perfil: gomo)    │                 │
│  │  gateway-gomo     │     │                    │                 │
│  └──────────────────┘     │  ┌──────────────┐  │                 │
│                            │  │ DeepSeek V4   │  │                 │
│                            │  │ Pro (GenPlat) │  │                 │
│                            │  └──────────────┘  │                 │
│                            │  ┌──────────────┐  │                 │
│                            │  │ Cron: 7h BRT  │  │                 │
│                            │  │ radar diário   │  │                 │
│                            │  └──────────────┘  │                 │
│                            └────────┬─────────┘                  │
│                                     │                             │
│                            Socket Mode                            │
│                                     │                             │
└─────────────────────────────────────┼─────────────────────────────┘
                                      │
                              ┌───────┴───────┐
                              │    Slack       │
                              │  iFood Global  │
                              │                │
                              │  @Gomes (bot)  │
                              │  #gomo-insights│
                              └────────────────┘
```

### Componentes

| Componente | Local | Função |
|---|---|---|
| **Hermes Agent** | `~/.hermes/` | Runtime do agente AI |
| **Perfil gomo** | `~/.hermes/profiles/gomo/` | Config, persona, skills, tokens isolados |
| **Gateway** | launchd `ai.hermes.gateway-gomo` | Conecta Slack ao agente, gerencia sessões |
| **Skill** | `skills/ifood/gomes-insights-agent/SKILL.md` | Instruções de como gerar o radar (APIs, templates, pitfalls) |
| **Cron** | `gomes-daily-radar` | Dispara o agente todo dia às 7h BRT |
| **Repo** | `github.com/penha-ifd/gomes-insights` | Radares históricos, métricas, decisões |

### Por que no Mac e não em cloud?

- O Mac do Penha já fica ligado 24/7
- Token OAuth do iFood (GenPlat) só funciona da rede corporativa ou via `tompero`
- Evita complexidade de deploy, CI/CD, secrets management
- Troca: se o Mac hibernar, o agente para. Launchd reinicia automaticamente.

## 3. Stack

| Camada | Escolha | Motivo |
|---|---|---|
| **Runtime** | Hermes Agent (CLI + Gateway) | Suporte nativo a Slack, cron, skills, MCPs |
| **Modelo** | DeepSeek V4 Pro | Melhor custo/latência via GenPlat. Claude é backup. |
| **Gateway** | Socket Mode (Slack) | Não precisa de URL pública. Comunicação via WebSocket. |
| **Auth** | OAuth (Bot Token + App Token) | Token de bot (`xoxb-...`) e app-level token (`xapp-...`) |
| **MCPs** | Atlassian, Google, Slack, Databricks | Conexão com Jira, Confluence, Sheets, SQL |
| **Cron** | Hermes cron (interno) | Não depende de cron do SO. Gerencia retry e entrega. |

## 4. Configuração

### 4.1 Perfil Hermes

```bash
hermes profile create gomo
```

Isso cria `~/.hermes/profiles/gomo/` com estrutura isolada. Toda operação do Gomes usa:

```bash
HERMES_HOME=~/.hermes/profiles/gomo hermes <comando>
```

### 4.2 Slack App

O app Slack foi criado em https://api.slack.com/apps com o manifest em `slack-app-manifest.yaml`.

**Scopes do Bot Token:**
```
channels:read, channels:history, groups:read, groups:history,
chat:write, chat:write.customize, app_mentions:read, users:read,
im:history, im:write
```

**Event Subscriptions:**
```
app_mention, message.im
```

**Configurações:**
- Socket Mode: enabled
- Messages Tab: enabled, read_only disabled
- Token Rotation: disabled
- Org Deploy: disabled (app instalado manualmente no workspace)

### 4.3 .env

Arquivo: `~/.hermes/profiles/gomo/.env`

```bash
# Tokens Slack
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...

# Permissões
SLACK_ALLOWED_USERS=
SLACK_HOME_CHANNEL=C0BKAVAV8KE
GATEWAY_ALLOW_ALL_USERS=true    ← CRÍTICO: sem isso, exige pairing code

# APIs
POSTHOG_API_KEY=phx_...
GITHUB_TOKEN=ghp_...

# GenPlat (iFood)
HERMES_GENPLAT_REQUESTER_TOKEN_COMMAND=/Users/tiago.penha/.local/bin/tompero auth requester-token get

# Provider
OPENROUTER_API_KEY=...
OPENAI_API_KEY=...
```

**⚠️ `GATEWAY_ALLOW_ALL_USERS` é lido do `.env`, NÃO do `config.yaml`.**  
Esse foi o bug mais difícil de diagnosticar. O `config.yaml` tinha `gateway.allow_all_users: true` mas o gateway só lê a env var. Resultado: usuários novos viam pairing code e o bot parecia "ignorar" pessoas.

### 4.4 Skill

A skill `gomes-insights-agent` está em:
```
~/.hermes/profiles/gomo/skills/ifood/gomes-insights-agent/SKILL.md
```

Ela contém:
- Persona do agente (tom, estilo)
- Fontes de dados com endpoints e tokens
- Templates de radar (completo e conciso)
- Framework de métricas de produto
- Método de pesquisa de mercado
- **22 pitfalls** documentados (coisas que deram errado e como resolver)

### 4.5 Cron

```bash
HERMES_HOME=~/.hermes/profiles/gomo hermes cron create \
  --name "gomes-daily-radar" \
  --schedule "0 7 * * *" \
  --skill gomes-insights-agent \
  --prompt "Gere o radar diário do Gomo" \
  --deliver slack:C0BKAVAV8KE
```

## 5. Comandos de manutenção

```bash
# Ver status
launchctl list | grep hermes-gateway-gomo

# Reiniciar gateway (após mudar .env ou token)
HERMES_HOME=~/.hermes/profiles/gomo hermes gateway restart

# Ver logs
tail -f ~/.hermes/profiles/gomo/logs/gateway.log

# Aprovar usuário manualmente
HERMES_HOME=~/.hermes/profiles/gomo hermes pairing approve slack CODIGO

# Testar conexão Slack
HERMES_HOME=~/.hermes/profiles/gomo hermes gateway status

# Listar crons
HERMES_HOME=~/.hermes/profiles/gomo hermes cron list

# Rodar cron manualmente
HERMES_HOME=~/.hermes/profiles/gomo hermes cron run JOB_ID
```

## 6. Problemas conhecidos e soluções

### Bot não responde ninguém (nem DM, nem canal)

1. Verificar se o gateway está rodando: `launchctl list | grep gomo`
2. Ver logs: `tail -50 ~/.hermes/profiles/gomo/logs/gateway.log`
3. Se aparecer "No user allowlists configured":
   - `GATEWAY_ALLOW_ALL_USERS=true` está no `.env`? Não está comentado?
   - Reiniciar gateway

### Bot responde só o Penha, ignora outros (pairing code)

**Causa:** `GATEWAY_ALLOW_ALL_USERS` não está ativo.  
**Solução:**
1. Verificar `.env`: `grep GATEWAY_ALLOW_ALL_USERS ~/.hermes/profiles/gomo/.env`
2. Deve estar `=true`, sem `#` na frente
3. `HERMES_HOME=~/.hermes/profiles/gomo hermes gateway restart`

### Erro `restricted_action_read_only_channel`

O canal Slack não permite post do bot. Resolver no Slack > canal > Settings > Permissions.

### Erro `missing_argument: team_id` (channel directory)

Warning não crítico. O gateway não consegue listar canais, mas responde normalmente. Provável falta do scope `users:read` (já adicionado).

### Token OAuth expirou

Tokens OAuth do iFood expiram ~3 semanas. Reautenticar:
```bash
hermes mcp test atlassian
hermes mcp test google
hermes mcp test slack
```
Se falhar, reautenticar via browser (o CLI abre o fluxo OAuth).

### Reinstalou o app Slack → token mudou

Atualizar `SLACK_BOT_TOKEN` no `.env` com o novo token e reiniciar gateway.

## 7. Histórico de decisões

| Data | Decisão |
|---|---|
| Jul/2026 | App unificado "Gomes" (antes eram dois: Gomo Agent + Gomito) |
| Jul/2026 | Renomeado de "Gomo Insights" para "Gomes" (SOUL.md, SKILL.md, repo) |
| Jul/2026 | `GATEWAY_ALLOW_ALL_USERS=true` no `.env` (antes dependia de pairing manual) |
| Jul/2026 | Repo movido para `gomes-insights/` (antes `gomo-insights/`) |
| Jul/2026 | Scopes adicionados: `channels:history`, `groups:history`, `users:read` |

## 8. Dependências

- **Mac do Penha ligado** (o gateway é local)
- **`tompero` CLI** para token GenPlat (requer login iFood)
- **Hermes Agent** instalado e atualizado
- **Slack app** instalado no workspace iFood Global (E017UJXRZQ8)
- **PostHog** project 351731 com API key
- **GitHub token** com acesso aos repos `mmarqueti/restu-*`

---

**Última atualização:** Julho 2026  
**Responsável original:** Tiago Penha (tiago.penha@ifood.com.br)
