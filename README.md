# Documentação do Tuzzello Marketplace

Índice da documentação técnica e de produto do marketplace Tuzzello.

**Data:** 2026-11-03.

A documentação descreve o HEAD do frontend `c1bd75d` e do backend `e53fb27` em 2026-11-03.

## Seções

| Seção | Conteúdo | Arquivos | Status indicado nos documentos |
| --- | --- | ---: | --- |
| [01-architecture](./01-architecture/) | Arquitetura do sistema, componentes, decisões, fluxos de dados e limites de integração. | 6 | UNVERIFIED |
| [02-security](./02-security/) | Segurança, autenticação, autorização, segredos, auditoria e resposta a incidentes. | 9 | UNVERIFIED |
| [03-api-integrations](./03-api-integrations/) | APIs, autenticação, versionamento, integrações, webhooks e especificação OpenAPI. | 6 | UNVERIFIED |
| [04-database](./04-database/) | Arquitetura de dados, dicionário, integridade, retenção, migrações e consultas. | 7 | UNVERIFIED |
| [05-finance](./05-finance/) | Pagamentos, comissões, tarifas, ledger, payouts, reconciliação, reembolsos e auditoria financeira. | 8 | UNVERIFIED |
| [06-infrastructure](./06-infrastructure/) | Infraestrutura, ambientes, implantação, CI/CD, observabilidade, backup e recuperação. | 9 | UNVERIFIED |
| [07-business-rules](./07-business-rules/) | Regras do marketplace e ciclos de vida de pedidos, produtos, vendedores e afiliados. | 8 | DRAFT, UNVERIFIED |
| [08-development](./08-development/) | Orientações de desenvolvimento, padrões, arquitetura frontend, dependências e fluxo Git. | 7 | UNVERIFIED |
| [09-quality-assurance](./09-quality-assurance/) | Estratégia, critérios, casos e tipos de teste e garantia de qualidade. | 7 | UNVERIFIED |
| [10-operations](./10-operations/) | Runbooks, incidentes, manutenção, monitoramento, troubleshooting e postmortem. | 6 | UNVERIFIED |
| [11-compliance](./11-compliance/) | Privacidade, tratamento de dados, direitos de titulares, conformidade e termos. | 7 | UNVERIFIED |
| [12-product-design](./12-product-design/) | Requisitos, catálogo, jornadas, sistema de design, acessibilidade e roadmap. | 7 | UNVERIFIED |

### [01-architecture](./01-architecture/)

- [ARCHITECTURE-DECISIONS.md](./01-architecture/ARCHITECTURE-DECISIONS.md)
- [C4-DIAGRAMS.md](./01-architecture/C4-DIAGRAMS.md)
- [COMPONENT-CATALOG.md](./01-architecture/COMPONENT-CATALOG.md)
- [DATA-FLOW.md](./01-architecture/DATA-FLOW.md)
- [INTEGRATION-BOUNDARIES.md](./01-architecture/INTEGRATION-BOUNDARIES.md)
- [SYSTEM-ARCHITECTURE.md](./01-architecture/SYSTEM-ARCHITECTURE.md)

### [02-security](./02-security/)

- [API-SECURITY.md](./02-security/API-SECURITY.md)
- [AUDIT-LOGGING.md](./02-security/AUDIT-LOGGING.md)
- [AUTHENTICATION.md](./02-security/AUTHENTICATION.md)
- [AUTHORIZATION-RBAC.md](./02-security/AUTHORIZATION-RBAC.md)
- [INCIDENT-RESPONSE.md](./02-security/INCIDENT-RESPONSE.md)
- [SECRETS-MANAGEMENT.md](./02-security/SECRETS-MANAGEMENT.md)
- [SECURITY-ARCHITECTURE.md](./02-security/SECURITY-ARCHITECTURE.md)
- [THREAT-MODEL.md](./02-security/THREAT-MODEL.md)
- [VULNERABILITY-MANAGEMENT.md](./02-security/VULNERABILITY-MANAGEMENT.md)

### [03-api-integrations](./03-api-integrations/)

- [API-AUTHENTICATION.md](./03-api-integrations/API-AUTHENTICATION.md)
- [API-OVERVIEW.md](./03-api-integrations/API-OVERVIEW.md)
- [API-VERSIONING.md](./03-api-integrations/API-VERSIONING.md)
- [INTEGRATIONS.md](./03-api-integrations/INTEGRATIONS.md)
- [OPENAPI.yaml](./03-api-integrations/OPENAPI.yaml)
- [WEBHOOKS.md](./03-api-integrations/WEBHOOKS.md)

### [04-database](./04-database/)

- [DATA-DICTIONARY.md](./04-database/DATA-DICTIONARY.md)
- [DATA-INTEGRITY.md](./04-database/DATA-INTEGRITY.md)
- [DATA-RETENTION.md](./04-database/DATA-RETENTION.md)
- [DATABASE-ARCHITECTURE.md](./04-database/DATABASE-ARCHITECTURE.md)
- [DATABASE-MIGRATIONS.md](./04-database/DATABASE-MIGRATIONS.md)
- [ER-DIAGRAM.md](./04-database/ER-DIAGRAM.md)
- [QUERY-OPTIMIZATION.md](./04-database/QUERY-OPTIMIZATION.md)

### [05-finance](./05-finance/)

- [COMMISSION-RULES.md](./05-finance/COMMISSION-RULES.md)
- [FEES-AND-TAXES.md](./05-finance/FEES-AND-TAXES.md)
- [FINANCIAL-AUDIT.md](./05-finance/FINANCIAL-AUDIT.md)
- [FINANCIAL-LEDGER.md](./05-finance/FINANCIAL-LEDGER.md)
- [PAYMENT-FLOW.md](./05-finance/PAYMENT-FLOW.md)
- [PAYOUTS.md](./05-finance/PAYOUTS.md)
- [RECONCILIATION.md](./05-finance/RECONCILIATION.md)
- [REFUNDS-CHARGEBACKS.md](./05-finance/REFUNDS-CHARGEBACKS.md)

