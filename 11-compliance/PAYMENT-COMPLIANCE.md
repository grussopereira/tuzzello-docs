# Conformidade de Pagamentos e PCI DSS — rascunho

**Propósito:** descrever o que o código evidencia sobre dados de cartão e delimitar os elementos PCI DSS não verificados.

**Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nesta data.

> **RASCUNHO:** todo conteúdo legal está sujeito à revisão por profissionais jurídicos qualificados em cada jurisdição.

## Evidência do código sobre cartão

O backend lê metadados WooPayments com identificador de transação e taxa, e grava `order_id`, `transaction_id`, `payment_method='card'`, valor de taxa e moeda em `grpms_payment_fees`. `backend/fastapi_app/services/checkout/financial_fee_service.py:7-23`, `:37-49`.

O registro de pedido também contém `payment_method`, valores e identificadores de cliente/pedido. `backend/fastapi_app/services/checkout/order_service.py:261-281`.

Na parte de código revisada, não foi localizada evidência de gravação de número completo do cartão (PAN), código de segurança (CVV/CVC) ou trilha magnética pelo fluxo acima. Isto **não prova** que tais dados não sejam capturados/processados por WooCommerce, gateways, plugins, navegador, logs ou infraestrutura, nem estabelece escopo PCI.

## PCI DSS: itens não verificados

**UNVERIFIED:** versão aplicável, escopo e elegibilidade, fluxo completo de dados, tokenização pelo provedor, armazenamento/mascaramento de PAN, segmentação, controles de rede, gestão de chaves, varreduras, testes, evidências, SAQ/ROC, AOC, certificação ou validação de qualquer provedor, responsabilidade compartilhada e estado de conformidade da plataforma. Nenhuma certificação ou conformidade PCI DSS é afirmada por este documento.

## Revisão necessária

Confirmar arquitetura e fluxo com adquirente/gateway e responsáveis técnicos, avaliar ambiente efetivamente implantado e obter evidências independentes antes de declarar escopo ou conformidade.
