# Análise de Perfis de Usuário — Gomo
Data: 2026-08-04 · Autor: Gomes · Solicitante: Mairon Carvalho

## Metodologia
- Classificação comportamental via PostHog (eventos), janela de 30d (05/jul–03/ago) para definir o perfil; janela de 7d (28/jul–03/ago) para medir atividade.
- Base: 2.678 usuários ativos em 30d; WAU 7d = 385 usuários (3.831 sessões).
- Regras de prioridade: Criador (review/lista/comentário) > Interator (reação/like/follow/share) > Navegador (view de restaurante/busca/mapa/ranking) > Espectador (nada rastreado).
- Ground truth de cadastro: tabela `users` via MCP (1.604 cadastrados).
- Outlier removido: usuário cb1062… (235 sessões/semana, 225 com duração 0 — provável emulador/instância sempre aberta).

## Veredito de validação (somente usuários COM conta — 1.604 cadastrados)
> Filtros aplicados a pedido de Mairon (04/ago): (1) desconsiderar quem não tem conta (device-only); (2) fundir Interator (ex-Sociável) em Criador. WAU registrado = 251.

| Perfil | Existe? | Tamanho (WAU reg.) | Veredito |
|---|---|---|---|
| Criador (ex-Influencer + Sociável) | ✅ Sim, dominante | 128 (51,0%) | Validado. Inclui quem reage/comenta sem criar review — estágio inicial de criação. |
| Navegador (ex-Explorador) | ✅ Sim | 49 (19,5%) | Validado, mas raso (2,1 views de restaurante/usuário/30d). |
| Espectador (não previsto) | ⚠️ **29,5% do WAU reg.** | 74 | Conta criada, abre o app e não faz nada rastreado. |

## Métricas por perfil (somente registrados; janela 7d salvo indicação)
| Perfil | WAU | % WAU | Acessos/sem (mediana) | p90 acessos | Duração sessão (mediana) |
|---|---|---|---|---|---|
| Criador | 128 | 51,0% | **5** | 53 | ~29–33s |
| Navegador | 49 | 19,5% | 2 | 6 | ~27–41s |
| Espectador | 74 | 29,5% | 2 | 6 | ~17s |

Os 2 perfis comportamentais hipotetizados (Criador + Navegador) explicam **70,5% do WAU registrado**.

## Indicadores que defendem cada perfil

### 1. Criador (Influencer + Sociável fundidos) — 128 WAU
- 1.696 reviews em 30d; mediana 3 reviews/usuário.
- **Concentração: top 5 = 41% das reviews; top 10 = 58%; top 20 = 78%.**
- Composição do merge: 38% reagem (49/128), 12 comentam, 28 criam listas, 14 compartilham.
- O subgrupo "só reage" (ex-Sociável, 18 usuários) é o estágio inicial: quem reage tende a criar depois.
- Sessão mais longa (~29–33s) e mais frequente (mediana 5/sem, p90 53).
- O `account_level` do backend só marca 2 usuários como `influenciador` — a base comportamental (128) é 64× maior que a flag manual.

### 2. Navegador (ex-Explorador) — 49 WAU
- 100% cadastrados — intenção real de uso.
- 30d: 103 views de restaurante (2,1/usuário), 35 buscas, 15 rankings, 16 coleções vistas, 13 mapas.
- Zero criação, zero interação — puro discovery ("onde vou comer").
- Sessões curtas (2/semana) mas engajamento de leitura (27–41s) acima de Espectador.

### 3. Espectador — 74 WAU registrados (29,5%; 850 em 30d — 60,6%)
- Criou conta, abre o app 2×/semana e sai: nenhum evento de navegação, busca ou interação rastreada.
- Em 30d: 850 registrados (60,6%) se comportam assim — a maioria abriu uma vez e não voltou.
- Mesmo sem device-only, o problema de ativação persiste: conta criada ≠ uso. É sintoma de funil, não persona.