### [06-infrastructure](./06-infrastructure/)

- [BACKUP-RESTORE.md](./06-infrastructure/BACKUP-RESTORE.md)
- [CI-CD.md](./06-infrastructure/CI-CD.md)
- [CONTAINERS.md](./06-infrastructure/CONTAINERS.md)
- [DEPLOYMENT.md](./06-infrastructure/DEPLOYMENT.md)
- [DISASTER-RECOVERY.md](./06-infrastructure/DISASTER-RECOVERY.md)
- [ENVIRONMENTS.md](./06-infrastructure/ENVIRONMENTS.md)
- [INFRASTRUCTURE.md](./06-infrastructure/INFRASTRUCTURE.md)
- [OBSERVABILITY.md](./06-infrastructure/OBSERVABILITY.md)
- [ROLLBACK.md](./06-infrastructure/ROLLBACK.md)

### [07-business-rules](./07-business-rules/)

- [ACCESS-POLICIES.md](./07-business-rules/ACCESS-POLICIES.md)
- [AFFILIATE-LIFECYCLE.md](./07-business-rules/AFFILIATE-LIFECYCLE.md)
- [BUSINESS-EXCEPTIONS.md](./07-business-rules/BUSINESS-EXCEPTIONS.md)
- [MARKETPLACE-RULES.md](./07-business-rules/MARKETPLACE-RULES.md)
- [ORDER-LIFECYCLE.md](./07-business-rules/ORDER-LIFECYCLE.md)
- [PRODUCT-LIFECYCLE.md](./07-business-rules/PRODUCT-LIFECYCLE.md)
- [SHIPPING-RULES.md](./07-business-rules/SHIPPING-RULES.md)
- [VENDOR-LIFECYCLE.md](./07-business-rules/VENDOR-LIFECYCLE.md)

### [08-development](./08-development/)

- [AI-AGENT-GUIDELINES.md](./08-development/AI-AGENT-GUIDELINES.md)
- [CODING-STANDARDS.md](./08-development/CODING-STANDARDS.md)
- [DEPENDENCY-MANAGEMENT.md](./08-development/DEPENDENCY-MANAGEMENT.md)
- [DEVELOPMENT-GUIDELINES.md](./08-development/DEVELOPMENT-GUIDELINES.md)
- [FRONTEND-ARCHITECTURE.md](./08-development/FRONTEND-ARCHITECTURE.md)
- [GIT-WORKFLOW.md](./08-development/GIT-WORKFLOW.md)
- [WORDPRESS-INTEGRATION.md](./08-development/WORDPRESS-INTEGRATION.md)

### [09-quality-assurance](./09-quality-assurance/)

- [ACCEPTANCE-CRITERIA.md](./09-quality-assurance/ACCEPTANCE-CRITERIA.md)
- [INTEGRATION-TESTS.md](./09-quality-assurance/INTEGRATION-TESTS.md)
- [PERFORMANCE-TESTS.md](./09-quality-assurance/PERFORMANCE-TESTS.md)
- [REGRESSION-TESTS.md](./09-quality-assurance/REGRESSION-TESTS.md)
- [SECURITY-TESTS.md](./09-quality-assurance/SECURITY-TESTS.md)
- [TEST-CASES.md](./09-quality-assurance/TEST-CASES.md)
- [TEST-STRATEGY.md](./09-quality-assurance/TEST-STRATEGY.md)

### [10-operations](./10-operations/)

- [INCIDENT-MANAGEMENT.md](./10-operations/INCIDENT-MANAGEMENT.md)
- [MAINTENANCE.md](./10-operations/MAINTENANCE.md)
- [MONITORING-ALERTS.md](./10-operations/MONITORING-ALERTS.md)
- [OPERATIONS-RUNBOOK.md](./10-operations/OPERATIONS-RUNBOOK.md)
- [POSTMORTEM.md](./10-operations/POSTMORTEM.md)
- [TROUBLESHOOTING.md](./10-operations/TROUBLESHOOTING.md)

### [11-compliance](./11-compliance/)

- [COMPLIANCE-MATRIX.md](./11-compliance/COMPLIANCE-MATRIX.md)
- [DATA-PROCESSING.md](./11-compliance/DATA-PROCESSING.md)
- [DATA-RETENTION.md](./11-compliance/DATA-RETENTION.md)
- [DATA-SUBJECT-RIGHTS.md](./11-compliance/DATA-SUBJECT-RIGHTS.md)
- [PAYMENT-COMPLIANCE.md](./11-compliance/PAYMENT-COMPLIANCE.md)
- [PRIVACY-POLICY.md](./11-compliance/PRIVACY-POLICY.md)
- [SELLER-AFFILIATE-TERMS.md](./11-compliance/SELLER-AFFILIATE-TERMS.md)

### [12-product-design](./12-product-design/)

- [ACCESSIBILITY.md](./12-product-design/ACCESSIBILITY.md)
- [CHANGELOG.md](./12-product-design/CHANGELOG.md)
- [DESIGN-SYSTEM.md](./12-product-design/DESIGN-SYSTEM.md)
- [FEATURE-CATALOG.md](./12-product-design/FEATURE-CATALOG.md)
- [PRODUCT-REQUIREMENTS.md](./12-product-design/PRODUCT-REQUIREMENTS.md)
- [PRODUCT-ROADMAP.md](./12-product-design/PRODUCT-ROADMAP.md)
- [USER-JOURNEYS.md](./12-product-design/USER-JOURNEYS.md)
