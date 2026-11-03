# Política de Privacidade — rascunho

**Propósito:** descrever, a partir do código, categorias de dados que os componentes revisados recebem ou mantêm.

**Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nesta data.

> **RASCUNHO:** todo conteúdo legal está sujeito à revisão por profissionais jurídicos qualificados em cada jurisdição.

## Escopo e inventário observado

O inventário abaixo limita-se aos campos e operações visíveis no código; não representa todos os dados da plataforma WordPress/WooCommerce nem confirma dados efetivamente existentes em produção.

- **Perfil e contato:** endereço de cobrança, país, telefone e preferências são atualizados em metadados do usuário. O código lista e grava endereço/telefone em `includes/profile/grpms-profile.php:827-887`.
- **Verificação de identidade (KYC):** o perfil aceita imagens de documento frente/verso, selfie e comprovante de endereço; grava referências de arquivos e metadados de validade/tipo. `includes/profile/grpms-profile.php:724-733`, `:780-801`, `:898-903`.
- **Arquivos KYC:** há função de armazenamento privado e proteção contra fallback a armazenamento público; os tipos MIME aceitos estão no código. `includes/profile/profile-kyc-files.php:13-15`, `:25-33`. A configuração real do servidor e a exposição de backups são **UNVERIFIED**.
- **Pedidos:** o backend persiste identificadores de cliente/pedido, método de pagamento, valores, tributos, comissões, status e IDs relacionados. `fastapi_app/services/checkout/order_service.py:261-281`.
- **Endereços associados ao pedido:** nome, empresa, endereço, cidade, estado, CEP, país; e-mail e telefone são persistidos para endereço de cobrança. `fastapi_app/services/checkout/order_service.py:419-454`.
- **Dados financeiros e afiliados:** são processados IDs de transação e valor de taxa de WooPayments, gravados com a indicação de método `card`; também existem IDs e comissões de afiliado nos registros de pedido. `fastapi_app/services/checkout/financial_fee_service.py:7-23`, `:37-49`; `fastapi_app/services/checkout/order_service.py:262-281`.

## Uso, compartilhamento e direitos

O código revisado mostra usos operacionais de dados de pedidos, perfil e verificação para fluxos de marketplace, pagamento e controle de acesso. Não permite confirmar controlador/operador por entidade, bases legais, destinatários contratuais, transferências internacionais, consentimento, direitos ou canal para solicitações. Esses pontos são **UNVERIFIED** e exigem definição e validação jurídica por jurisdição.

## Controles e limites

O armazenamento privado de arquivos KYC e a checagem de permissão de revisor estão implementados em `includes/profile/profile-kyc-files.php:21-33`; isso não comprova conformidade integral, criptografia em repouso, controle de backup, monitoramento ou configuração de produção. **UNVERIFIED**.

## Questões pendentes

Confirmar com responsáveis e assessoria jurídica: entidades e papéis; finalidades e bases; integrações e suboperadores; transferências; cookies/telemetria; canais de direitos; medidas de segurança implantadas; e textos públicos vigentes. Não inferir estes elementos apenas do código.
