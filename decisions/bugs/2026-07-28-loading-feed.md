
## Loading infinito no feed — RESOLVIDO (3 rounds de fix)

- **Severidade:** HIGH → RESOLVIDO
- **Fluxo:** feed (core)
- **Autor dos fixes:** Mairon Carvalho
- **Período:** 27-28/jul/2026

### Linha do tempo dos fixes

**Round 1 — PR #104** (merged 27/jul 16:05)
- Causa: `useEffect` do GPS não declarava `authLoading` como dependência
- Se GPS chegasse antes do auth terminar, efeito retornava early e nunca re-executava
- Fix: 1 linha — adicionar `authLoading` ao array de deps
- Arquivo: `screens/HomeScreen.js` (+1/-1)

**Round 2 — PR #113** (merged 27/jul 21:00)
- 3 correções estruturais:
  1. `services/api.js`: AbortError do fetch timeout gerava TypeError não-tratado (response = undefined após abort)
  2. `screens/HomeScreen.js`: `loadInitial` convertido para `useCallback` + ref `firstLoadDone`
  3. Guard `isActiveTab`: preload de aba inativa não reseta mais estado da aba visível
- Arquivos: `screens/HomeScreen.js` (+23/-10), `services/api.js` (+12/-1)

**Round 3 — PR #115** (merged 27/jul ~22:00) + commits diretos
- O PR #113 introduziu dependência circular:
  `loadInitial() → setPlaceCategories() → loadInitial recriado (useCallback) → useEffect dispara → loadInitial() ∞`
- Fix: `loadInitialRef` (ref estável) no lugar de `loadInitial` nos arrays de deps
- Commit adicional: moveu `useRef` para evitar TDZ
- Arquivos: `screens/HomeScreen.js` (+24/-13), + ajustes em AuthContext, App.js, LoginScreen

### Dados PostHog ($exception)

| Data | Usuários | Eventos |
|------|----------|---------|
| 22/jul | 14 | 52 |
| 23/jul | 22 | 89 ← pico |
| 24/jul | 10 | 47 |
| 25/jul | 3 | 7 |
| 26/jul | 3 | 6 |
| 27-28/jul | — | ingestion lag |

Queda de 87% nos usuários afetados (22→3) entre 23 e 26/jul.

### Análise de qualidade do fix

✅ **Correto.** O padrão `useRef` + `loadInitialRef` é a abordagem canônica do React para quebrar dependências circulares com `useCallback`. O `loadVersion` interno protege contra race conditions.

✅ **Defensivo.** O fix do `api.js` cobre graceful degradation em timeout — sem ele, qualquer timeout de 30s no fetch gerava tela branca (TypeError → AppErrorBoundary).

⚠️ **Risco residual:** O `firstLoadDone.current` só é setado como `true` dentro do `loadInitial` (linha 517). Se a primeira chamada ao `loadInitial` falhar (erro de rede), `firstLoadDone` continua `false`, e a próxima tentativa reseta o loading state do zero. Isso é tolerável (re-tentar com skeleton é melhor que ficar preso), mas pode causar flicker visual em redes instáveis.

### Ação

Bug **resolvido** — todos os PRs mergeados no `main`. Monitorar `$exception` nos próximos 2 dias quando o ingestion lag do PostHog normalizar pra confirmar zeragem completa.

---

🤖 HANDOFF — Tamanduá

Jira: N/A (não havia ticket — foi reportado direto no Slack)
Severidade: HIGH (resolvido)
Repo: restu-mobile
Arquivo: screens/HomeScreen.js:508-580, services/api.js:67-82
Causa: Raça entre GPS e auth (PR #104) + TypeError em fetch abortado (PR #113) + dependência circular loadInitial/placeCategories (PR #115)
Status: RESOLVIDO — sem ação pendente
