Propósito: registrar estados e validações do produto pertinentes à publicação e à venda.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Publicação

O formulário frontend salva produto novo como `draft`, aplica as flags virtual/baixável e marca produto para aprovação pendente quando aplicável. Produto de treinamento segue condição separada que exige completar o construtor antes de aprovação. `frontend/includes/products/grpms-products.php:6199-6222, 6308`

O backend considera elegível para checkout um produto publicado cujo status de aprovação seja `approved` ou ausente. `backend/fastapi_app/services/checkout/checkout_service.py:28-38`

## Elegibilidade no checkout

Para validação de afiliado, produto não pode estar desativado, fora de estoque ou com link inválido; além disso, o registro de desconto/quantidade precisa existir quando quantidade é maior que um. `backend/fastapi_app/services/checkout/checkout_service.py:45-89`

No cadastro do pedido, flags `_virtual` e `_downloadable` afetam estado de entrega: qualquer uma das duas define `delivered` no início; caso contrário, `pending`. `backend/fastapi_app/services/checkout/order_service.py:285-318`

O status do ciclo de produto inclui estados de aprovação e exclusão exibidos no painel, tais como `pending`, `rejected`, `training_incomplete` e solicitação de exclusão pendente. `frontend/includes/products/grpms-products.php:320-349`

## Edição e retirada

O painel tem lógica de solicitação de exclusão para aprovado e exclusão direta para certos estados não aprovados; ações são bloqueadas para pendência de exclusão/aprovação em condições específicas. `frontend/includes/products/grpms-products.php:328-338, 406-465`

As regras administrativas completas de aprovação, publicação final e remoção definitiva são **UNVERIFIED** aqui; este documento limita o fluxo ao que aparece nas funções citadas.
