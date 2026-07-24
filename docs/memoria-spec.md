# Base de memória agêntica — Gomo

Spec executável. Entregue este arquivo ao agente com a instrução: *"scaffold este repositório conforme a spec"*.

**Princípio estrutural:** separar `log/` (append-only, imutável, datado) de `estado/` (poucos documentos vivos, sobrescritos). O agente lê `indice.md` + `estado/` sempre; lê `log/` apenas sob demanda, por filtro de frontmatter. Sem essa separação, o custo de contexto inviabiliza a base em ~3 meses.

**Gate humano:** todo output do agente entra por Pull Request. Nada é escrito direto na main.

---

## 1. Estrutura de pastas

```
gomo-memoria/
├── AGENT.md                    # contrato operacional (o que ler, quando, como escrever)
├── indice.md                   # GERADO por script — nunca editar à mão
│
├── estado/                     # documentos vivos, sobrescritos
│   ├── produto.md              # o que é o Gomo hoje, features ativas
│   ├── metricas.md             # dicionário: definição, janela, sazonalidade, baseline
│   ├── arquitetura.md          # mapa de módulos + ownership (feature → área do código)
│   ├── glossario.md            # termos do domínio (evita o agente inventar nomes)
│   ├── guardrails.md           # o que o agente NÃO pode fazer/alterar
│   └── anti-backlog.md         # o que já foi tentado e não funcionou, com o motivo
│
├── log/                        # append-only. um arquivo = um fato datado
│   ├── decisoes/               # YYYY-MM-DD-slug.md
│   ├── postmortems/
│   ├── pesquisas/
│   ├── alertas/
│   ├── eventos/                # release, campanha, incidente, evento externo
│   └── reports/                # diários
│
├── dados/
│   ├── metricas/               # snapshots YYYY-MM-DD.json (coleta automática)
│   └── feedback/               # YYYY-Www.json (consolidado dos PRs)
│
├── schemas/                    # JSON Schema por tipo de registro
├── scripts/                    # Python (não Node.js — stack do Gomo é Python/Hermes)
├── .github/workflows/          # apenas ciclo semanal de aprendizado + CI de validação
└── README.md                   # visão geral + link para gomes-insights (runtime)
```

---

## 2. Schema de frontmatter

### Campos comuns (obrigatórios em todo arquivo de `log/`)

```yaml
---
id: dec-2026-07-24-ranking-feed        # {prefixo}-{data}-{slug}, único
tipo: decisao                          # decisao|postmortem|pesquisa|alerta|evento|report
data: 2026-07-24
titulo: Priorizar recência sobre proximidade no ranking do feed
origem: assistido                      # automatico|assistido|humano
autor: agente-gomo
entidades: [feed, ranking, restaurante]
metricas: [engajamento_d7, sessoes_dia]
confianca: media                       # alta|media|baixa
links:
  issue: null
  pr: null
  commit: null
  slack: null
---
```

`entidades` e `metricas` são as chaves de cruzamento. Se estiverem vazias ou inconsistentes, o registro não cruza com nada — por isso a validação em CI é obrigatória, não opcional.

### Campos adicionais por tipo

**decisao** — o registro que fecha o loop de aprendizado:
```yaml
contexto: string
hipotese: string                  # "se X, então Y"
alternativas_descartadas: [string]
metrica_alvo: engajamento_d7
valor_baseline: 0.34
valor_esperado: 0.40
checar_em: 2026-08-23             # dispara o job semanal
resultado: pendente               # pendente|confirmado|refutado|inconclusivo
valor_observado: null
verificado_em: null
status: ativo                     # proposto|ativo|revertido|superado
supersede: null                   # id da decisão que esta substitui
```

**postmortem**:
```yaml
sintoma: string
causa_raiz: string
correcao: string
prevencao: string
recorrencia_de: null              # id de postmortem anterior — sinal mais rico do backlog
tempo_deteccao_h: 6
```

**alerta**:
```yaml
metrica: engajamento_d7
valor: 0.28
esperado: [0.32, 0.38]
desvio_pct: -12.5
veredito_humano: pendente         # pendente|verdadeiro|falso_positivo|inconclusivo
causa_atribuida: null             # id de evento, se houver
```

**evento** — o calendário sem o qual o agente atribui causa errada a toda queda:
```yaml
categoria: release                # release|campanha|incidente|externo|sazonal
inicio: 2026-07-24T14:00:00Z
fim: null
impacto_esperado: [engajamento_d7]
```

**pesquisa**:
```yaml
fontes: [url]
hipotese: string
valida_ate: 2026-10-24            # pesquisa de mercado tem prazo de validade
```

---

## 3. Coleta automática

A coleta usa a infraestrutura que já existe — os cron jobs do Hermes no perfil `gomo`. Zero dependência nova.

### 3.1 Snapshot de métricas (já existe)

O cron `gomes-daily-radar` (7h BRT, `C0BKAVAV8KE`) já gera um radar completo com PostHog + MCP Gomo. Basta adicionar ao fluxo: após gerar o radar, salvar o snapshot em `gomo-memoria/dados/metricas/YYYY-MM-DD.json`.

