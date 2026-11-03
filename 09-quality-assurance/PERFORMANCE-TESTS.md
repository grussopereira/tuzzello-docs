Propósito: identificar a evidência existente sobre desempenho e concorrência.
Data: 2026-11-03.
Escopo: descreve o HEAD de FRONTEND e BACKEND nesta data (c1bd75d e e53fb27).

# Testes de desempenho

Inventário: 55 arquivos rastreados no FRONTEND e 18 no BACKEND em `tests/**` e `scripts/test-*`. Não foi identificado, no inventário examinado, benchmark com metas de latência, vazão ou memória; essa ausência limita-se aos nomes e conteúdos relevantes inspecionados e não prova ausência em ferramentas externas.

Há testes de concorrência/contenda que avaliam invariantes, não capacidade: entregas simultâneas idênticas no webhook (`BACKEND/tests/test_wc_webhook.py:274-290`), corrida entre webhook e reembolso (`:222-271`) e claim concorrente no probe MariaDB (`BACKEND/tests/test_wc_outbox_db.py:47-82`). O último é opt-in (`:19-27`) e não foi executado.

Não há métricas de desempenho verificadas neste levantamento. Ambiente e dados de benchmark, limites de SLA, volume esperado, execução em CI e regressões de performance são UNVERIFIED. Nenhuma suíte foi executada.
