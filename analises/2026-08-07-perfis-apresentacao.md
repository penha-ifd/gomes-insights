# Perfis de Usuário Gomo — Material de Apresentação
Data: 2026-08-07 · Autor: Gomes · Solicitante: Mairon Carvalho
Base: análise comportamental PostHog 30d (05/jul–03/ago) para classificação, 7d (28/jul–03/ago) para medição. WAU = 385 (inclui device-only). Ground truth cadastro: tabela `users` (1.604).

## Mapa dos 4 perfis (1 slide)

| Perfil | WAU | % | Acessos/sem (mediana) | Duração sessão | Com conta | Job-to-be-done |
|---|---|---|---|---|---|---|
| **Criador** (ex-Influencer) | 116 | 30% | **5** (p90: 58) | ~38s | 95% | Produzir conteúdo que alimenta a comunidade |
| **Interator** (ex-Sociável) | 18 | 5% | 3 | ~20s | 100% | Validar/amplificar conteúdo alheio |
| **Navegador** (ex-Explorador) | 49 | 13% | 2 | ~27–41s | 100% | Decidir onde comer |
| **Espectador** | 202 | 53% | 2 | ~15–20s | 37% | (sem job claro — não ativou) |

## Slide por perfil

### 1. Criador — 116 WAU (30%) — o motor de conteúdo
**Comportamento no app:** cria review (1.712 em 30d, mediana 3/usuário), lista e comenta; 36% também reage — criação e interação andam juntas. Sessão mais longa (~38s) e mais frequente (5/sem, p90 58).
**O que defende o perfil:** top 5 = 40% das reviews, top 20 = 77%. Sem eles não há feed. O backend só marca 2 usuários como `influenciador` — a base comportamental é 58× maior que a flag manual.
**Risco:** dependência de ~20 pessoas para a saúde do feed.
**Onde melhorar:** programa de creators (flag já existe no backend) para escalar de 2 → 20+; reconhecimento/gamificação para quem cria.

### 2. Interator — 18 WAU (5%) — o estágio inicial
**Comportamento no app:** reage (50% do grupo só reage), curte, segue; comenta pouco (9 comentários em 7d no app inteiro). 3 acessos/sem, sessão ~20s.
**O que defende o perfil:** 100% com conta; está a 1 passo da criação — 36% dos Criadores passaram por aqui.
**Leitura:** é estágio de transição, não persona durável (por isso o merge faz sentido — mas vale apresentar separado para mostrar o funil).
**Onde melhorar:** transformar reação em milestone de ativação — nudge pós-reação ("viu algo bom? conta pra gente"), CTA para primeira review.

### 3. Navegador — 49 WAU (13%) — o público-alvo do core value
**Comportamento no app:** puro discovery — 103 views de restaurante, 35 buscas, 15 rankings, 16 coleções vistas, 13 mapas em 30d. Zero criação, zero interação. 2 acessos/sem, mas sessão de leitura longa (27–41s) — é o perfil que mais lê por visita.
**O que defende o perfil:** 100% cadastrados (intenção real de uso); job é exatamente "onde vou comer" — a promessa central do Gomo.
**Onde melhorar:** é o perfil que valida o produto, mas é raso (2,1 views/usuário/30d) — medir conversão leitura→decisão (salvar em coleção, chamar rota); testar feed de lugares, recomendações personalizadas, "perto de mim".

### 4. Espectador — 202 WAU (53%) — falha de ativação, não persona
**Comportamento no app:** abre o app 2×/semana e sai. 63% nem têm conta (device-only). Dos 74 registrados: 57% abre-e-sai; **20% abriu o fluxo de review e abandonou** (17 abandonos); 15% navega feed/tabs sem entrar em restaurante; **zero** taps em card de restaurante.
**O que defende o diagnóstico:** 73/74 emitem autocapture (2.986 eventos) — interagem com a UI, mas quase nada vira ação semântica. Rolam o feed de *pessoas* (14) mas nunca o de *lugares* (0).
**Leitura:** o maior grupo do WAU é o que menos entrega valor — e metade dele nem conta tem.
**Onde melhorar:** (a) corrigir o elo feed → detalhe do restaurante (0 card taps — pode ser bug de tracking ou de UI); (b) onboarding orientado a discovery (busca antes de review); (c) reduzir fricção do fluxo de review (17 abandonos).

## Insights priorizados (onde podemos melhorar)
1. **Funil feed → restaurante quebrado** — 0 card taps nos Espectadores: o app não converte leitura de review em intenção de "onde comer". Instrumentar `restaurant_card_tapped` e testar CTA de entrada no restaurante. [Owner: Engenharia + Product]
2. **Ativação:** 53% do WAU não tem conta e 63% dos Espectadores não cadastram. Testar onboarding discovery-first (buscar restaurante antes de pedir review). [Owner: Growth]
3. **Concentração de conteúdo:** 77% das reviews vêm de 20 pessoas. Escalar programa de creators de 2 → 20+. [Owner: Comunidade]
4. **Interator é atalho:** nudge pós-reação para primeira criação — 20% dos Espectadores já tentaram criar e abandonaram. [Owner: Product]
5. **Share não sustenta perfil:** 0,9% dos ativos compartilham em 30d; invite sem tracking. Instrumentar antes de investir em share. [Owner: Engenharia]

## Limitações (levar para a apresentação)
- Duração de sessão cobre ~44 usuários no WAU (`app_session_ended` esparso) — gradiente direcional.
- WAU conta distinct_id (dispositivo), não conta — Espectador inclui reinstalações.
- Comentários/share têm n pequeno — conclusões direcionais.

## Gráficos (um por métrica — /charts/)
- `perfis_wau.png` — distribuição do WAU
- `perfis_acessos.png` — acessos/semana por perfil
- `perfis_duracao.png` — duração de sessão por perfil
