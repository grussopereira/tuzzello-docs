# Matriz de Conformidade por Jurisdição, Produto e Modelo Operacional — rascunho

**Propósito:** expor evidências técnicas e lacunas de conformidade sem presumir que uma jurisdição governa os demais mercados.

**Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nesta data.

> **RASCUNHO:** todo conteúdo legal está sujeito à revisão por profissionais jurídicos qualificados em cada jurisdição.

**Regra de leitura:** o código não demonstra aplicabilidade ou conformidade jurídica. Cada linha é uma pauta de avaliação; não constitui afirmação de obrigação legal.

## Brasil

| Produto | Modelo operacional observado | Evidência no código | Avaliação jurídica |
|---|---|---|---|
| Marketplace / perfis | Perfis de usuário, endereço, telefone e dados KYC; pedidos com endereços de cobrança/entrega | `frontend/includes/profile/grpms-profile.php:827-903`; `backend/fastapi_app/services/checkout/order_service.py:419-454` | Aplicabilidade, papéis, bases, direitos, retenção, transferências e controles: **UNVERIFIED** |
| Checkout e pagamentos | Pedido registra método e valores; backend persiste ID de transação e taxa | `backend/fastapi_app/services/checkout/order_service.py:261-281`; `backend/fastapi_app/services/checkout/financial_fee_service.py:7-23`, `:37-49` | Regras de pagamentos e responsabilidades: **UNVERIFIED** |
| Afiliados e vendedores | Elegibilidade usa papel de afiliado e status KYC; comissões ligadas ao pedido | `backend/fastapi_app/services/checkout/checkout_service.py:6-25`; `backend/fastapi_app/services/checkout/order_service.py:262-281` | Natureza contratual, tributária e regulatória: **UNVERIFIED** |

## União Europeia / Espaço Econômico Europeu

| Produto | Modelo operacional observado | Evidência no código | Avaliação jurídica |
|---|---|---|---|
| Marketplace / perfis | Dados de perfil, KYC e pedido conforme acima | `frontend/includes/profile/grpms-profile.php:724-733`, `:780-801`; `backend/fastapi_app/services/checkout/order_service.py:419-454` | Aplicabilidade de regras locais, papéis, bases, direitos, transferências e retenção: **UNVERIFIED** |
| Checkout / afiliados | Processamento de pedido, pagamento e comissões | `backend/fastapi_app/services/checkout/order_service.py:261-281`; `backend/fastapi_app/services/checkout/financial_fee_service.py:7-23` | Requisitos específicos, residência e pagamentos: **UNVERIFIED** |

## África do Sul

| Produto | Modelo operacional observado | Evidência no código | Avaliação jurídica |
|---|---|---|---|
| Marketplace / perfis | Dados de usuário, KYC, endereço e pedido | `frontend/includes/profile/grpms-profile.php:724-733`, `:780-801`; `backend/fastapi_app/services/checkout/order_service.py:419-454` | Aplicabilidade, papéis, direitos, retenção, transferências e controles: **UNVERIFIED** |
| Checkout / afiliados | Método e valores de pagamento; comissões | `backend/fastapi_app/services/checkout/order_service.py:261-281`; `backend/fastapi_app/services/checkout/checkout_service.py:6-25` | Requisitos específicos: **UNVERIFIED** |

## Outros mercados

Países de operação, clientes atendidos, vendedores, localização de processamento e mercados planejados não podem ser inferidos com segurança apenas por arquivos de código. Para cada jurisdição adicional, identificar produto e modelo (marketplace, checkout, vendedor/afiliado, pagamentos e integrações) e avaliar com assessoria local. **UNVERIFIED**.

## Lacunas transversais

Papéis de controlador/operador, bases legais, consentimento, cookies, avisos publicados, subprocessadores, transferências, retenção, resposta a incidentes, avaliações formais e certificações são **UNVERIFIED**. Não se deve tratar esta matriz como determinação legal nem presumir que a análise do Brasil basta para outros mercados.
