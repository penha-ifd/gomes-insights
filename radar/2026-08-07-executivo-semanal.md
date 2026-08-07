# Gomo — Report Executivo Semanal (S32 WTD + S31 fechada)

**Período:** S31 (27/jul–02/ago, fechada) + S32 WTD (03–06/ago, 4 dias)
**Elaborado:** 07/ago/2026 · Gomes (analista de produto)
**Para:** Agenda executiva · solicitado por Bruno Parizotto
**Comparativo:** S29 (pré-lançamento) · S30 (lançamento) · S31 (fechada) · S32 WTD

## Headline

| Métrica | S30 (launch) | S31 (fechada) | S32 WTD (4d) |
|---|---|---|---|
| WAU | 2.354 | 423 | 167* |
| DAU (fim de janela) | 123 (26/jul) | 65 (02/ago) | 78 (06/ago) |
| Reviews | 1.363 | 639 | 227* |
| Cadastros | 1.180 | 82 | 36* |
| WAC | 184 | 85 | 43* |
| Retenção D1 (coorte) | 9,0% (212/2.353) | 4,7% (10/213) | — |
| Retenção D7 (coorte) | 1,0% (22/2.130) | 1,4% parcial (3/213) | — |

*S32 WTD = 4 dias (03–06/ago) — distinct-user (WAU/WAC) não comparável a semana completa; contagens com ressalva.
Base de referência S29 (pré-lançamento): WAU 73 · Reviews 26 · Cadastros 6 · WAC 21.

## Insights

1. **O lançamento foi spike de distribuição, não tração — e a base real continua encolhendo.** S31 fechou em 423 WAU (−82% vs launch); S32-WTD (167 em 4 dias) projeta semana abaixo de S31. DAU oscilou entre nova mínima de 43 (05/ago) e respiros como 78 (06/ago) — sem tendência de estabilização.
2. **Retenção segue o ponto de virada.** Coorte orgânica S31: D1 = 4,7%, D7 parcial = 1,4% (fecha 09/ago). Quem chega hoje não volta — sem D7 não existe loop de crescimento.
3. **Funil vazio na entrada.** Cadastros: 1.180 (S30) → 82 (S31) → 36 WTD (S32). Sem canal de aquisição ativo, a base contrai mês a mês.
4. **Núcleo criador é o ativo real.** Reviews S31 = 47% do volume do lançamento (639/1.363), ~91/dia, foto em 73–93%. WAC 85 = 20% do WAU. SIR estável ~18–19% apesar do volume menor.
5. **Ritmo de engenharia alto e relevante:** suggestion-posts (recommend flow + swipe deck), pre-warm de ranking, security hardening v1.1.0 (17 vulns), perf feed/home <500ms. Não está faltando execução — está faltando produto que retenha.

## Diagnóstico

Três semanas pós-lançamento provaram demanda por distribuição (pico 1.771 DAU) e capacidade de criação (639 reviews/semana com qualidade), mas não provaram retenção nem aquisição sustentável. A base real (~423 WAU) está encolhendo em vez de estabilizar. Crescer antes de resolver retenção é encher balde furado. A execução de engenharia existe; o gargalo é produto: o primeiro acesso não vira hábito.

## Ações recomendadas

1. **Retenção/ativação como prioridade nº1 — medir impacto do tutorial de 1ª review** (PR #106 mergeado) em D1/D7 de coorte semanal; ativação hoje ~11%. [Owner: Product]
2. **Push engagement v2 em produção + instrumentar `push_tapped`** — canal de retorno mais direto; hoje push não é instrumentado (open rate = 0 sem dados). [Owner: Mobile]
3. **Não retomar aquisição até D1 ≥ 20% estável; enquanto isso, re-engajar os 1.180 cadastrados do lançamento que nunca voltaram** (coorte de 1.180 com D7 1%). [Owner: Growth]

## Gráficos

- `charts/s32_dau_reviews_14d.png` — DAU + Reviews 14d (base real ~65–85/dia; reviews ~91/dia)
- `charts/s32_comparativo_semanal.png` — S29 → S32 WTD (WAU, Reviews, Cadastros, WAC)
- `charts/s32_retencao_coortes.png` — D1/D7 por coorte fechada (D7 S31 parcial, fecha 09/ago)

## Fontes

PostHog (queries HogQL 07/ago, sem lag até 06/ago) · séries S29/S30 de `trends/metrics.jsonl` (radares 27/jul–07/ago).
