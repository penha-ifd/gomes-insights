# Gomo — Report Executivo Semanal (S31)

**Período:** 27/jul – 02/ago/2026 (encerrado ontem)
**Elaborado:** 03/ago/2026 · Gomes (analista de produto)
**Para:** C-level · solicitado por Bruno Parizotto
**Comparativo:** S29 (pré-lançamento) · S30 (lançamento) · S31 (fechada)

---

## Headline (semana fechada)

| Métrica | S30 (lançamento) | S31 (27/jul–02/ago) | Delta |
|---|---|---|---|
| WAU | 2.354 | 423 | −82% |
| DAU (fim de semana) | 123 (26/jul) | 65 (02/ago) | −47% |
| Reviews | 1.363 | 639 | −53% |
| Cadastros | 1.180 | 82 | −93% |
| WAC | 184 | 85 | −54% |
| Retenção D1 (coorte) | 9,0% (n=212) | 4,7% (n=10/213) | ↓ |
| Retenção D7 (coorte) | 1,0% (n=22) | 0,0% (n=0/213) | ↓ |

Base de referência S29 (pré-lançamento): WAU 73 · Reviews 26 · Cadastros 6 · WAC 21.

## Insights

1. **O lançamento foi um spike de distribuição, não tração.** WAU caiu 82% na primeira semana completa pós-lançamento. A base real do produto é ~423 WAU / ~85 DAU — é nesse patamar que devemos operar.
2. **Retenção é o ponto de virada.** Coorte S31 (a primeira sem o efeito distribuição): D1 = 4,7%, D7 = 0%. Quem chega hoje não volta. Com D7=0 não existe loop de crescimento.
3. **O funil está vazio na entrada.** Cadastros caíram de 1.180 → 82/semana. Sem canal de aquisição ativo, não há volume para a retenção corrigir.
4. **Criação de conteúdo é o ativo saudável.** Reviews pós-lançamento = 47% do volume do lançamento (639 vs 1.363), ~120/dia firmes, com foto em 78–93%. Quem fica, cria — e cria com qualidade.
5. **Concentração saudável de criadores:** WAC 85 = 20% do WAU 423. Base pequena, mas engajada.

## Diagnóstico

Lançamento provou demanda por distribuição e capacidade de gerar conteúdo. Não provou retenção: a coorte orgânica/pós-evento não volta (D7=0%). Crescer antes de resolver retenção é encher balde furado — cada real de aquisição vira WAU de uma semana.

## Ações recomendadas

1. **Ataque à retenção como prioridade nº1** — diagnóstico de queda pós-1º acesso, ativação (1ª review em 7d, hoje ~11%) e re-engajamento. [Owner: Product]
2. **Push engagement v2 em produção** — canal de retorno mais direto; push não instrumentado hoje. [Owner: Mobile]
3. **Só retomar aquisição paga/orgânica após D1 ≥ 20% estável em coorte semanal.** [Owner: Growth]

## Gráficos

- `charts/s31_dau_reviews_14d.png` — DAU + Reviews 14d (pico 1.772 → base ~85; reviews firmes)
- `charts/s31_comparativo_semanal.png` — S29 vs S30 vs S31 (WAU, Reviews, Cadastros, WAC)
- `charts/s31_retencao_coortes.png` — D1/D7 por coorte fechada

## Fontes

PostHog (queries HogQL 03/ago, sem lag) · séries S29/S30 de `trends/metrics.jsonl` (radares 27/jul–03/ago).
