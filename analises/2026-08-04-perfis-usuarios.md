# Análise de Perfis de Usuário — Gomo
Data: 2026-08-04 · Autor: Gomes · Solicitante: Mairon Carvalho

## Metodologia
- Classificação comportamental via PostHog (eventos), janela de 30d (05/jul–03/ago) para definir o perfil; janela de 7d (28/jul–03/ago) para medir atividade.
- Base: 2.678 usuários ativos em 30d; WAU 7d = 385 usuários (3.831 sessões).
- Regras de prioridade: Criador (review/lista/comentário) > Interator (reação/like/follow/share) > Navegador (view de restaurante/busca/mapa/ranking) > Espectador (nada rastreado).
- Ground truth de cadastro: tabela `users` via MCP (1.604 cadastrados).
- Outlier removido: usuário cb1062… (235 sessões/semana, 225 com duração 0 — provável emulador/instância sempre aberta).

## Veredito de validação
| Perfil hipotético | Existe? | Tamanho (WAU) | Veredito |
|---|---|---|---|
| Influencer | ✅ Sim, forte | 116 (30,1%) | Validado. Renomear para **Criador**. |
| Sociável | ⚠️ Fraco | 18 (4,7%) | Existe em forma embrionária, pequeno demais para segmento estratégico. |
| Explorador | ✅ Sim | 49 (12,7%) | Validado, mas raso (2,1 views de restaurante/usuário/30d). |
| — (não previsto) | 🚨 **52,5% do WAU** | 202 | **Espectador**: abre o app e não faz nada rastreado. 63% nem são cadastrados. |

## Métricas por perfil (janela 7d salvo indicação)
| Perfil | WAU | % WAU | Acessos/sem (mediana) | p90 acessos | Duração sessão (mediana) | Cadastrados |
|---|---|---|---|---|---|---|
| Criador | 116 | 30,1% | **5** | 58 | **~38s** | 95% |
| Interator | 18 | 4,7% | 3 | 13 | ~20s | 100% |
| Navegador | 49 | 12,7% | 2 | 6 | ~27–41s | 100% |
| Espectador | 202 | 52,5% | 2 | 5 | ~15–20s | 37% |

## Indicadores que defendem cada perfil

### 1. Criador (ex-Influencer) — 116 WAU
- 1.712 reviews em 30d; mediana 3 reviews/usuário.
- **Concentração: top 5 = 40% das reviews; top 10 = 57%; top 20 = 77%.**
- 36% também reagem (cross-behavior: quem cria também interage).
- 29/116 criam listas/coleções; 16/116 compartilharam em 30d.
- Sessão mais longa (~38s) e mais frequente (mediana 5/sem, p90 58).
- O `account_level` do backend só marca 2 usuários como `influenciador` — a base comportamental (116) é 58× maior que a flag manual.

### 2. Interator (ex-Sociável) — 18 WAU
- 50% fazem apenas reações (review_reacted); zero comentários no segmento.
- Comentários no app inteiro: 9 em 7d. Follows: 23/7d. Reações: 118/7d.
- Sessão curta (~20s), mediana 3 acessos/semana.
- Não é um perfil durável — é **Criador em estágio inicial** (mesmo grupo que reage antes de criar).

### 3. Navegador (ex-Explorador) — 49 WAU
- 100% cadastrados — intenção real de uso.
- 30d: 103 views de restaurante (2,1/usuário), 35 buscas, 15 rankings, 16 coleções vistas, 13 mapas.
- Zero criação, zero interação — puro discovery ("onde vou comer").
- Sessões curtas (2/semana) mas engajamento de leitura (27–41s) acima de Interator/Espectador.

### 4. Espectador — 202 WAU (não previsto pelo Mairon)
- Abre o app 2×/semana e sai: nenhum evento de navegação, busca ou interação.
- **63% não têm conta cadastrada** (device-only) vs 95–100% nos demais perfis.
- Em 30d: 2.117 usuários (79% dos ativos) se comportam assim — a maioria abre uma vez e some.
- É um sintoma de funil de ativação, não um perfil de persona.

## Refino de nomes (proposta)
| Hipótese | Nome proposto | Por quê |
|---|---|---|
| Influencer | **Criador** | O comportamento medido é criação de conteúdo (review/lista), não influência/reach. "Influencer" implica audiência que não é rastreada. Backend tem flag `influenciador` (2 usuários) — reservar o termo para o programa de creators. |
| Sociável | **Interator** (ou absorver em Criador) | "Sociável" sugere comportamento dominante; na prática são 4,7% do WAU em transição. |
| Explorador | **Navegador** | Evita colisão com `user_expertise.nivel = 'explorador'` (364 usuários se autodeclaram explorador no onboarding). |
| — | **Espectador** | Descreve o comportamento real: abre, vê, sai. |

## Insights
1. **Metade do WAU não ativa.** 52,5% abrem e não navegam; 63% nem cadastram. O caminho Cadastro → 1ª navegação (busca/mapa/ranking) é o maior vazamento do funil. Ação: testar onboarding orientado a discovery (busca de restaurante antes de pedir review) [Owner: Product/Growth].
2. **Risco de concentração de conteúdo.** 77% das reviews vêm de 20 usuários. Saúde do feed depende de ~20 pessoas. Ação: programa de creators (o backend já tem a flag) para escalar de 2 → 20+ influenciadores reconhecidos [Owner: Comunidade].
3. **"Compartilhar" não existe como comportamento.** 23 usuários em 30d (0,9%); invite flow não tem tracking. A hipótese original dependia de share — hoje não sustenta segmentação. Ação: instrumentar share/invite antes de usar como critério de perfil [Owner: Engenharia].
4. **Sociável é um estágio, não um destino.** Reações precedem criação (36% dos criadores também reagem). Tratar interação como milestone de ativação (Criador em D+30), não como persona a servir.
5. **Taxonomia do onboarding pode substituir a segmentação ad-hoc.** Backend já tem `user_expertise.nivel` (curioso 909 / explorador 364 / especialista 114 / guru 36) e `account_level` (comum/influenciador). Correlacionar autodeclaração × comportamento (feito neste doc) é o caminho para perfis estáveis.

## Limitações
- Duração de sessão: `app_session_ended` com duração válida cobre só ~44 usuários no WAU — gradiente é direcional, não estatístico.
- Comentários/compartilhamento têm volume baixíssimo — conclusões sobre esses comportamentos têm n pequeno.
- WAU do PostHog conta distinct_id (dispositivo), não contas — Espectador pode incluir reinstalações.