Formato do snapshot:
```json
{
  "data": "2026-07-24",
  "coletado_em": "2026-07-24T10:00:00Z",
  "fontes": ["posthog", "mcp-gomo"],
  "metricas": {
    "dau": 180, "wau": 1130, "reviews_24h": 35,
    "cadastros_24h": 26, "buscas_24h": 24,
    "wac_semanal": 129, "sir_pct": 33.9,
    "usuarios_total": 1462, "reviews_publicos": 1192,
    "colecoes_publicas": 2977, "cidades_ativas": 69
  }
}
```

As fontes são PostHog (eventos de produto) e MCP Gomo (`executar_sql` no banco de produção). Nenhum endpoint novo — as duas fontes já estão configuradas e ativas.

### 3.2 Detecção de desvio e alertas (fast path, sem PR gate)

O cron `gomes-oncall` (a cada 30min, `C0BKGC9P62W`) já verifica anomalias. Ao detectar desvio:
1. Posta alerta imediato no Slack como **Gomes** (sem esperar PR)
2. Cria `log/alertas/{data}-{metrica}.md` com `veredito_humano: pendente`
3. O alerta entra direto na main — é append-only e assinado pelo agente, não precisa de gate humano para existir
4. Feedback humano chega depois, sem fricção: reação de emoji no thread do Slack (✅ verdadeiro / ❌ falso positivo / 🤷 inconclusivo)

### 3.3 Eventos (releases, campanhas, incidentes)

Releases são detectadas automaticamente pela GitHub API (tags novas nos repos `mmarqueti/restu-*`). Campanhas e incidentes entram manualmente ou via webhook.

O script `scripts/coletar-eventos.py` (Python, não Node.js) roda como parte do cron `gomes-daily-radar` e cria `log/eventos/{data}-{slug}.md`.

### 3.4 O que NÃO vai pra GitHub Actions

A coleta diária de métricas e detecção de desvios **não** usa GitHub Actions. O Hermes já tem cron jobs nativos rodando no Mac do Penha — duplicar schedule em Actions seria redundante e introduziria latência (o runner do Actions não tem acesso ao MCP Gomo).

---

## 4. Feedback humano (dois canais, zero fricção)

### Canal 1: Pull Requests (decisões, pesquisas, postmortems)

O próprio ciclo de PR já é o mecanismo. Não crie formulário.

| Desfecho do PR | Classificação | O que guardar |
|---|---|---|
| merge sem commits adicionais | `aceito` | id do registro |
| merge com commits do humano | `editado` | **o diff** — é aqui que está o aprendizado |
| closed sem merge | `rejeitado` | comentário de fechamento |

`scripts/consolidar_feedback.py` (Python, não Node.js):

```python
from github import Github
gh = Github(os.environ["GH_TOKEN"])
repo = gh.get_repo(f"{owner}/{repo_name}")

# Varre PRs fechados na última semana com label 'agente'
# Classifica: merge sem commits humanos = aceito, com commits = editado, fechado = rejeitado
# Grava em dados/feedback/{ano}-W{semana}.json
```

Depois de ~8 semanas, a taxa de `editado` por tipo de registro mostra onde o agente é fraco. Alimente isso de volta no `AGENT.md`.

### Canal 2: Reações de emoji no Slack (alertas, fast path)

Alertas não passam por PR — são postados direto no Slack como **Gomes**. O feedback humano é uma reação de emoji no thread:

| Reação | Significado | Atualiza no registro |
|---|---|---|
| ✅ | Verdadeiro | `veredito_humano: verdadeiro` |
| ❌ | Falso positivo | `veredito_humano: falso_positivo` |
| 🤷 | Inconclusivo | `veredito_humano: inconclusivo` |

O script `scripts/coletar_feedback_slack.py` lê as reações via Slack API (`reactions.get`) e atualiza o `veredito_humano` no registro correspondente. Zero digitação, zero fricção.

---

## 5. O cruzamento — job semanal

Esta é a peça que transforma histórico em aprendizado, e é a que quase nunca é construída.

`.github/workflows/ciclo-semanal.yml`

```yaml
name: ciclo-semanal
on:
  schedule: [{ cron: '0 11 * * 1' }]   # segunda, 08:00 BRT
  workflow_dispatch:

jobs:
  ciclo:
    runs-on: ubuntu-latest
    permissions: { contents: write, pull-requests: write }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -r scripts/requirements.txt
      - run: python scripts/checar_decisoes.py    # decisões com checar_em vencido
      - run: python scripts/consolidar_feedback.py
      - run: python scripts/gerar_indice.py
      - uses: peter-evans/create-pull-request@v6
        with:
          branch: ciclo/${{ github.run_id }}
          title: "ciclo semanal — veredito de decisões + feedback"
          labels: agente,tipo:ciclo
```

`scripts/checar-decisoes.js`:

1. varre `log/decisoes/` por `resultado: pendente` e `checar_em <= hoje`
2. puxa `metrica_alvo` dos snapshots em `dados/metricas/`, na janela entre `data` e `checar_em`
3. cruza com `log/eventos/` no mesmo período — se houve release ou campanha sobreposta, marca `confianca: baixa` e registra o confundidor
4. escreve `valor_observado`, `verificado_em` e um `resultado` **em rascunho**
5. abre PR. **O veredito final é humano.**

