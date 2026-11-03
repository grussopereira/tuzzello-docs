Propósito: descrever o ciclo do vendedor que pode ser confirmado no código disponível.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Ciclo observado

A camada frontend identifica vendedores pelo papel WCFM `wcfm_vendor` em relatório de afiliados e limita a listagem de produtos publicados ao autor vendedor. `frontend/includes/seller/grpms-seller.php:145-184`

No pagamento de comissões, o papel resolvido como vendedor consulta linhas da própria conta que estejam `completed` e `delivered`, sem `vendor_paid_date`, sem reembolso/remoção e com `vendor_commission <> 0`. A separação entre indisponível e disponível usa `created + 15 dias`; pedidos sem solicitação associada aparecem na classe disponível após esse prazo. `frontend/includes/financial/grpms-payment.php:187-237`

A disponibilidade não prova pagamento efetuado. O código de conclusão automática diz expressamente que não executa liberação de payout. `backend/fastapi_app/services/checkout/auto_complete.py:1-7`

## Estados

Estados de ciclo de vida de vendedor (cadastro, aprovação, suspensão, KYC e reativação) são **UNVERIFIED**: a fonte externa WCFM/YITH não está incluída como implementação local e o código inspecionado não estabelece uma máquina de estados completa do vendedor. A presença de `wcfm_vendor` comprova apenas o papel consultado no relatório citado.

O onboarding do vendedor, documentos exigidos e critérios de aprovação são **UNVERIFIED**. Não inferir esses procedimentos a partir do papel.

## Referências cruzadas

Veja [MARKETPLACE-RULES.md](MARKETPLACE-RULES.md) para a sequência de venda e condições de elegibilidade financeira; [ACCESS-POLICIES.md](ACCESS-POLICIES.md) para os gates de interface confirmados.
