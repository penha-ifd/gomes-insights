---
id: evt-2026-07-21-abertura-sementes
tipo: evento
data: 2026-07-21
titulo: Abertura para ~8k pessoas do iFood — canal de distribuição semeado
categoria: campanha
inicio: 2026-07-21T00:00:00Z
fim: null
impacto_esperado: [dau, cadastros, reviews, retencao_d1, retencao_d7, convites, wac, todas_as_metricas]
autor: agente-gomo
---

# Abertura para ~8k pessoas do iFood

Em 21/07/2026, o Gomo foi liberado para aproximadamente 8.000 pessoas do iFood com pedido explícito de divulgação. Estas pessoas **não são população de teste** — são canal de distribuição semeado.

## Impacto nas métricas

Qualquer métrica que atravesse a data de 21/07/2026 carrega esse evento como confundidor. Em particular:

- DAU saltou de ~96 para 1.336 (+1.277%) no dia da abertura
- Reviews foram de ~4/dia para 509 no pico
- Sessões/usuário caíram de 2.42 para <1 — consistente com leva de novos usuários

## Como ler os segmentos

| Segmento | Quem são | Retenção significa |
|---|---|---|
| **Semente** (~8k iFood) | Pessoas com acesso direto | Nada sobre o produto. Mede engajamento com a iniciativa |
| **1º grau** | Trazidos por uma semente | Sinal real, com viés social positivo |
| **Orgânico** | Chegaram pela loja sem indicação | O sinal mais limpo que existe |

A D1 de 8,2% da coorte de 22/07 mistura os três segmentos. Não é interpretável sem separação por origem.

## Ações pendentes

- [ ] Instrumentar atribuição de origem no cadastro (campo `source`: `semente` | `convite` | `organico`)
- [ ] Separar coorte de 22/07 por origem para calcular D1 real
- [ ] Registrar este evento como confundidor em toda métrica que cruzar 21/07
