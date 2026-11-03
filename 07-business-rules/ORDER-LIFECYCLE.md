Propósito: enumerar estados de pedido e transições executadas pela sincronização do backend.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Estados e criação

Na tabela financeira `grpms_main_orders`, a sincronização inicial usa status recebido, com fallback `pending`; pedidos novos só são sincronizados se chegarem como `processing` ou `completed`. As linhas começam com `commission_status`, `withdraw_status` e `refund_status` como `pending`, e datas de pagamento nulas. `backend/fastapi_app/services/checkout/order_service.py:250-283, 504-514`

## Transições

| De | Para | Condições observadas |
|---|---|---|
| `pending` ou `on-hold` | `processing` | Transição permitida pelo mapa; linhas e itens do pedido devem corresponder e nenhum estado financeiro deve bloquear. |
| `pending`, `processing` ou `on-hold` | `completed` | Mesmas travas financeiras e consistência; sincronizador rejeita outros estados de origem. |
| `failed`, `cancelled` ou qualquer outro | `processing`/`completed` | Sem transição permitida pelo mapa local; reconciliação necessária. |

Mapa explícito: `processing` aceita `pending`,`on-hold`; `completed` aceita `pending`,`processing`,`on-hold`. `backend/fastapi_app/services/checkout/order_service.py:16-20`

Ao tentar alterar linhas existentes, o backend exige que todas as linhas estejam no status anterior admissível; bloqueia se houver reembolso total/parcial, remoção, data de comissão paga ou payout associado e só então atualiza com guarda condicional. `backend/fastapi_app/services/checkout/order_service.py:541-587`

## Estado de entrega

`shipping_status` inicia como `delivered` para produto virtual/baixável e `pending` para item físico. `backend/fastapi_app/services/checkout/order_service.py:285-318`

## Conclusão automática

Há código de conclusão automática condicionado por flag, mas o próprio módulo se descreve como não conectado; requer pagamento confirmado por evento servidor autenticado e declara sem gravação WooCommerce, âncora de entrega, persistência da janela de reembolso ou payout. Portanto, não documentar como fluxo ativo. `backend/fastapi_app/services/checkout/auto_complete.py:1-7, 61-97`

Outros status WooCommerce podem existir no sistema externo, mas não são enumeração completa verificável deste código (**UNVERIFIED**).
