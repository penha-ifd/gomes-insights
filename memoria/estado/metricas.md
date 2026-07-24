---
id: met-2026-07-24
tipo: estado
data: 2026-07-24
titulo: Dicionário de métricas do Gomo
autor: agente-gomo
---

# Dicionário de Métricas

## DAU (Daily Active Users)
- **Definição:** Usuários únicos que abriram o app no dia (evento `Application Became Active`)
- **Janela:** 1 dia (`yesterday()` para dado confiável; `today()` é parcial)
- **Fonte:** PostHog (project 351731)
- **Baseline:** ~180-330 (pós-pico de 21/07); valor anterior de ~3.300 pode estar inflado pelo spike
- **Sazonalidade:** Fins de semana tendem a ser mais baixos
- **Target KR:** 500 (a definir)

## WAU (Weekly Active Users)
- **Definição:** Usuários únicos em 7 dias
- **Janela:** 7 dias rolantes
- **Fonte:** PostHog
- **Baseline:** ~1.130

## Reviews Submetidos
- **Definição:** Contagem de eventos `review_submitted`
- **Janela:** 1 dia ou 7 dias
- **Fonte:** PostHog (eventos) + MCP Gomo (`user_reviews` para total acumulado)
- **Baseline:** ~35/dia

## Novos Cadastros
- **Definição:** Contagem de eventos `user_registered`
- **Janela:** 1 dia
- **Fonte:** PostHog + MCP Gomo (`users.created_at`)
- **Baseline:** 26-97/dia (alta variância)

## WAC (Weekly Active Contributors)
- **Definição:** Usuários únicos que publicaram review, foto ou comentário na semana
- **Eventos:** `review_submitted`, `review_flow_photo_added`, `comment_posted`
- **Janela:** 7 dias
- **Fonte:** PostHog
- **Baseline:** 129

## SIR (Social Interaction Rate)
- **Definição:** Reviews que receberam ≥1 interação / total de reviews
- **Fonte:** PostgreSQL (`review_likes`, `review_comments`) — PostHog tem limitação (pode >100%)
- **Baseline:** 33.9%

## Retenção D1
- **Definição:** Usuários que voltaram no dia seguinte ao primeiro acesso
- **Janela:** Coorte semanal
- **Fonte:** PostHog (LEFT JOIN `Application Became Active` D+1)
- **Baseline:** 8.2% (coorte 22/07)

## Convites
- **Definição:** Contagem de convites enviados
- **Fonte:** PostgreSQL (`user_invites`)
- **Tracking:** Sem evento no PostHog — apenas backend
