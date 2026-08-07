# Perfis de Usuário Gomo — Material de Apresentação (v2 — hierarquia exclusiva)
Data: 2026-08-07 · Autor: Gomes · Solicitante: Mairon Carvalho
Base: análise comportamental PostHog 30d (05/jul–03/ago) para classificação, 7d (28/jul–03/ago) para medição.

## Regras da segmentação (v2)
1. **Somente usuários COM conta** (interseção com tabela `users` — 1.604 cadastrados). WAU cadastrado = 251 (de 385).
2. **Mutuamente exclusivo** — cada usuário pertence a exatamente 1 perfil, definido pelo comportamento de maior nível que executou em 30d. Hierarquia: **Criador > Interator > Navegador > Explorador**.
3. Prova de exclusividade: soma dos perfis = 251 = total de usuários (zero dupla contagem).

## Mapa dos 4 perfis (1 slide) — WAU com conta = 251

| Perfil | WAU | % | Acessos/sem (mediana) | p90 | Duração sessão | Regra de classificação |
|---|---|---|---|---|---|---|
| **Criador** | 110 | 43,8% | **5** | 61 | ~36s | Review OU lista OU comentário em 30d |
| **Interator** | 18 | 7,2% | 3 | 13 | ~20s | Reação/like/follow/share (sem criar) |
| **Navegador** | 49 | 19,5% | 2 | 6 | ~27s | View de restaurante/busca/mapa/ranking |
| **Explorador** | 74 | 29,5% | 2 | 6 | ~19s | Nada rastreado além da sessão |

## Slide por perfil

### 1. Criador — 110 WAU (43,8%) — o motor de conteúdo
**Comportamento no app:** cria review (1.696 em 30d, mediana 3,5/usuário), lista e comenta. Sessão mais longa (~36s) e mais frequente (5/sem, p90 61). 36% também reagem — criação e interação andam juntas (composição interna do perfil, não dupla contagem).
**O que defende o perfil:** top 5 = 41% das reviews, top 10 = 58%, top 20 = 78%. Sem eles não há feed. O backend só marca 2 usuários como `influenciador` — a base comportamental é 55× maior que a flag manual.
**Risco:** dependência de ~20 pessoas para a saúde do feed.
**Onde melhorar:** programa de creators (flag já existe no backend) para escalar de 2 → 20+; reconhecimento/gamificação para quem cria.

### 2. Interator — 18 WAU (7,2%) — o estágio inicial
**Comportamento no app:** reage/curte/seguem (50% do grupo só reage); comenta pouco (9 comentários em 7d no app inteiro). 3 acessos/sem, sessão ~20s.
**O que defende o perfil:** 100% com conta; está a 1 passo da criação — 36% dos Criadores também reagem (interação precede criação).
**Leitura:** é estágio de transição, não persona durável (por isso o merge faz sentido — mas vale apresentar separado para mostrar o funil).
**Onde melhorar:** transformar reação em milestone de ativação — nudge pós-reação ("viu algo bom? conta pra gente"), CTA para primeira review.

### 3. Navegador — 49 WAU (19,5%) — o público-alvo do core value
**Comportamento no app:** puro discovery — 103 views de restaurante, 35 buscas, 15 rankings, 16 coleções vistas, 13 mapas em 30d. Zero criação, zero interação. 2 acessos/sem, sessão de leitura (~27s) acima de Explorador.
**O que defende o perfil:** 100% cadastrados (intenção real de uso); job é exatamente "onde vou comer" — a promessa central do Gomo.
**Onde melhorar:** é o perfil que valida o produto, mas é raso (2,1 views/usuário/30d) — medir conversão leitura→decisão (salvar em coleção, chamar rota); testar feed de lugares, recomendações personalizadas, "perto de mim".

### 4. Explorador — 74 WAU (29,5%) — falha de ativação, não persona
**Comportamento no app:** abre o app 2×/semana e sai, sem nenhuma ação rastreada além da sessão (em 30d: 850 cadastrados — 60,6% — se comportam assim). Dos 74: 57% abre-e-sai; **20% abriu o fluxo de review e abandonou** (17 abandonos); 15% navega feed/tabs sem entrar em restaurante; **zero** taps em card de restaurante.
**O que defende o diagnóstico:** 73/74 emitem autocapture (2.986 eventos) — interagem com a UI, mas quase nada vira ação semântica. Rolam o feed de *pessoas* (14) mas nunca o de *lugares* (0).
**Leitura:** o maior grupo do WAU é o que menos entrega valor.
**Onde melhorar:** (a) corrigir o elo feed → detalhe do restaurante (0 card taps — pode ser bug de tracking ou de UI); (b) onboarding orientado a discovery (busca antes de review); (c) reduzir fricção do fluxo de review (17 abandonos).

## Insights priorizados (onde podemos melhorar)
1. **Funil feed → restaurante quebrado** — 0 card taps nos Exploradores: o app não converte leitura de review em intenção de "onde comer". Instrumentar `restaurant_card_tapped` e testar CTA de entrada no restaurante. [Owner: Engenharia + Product]
2. **Ativação:** 74 usuários com conta (29,5% do WAU cadastrado) não navegam; em 30d são 850 (60,6%). Testar onboarding discovery-first (buscar restaurante antes de pedir review). [Owner: Growth]
3. **Concentração de conteúdo:** 78% das reviews vêm de 20 pessoas. Escalar programa de creators de 2 → 20+. [Owner: Comunidade]
4. **Interator é atalho:** nudge pós-reação para primeira criação — 20% dos Exploradores já tentaram criar e abandonaram. [Owner: Product]
5. **Share não sustenta perfil:** 0,9% dos ativos compartilham em 30d; invite sem tracking. Instrumentar antes de investir em share. [Owner: Engenharia]