---

## 6. Trava de qualidade em CI

`.github/workflows/validar.yml`

```yaml
name: validar
on: [pull_request]
jobs:
  validar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -r scripts/requirements.txt
      - run: python scripts/validar_frontmatter.py  # falha o PR se inválido
      - run: python scripts/gerar_indice.py && git diff --exit-code indice.md
```

`validar_frontmatter.py` usa `python-frontmatter` + `jsonschema` contra `schemas/{tipo}.schema.json`. Valida também:
- `id` único no repositório inteiro
- `entidades` e `metricas` presentes no `estado/glossario.md` e `estado/metricas.md` (nada de termo inventado)
- arquivos em `log/` não podem ser modificados, só criados — append-only de verdade

---

## 7. Índice — a porta de entrada do agente

`gerar-indice.js` monta `indice.md` a partir do frontmatter de todos os registros: tabela com id, tipo, data, título, entidades, métricas, status. O agente lê **isso** primeiro e só então abre os arquivos relevantes.

Adicione um bloco no topo com o que muda o comportamento imediato:

```markdown
## Aberto agora
- Decisões pendentes de veredito: 4 (2 vencidas)
- Alertas sem rótulo humano: 7
- Postmortems recorrentes nos últimos 90d: 2
```

---

## 8. AGENT.md — contrato operacional

```markdown
# Contrato do agente Gomo

## Runtime
Este agente roda no Hermes (perfil `gomo`), com as skills em `gomes-insights-agent`, `gomes-bug-triage`, `gomes-code-fixer`, `gomes-pr-reviewer`, `gomes-oncall`, `gomes-deep-dive`, `gomes-competitive-alert`, `gomes-release-notes`. O código e os cron jobs estão no repo `penha-ifd/gomes-insights`.

Este repo (`gomo-memoria`) é a **memória**. O runtime está em `gomes-insights`.

## Antes de qualquer resposta
1. Ler `indice.md` (sempre).
2. Ler os documentos de `estado/` relevantes ao tema.
3. Só então abrir arquivos de `log/`, filtrando por entidade ou métrica.

## Ao propor algo novo
- Checar `estado/anti-backlog.md`. Se já foi tentado, dizer isso em vez de propor de novo.
- Checar `estado/guardrails.md`.
- Toda decisão nasce com `hipotese`, `metrica_alvo` e `checar_em`. Sem os três, não é decisão — é opinião, e não entra no log.

## Ao escrever
- Nunca escrever direto na main. Sempre PR com label `agente` e `tipo:{tipo}`.
- Nunca preencher `veredito_humano` nem `resultado` como definitivo.
- Nunca editar arquivo existente em `log/`. Para corrigir, criar novo registro com `supersede`.
- Frontmatter completo, sempre. PR sem schema válido é rejeitado no CI.

## Ao analisar número
- Cruzar com `log/eventos/` antes de atribuir causa. Se houver release ou campanha sobreposta, declarar o confundidor explicitamente.
```

---

## 9. Ordem de implementação

Comece por **duas** fontes automáticas, não por oito.

| Fase | O que | Fecha qual loop | Onde roda |
|---|---|---|---|
| 1 | `estado/metricas.md` + snapshot diário no `gomes-daily-radar` + `validar.yml` | baseline confiável | Hermes + GitHub Actions (CI) |
| 2 | decisões com `checar_em` + `checar_decisoes.py` | **decisão → resultado** (o loop principal) | GitHub Actions (semanal) |
| 3 | `consolidar_feedback.py` + `coletar_feedback_slack.py` | qualidade do próprio agente | GitHub Actions (semanal) |
| 4 | eventos, alertas, postmortems | atribuição de causa | Hermes (diário) + Slack (feedback) |
| 5 | anti-backlog, glossário, arquitetura | precisão semântica | manual inicial, depois agente |

As fases 1 e 2 sozinhas já entregam a maior parte do valor. O resto é refinamento.

## 10. Coexistência com `gomes-insights`

Dois repos, papéis diferentes:

| Repo | Papel | O que contém |
|---|---|---|
| `penha-ifd/gomes-insights` | **Runtime** | Skills, cron jobs, dashboards, templates de radar, config do Hermes |
| `penha-ifd/gomo-memoria` | **Memória** | Estado vivo do produto, log de decisões, métricas históricas, feedback |

**Integração:**
- O cron `gomes-daily-radar` (definido em `gomes-insights`) escreve snapshots em `gomo-memoria/dados/metricas/`
- O `AGENT.md` em `gomo-memoria` referencia as skills em `gomes-insights`
- O GitHub Actions de CI em `gomo-memoria` valida PRs criados pelo agente
- O agente lê `gomo-memoria/estado/` antes de responder no Slack
- Alertas nascem no Slack (Hermes), são registrados em `gomo-memoria/log/alertas/`, e o feedback volta via reação de emoji

Nada é duplicado. Cada repo tem um dono claro.
