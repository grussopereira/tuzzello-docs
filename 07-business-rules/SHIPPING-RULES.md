Propósito: explicar a lógica local de entrega/frete e o escopo não aplicável a produtos digitais.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## O que o código faz

A importação marca produto `_virtual` ou `_downloadable` como `shipping_status=delivered`; produto sem ambas as flags começa como `pending`. O frete registrado integra total pago conforme dados recebidos. `backend/fastapi_app/services/checkout/order_service.py:285-329, 343-350`

O cálculo simples consulta zona habilitada do vendedor para o país; se não encontrar, procura localização `OT`; sem correspondência retorna `0.00`. `backend/fastapi_app/services/checkout/shipping_service.py:5-30`

Existe código de rotas e integração EasyPost, incluindo cálculo/validação, tracking e histórico, mas isso não prova chamada em uma venda específica. A rota de compra de etiqueta responde `503` com mensagem de operação desabilitada neste deployment; o webhook também indica integração não configurada. `backend/fastapi_app/routers/checkout/easypost_router.py:80-85, 104-138, 141-192`

## Produtos digitais

Para a regra de negócio deste marketplace digital, produtos virtuais/baixáveis não aguardam transporte físico: a lógica local os marca entregues na entrada da linha. A conclusão automática de pedidos valida flags digitais e mantém produto não digital em estado `held`. `backend/fastapi_app/services/checkout/order_service.py:285-318`; `backend/fastapi_app/services/checkout/auto_complete.py:28-48`

## Não aplicável / não verificado

Transportadora física, compra de postagem ativa, SLA de entrega, cálculo de frete para todos os checkouts e operações externas foram deliberadamente não executados nesta documentação. A rota indica compra de etiqueta desabilitada; não fizemos chamadas a provedores. Não há regra local que estabeleça prazos de entrega física (**UNVERIFIED**).
