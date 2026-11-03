Propósito: descrever gates de acesso e elegibilidade relacionados a vendedores, afiliados e produtos comprovados no código.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Gates verificados

- **Afiliado no checkout:** conta WordPress existente com capability `yith_affiliate` e `kyc_last_status=approved`; duplicidade/conflito dos metadados e falha de desserialização fecham a elegibilidade. `backend/fastapi_app/services/checkout/checkout_service.py:6-25`
- **Link de afiliado:** precisa estar `active`, corresponder ao produto e ser elegível; produto desativado, sem estoque ou produto não publicado/aprovado invalida a validação. `backend/fastapi_app/services/checkout/checkout_service.py:28-77`
- **Relatório de vendedor:** shortcode exige sessão e capability `wcfm_vendor`; consulta os produtos publicados do usuário. `frontend/includes/seller/grpms-seller.php:145-184`
- **Área financeira:** shortcode exige login e validação de acesso `request-transfer-fin`; sem permissão retorna mensagem de acesso negado. A resolução do papel é condição posterior para exibição. `frontend/includes/financial/grpms-payment.php:148-170`

## Limites

Este resumo não equivale a uma matriz completa de autorização do WordPress. Papéis e capabilities de terceiros dependem de plugins e configurações não verificadas no código local. Acesso administrativo, escopo de cada ação financeira e política completa de KYC são **UNVERIFIED** fora dos gates acima.