## Comparativo v1 (todos) vs v2 (só cadastrados)
| Perfil | v1 WAU total (385) | v2 WAU com conta (251) |
|---|---|---|
| Criador | 116 (30%) | 110 (43,8%) |
| Interator | 18 (5%) | 18 (7,2%) |
| Navegador | 49 (13%) | 49 (19,5%) |
| Explorador (ex-Espectador) | 202 (53%) | 74 (29,5%) |

O filtro de conta corta o Explorador em 63% — a maior parte dele era device-only (sem conta). Nos demais perfis quase não muda (≥95% já tinham conta).

## Limitações (levar para a apresentação)
- Duração de sessão cobre 8–13 usuários por perfil (`app_session_ended` esparso) — gradiente direcional.
- WAU conta distinct_id (dispositivo), não conta — Explorador pode incluir reinstalações.
- Comentários/share têm n pequeno — conclusões direcionais.

## Gráficos (um por métrica — /charts/)
- `perfis_wau.png` — distribuição do WAU com conta (v2)
- `perfis_acessos.png` — acessos/semana por perfil (v2)
- `perfis_duracao.png` — duração de sessão por perfil (v2)

## Slides (formato apresentação)

### Slide 1 — A nova segmentação
**Quem usa o Gomo com conta: 4 perfis, hierarquia exclusiva (n=251)**

| Perfil | WAU | % | Acessos/sem | Duração |
|---|---|---|---|---|
| Criador | 110 | 43,8% | 5 (p90: 61) | ~36s |
| Interator | 18 | 7,2% | 3 | ~20s |
| Navegador | 49 | 19,5% | 2 | ~27s |
| Explorador | 74 | 29,5% | 2 | ~19s |

Regras: só quem tem conta · cada usuário em exatamente 1 perfil (maior nível de ação em 30d).
**Insight:** quem tem conta e volta, cria — o Gomo é um app de criação (51% criam ou interagem), não de consumo passivo. O desafio não é engajar quem ativa; é ativar quem não navega (Explorador).

### Slide 2 — Criador (110 · 43,8%) — o motor de conteúdo
- 1.696 reviews em 30d (mediana 3,5/usuário) + listas e comentários.
- Sessão mais longa (~36s) e mais frequente (5/sem, p90 61).
- 36% também reagem — criação e interação andam juntas.
- **Insight:** top 5 = 41% das reviews, top 20 = 78%. O feed depende de ~20 pessoas; o backend marca só 2 como `influenciador` (55× menos que a realidade).
- **Ação:** programa de creators de 2 → 20+ (flag já existe). [Owner: Comunidade]

### Slide 3 — Interator (18 · 7,2%) — o estágio inicial
- Reage, curte, segue — 50% do grupo só reage; comenta pouco (9/7d no app todo).
- 100% com conta; 3 acessos/sem, sessão ~20s.
- **Insight:** está a 1 passo da criação — 36% dos Criadores também reagem. Interação precede criação.
- **Ação:** nudge pós-reação ("viu algo bom? conta pra gente") como CTA da 1ª review. [Owner: Product]

### Slide 4 — Navegador (49 · 19,5%) — o core value
- Puro discovery: 103 views de restaurante, 35 buscas, 15 rankings em 30d. Zero criação/interação.
- 100% com conta; sessão de leitura (~27s) acima de Explorador.
- **Insight:** é quem valida o "onde vou comer" — mas é raso (2,1 views/usuário/30d). O produto não mede nem incentiva a decisão.
- **Ação:** medir conversão leitura→decisão (salvar, rota) e testar "perto de mim" / recomendação. [Owner: Product]

### Slide 5 — Explorador (74 · 29,5%) — falha de ativação, não persona
- Abre 2×/semana e sai: nenhuma ação rastreada além da sessão.
- 20% abriu o fluxo de review e abandonou (17 abandonos); zero taps em card de restaurante.
- Em 30d, 850 cadastrados (60,6%) se comportam assim.
- **Insight:** o elo feed → detalhe do restaurante não acontece — o app não converte leitura em intenção de "onde comer".
- **Ação:** instrumentar `restaurant_card_tapped` + onboarding discovery-first (busca antes de review). [Owner: Eng + Growth]

### Slide 6 — Síntese: o insight da nova análise
**O Gomo tem um núcleo criador saudável (43,8% do WAU com conta) e um vazamento de ativação de 29,5%.**
- A hierarquia exclusiva prova: cada perfil é um degrau — Explorador → Navegador → Interator → Criador.
- Prioridade: consertar o 1º degrau (Explorador → Navegador) — é o maior grupo e o elo quebrado (0 card taps).
- Secundário: proteger o topo (Criador) da concentração de 78%/20 pessoas.
- Risco de não agir: metade do WAU com conta fica presa na base, e o feed depende de 20 pessoas.