#### O que o Espectador faz de fato (catálogo completo de eventos, 30d — n=74)
> Resposta à pergunta do Mairon (04/ago): "o espectador faz o que se não é trackeável nenhuma ação dele?"

| Comportamento real | Usuários | % |
|---|---|---|
| Só sessões (abre e sai; autocapture mostra taps na UI) | 42 | 57% |
| **Abriu fluxo de review e abandonou** (21 flow_opens, 17 abandoned) | 15 | 20% |
| Navegou feed/tabs sem entrar em restaurante (14 feed_pag, 7 tab_switch) | 11 | 15% |
| Só mexeu no perfil | 4 | 5% |
| Só abriu via push | 2 | 3% |

- **Zero** `restaurant_card_tapped`, `restaurant_viewed`, `search_performed`, `map_opened`, `ranking_viewed` no grupo.
- 73/74 emitem `$autocapture` (2.986 eventos em 30d) — interagem com a UI, mas quase nada vira evento semântico (feed view não é trackeado; só paginação).
- Rolam o feed de *pessoas* (`home_people_feed_paginated`: 14) mas nunca o de *lugares* (`home_places_feed_paginated`: 0).

**Diagnóstico em 2 camadas:**
1. *Gargalo real de produto:* o passo feed → detalhe do restaurante não acontece para eles (0 card taps). O produto não converte leitura de review em intenção de "onde comer".
2. *Gap de instrumentação:* interação de UI existe (autocapture) mas não é mapeada em eventos semânticos de navegação.

## Refino de nomes (proposta final)
| Hipótese | Nome proposto | Por quê |
|---|---|---|
| Influencer + Sociável | **Criador** | O comportamento medido é criação (review/lista/comentário/reação). "Influencer" implica audiência que não é rastreada. O subgrupo que só reage é estágio inicial do mesmo perfil. |
| Explorador | **Navegador** | Evita colisão com `user_expertise.nivel = 'explorador'` (364 usuários se autodeclaram explorador no onboarding). |
| — | **Espectador** | Descreve o comportamento real: abre, vê, sai. |

## Insights
1. **1 em cada 3 registrados não ativa.** 29,5% do WAU registrado abre e não navega; em 30d são 850 (60,6%) que abriram uma vez e não voltaram. O caminho Cadastro → 1ª navegação (busca/mapa/ranking) é o maior vazamento do funil. Ação: testar onboarding orientado a discovery (busca de restaurante antes de pedir review) [Owner: Product/Growth].
2. **Risco de concentração de conteúdo.** 78% das reviews vêm de 20 usuários. Saúde do feed depende de ~20 pessoas. Ação: programa de creators (o backend já tem a flag) para escalar de 2 → 20+ influenciadores reconhecidos [Owner: Comunidade].
3. **"Compartilhar" não existe como comportamento.** 14 dos 128 criadores compartilharam em 30d (11%); invite flow não tem tracking. A hipótese original dependia de share — hoje não sustenta segmentação. Ação: instrumentar share/invite antes de usar como critério de perfil [Owner: Engenharia].
4. **Interação é estágio, não destino.** O merge Sociável→Criador confirma: quem reage/comenta sem criar é o mesmo perfil em estágio inicial. Tratar reação como milestone de ativação (Criador em D+30), não como persona a servir.
5. **Taxonomia do onboarding pode substituir a segmentação ad-hoc.** Backend já tem `user_expertise.nivel` (curioso 909 / explorador 364 / especialista 114 / guru 36) e `account_level` (comum/influenciador). Correlacionar autodeclaração × comportamento (feito neste doc) é o caminho para perfis estáveis.

## Limitações
- Duração de sessão: `app_session_ended` com duração válida cobre só ~44 usuários no WAU — gradiente é direcional, não estatístico.
- Comentários/compartilhamento têm volume baixíssimo — conclusões sobre esses comportamentos têm n pequeno.
- WAU do PostHog conta distinct_id (dispositivo), não contas — Espectador pode incluir reinstalações.
