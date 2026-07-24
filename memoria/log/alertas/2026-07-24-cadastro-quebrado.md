---
id: ale-2026-07-24-cadastro-quebrado
tipo: alerta
data: 2026-07-24
titulo: SWNPGMO-339 — cadastro quebrado, sem assignee há 48h
metrica: cadastros_24h
valor: 26
esperado: [50, 150]
desvio_pct: -73
veredito_humano: pendente
causa_atribuida: swpnpgmo-339
autor: agente-gomo
---

# Cadastro quebrado bloqueia aquisição

SWNPGMO-339 está em "Tarefas pendentes" sem assignee desde 22/07. O bug quebra o fluxo de cadastro (erro + redirect incorreto) e o serviço de e-mail de recuperação de senha.

**Impacto:** 0 novos cadastros reportados por 7 dias no PostHog. 26 cadastros em D-1 (possível bypass ou tracking parcial).

**Relação com outros bugs:** SWNPGMO-6 (mesmo fluxo, corrigido 25/05 — possível regressão), SWNPGMO-216 (fornecedor e-mail).
