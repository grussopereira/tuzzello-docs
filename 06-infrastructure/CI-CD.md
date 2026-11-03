# CI/CD

Finalidade: descrever validações automatizadas, versionamento e promoção entre ambientes segundo a configuração versionada.

> Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Validação encontrada

O script `scripts/check_b_t6_gate.sh` chama-se gate de etapa de release e declara explicitamente que não há enforcement de CI. Ele verifica arquivos e condições relacionadas à migração B-T6; não é uma pipeline geral dos projetos. [backend/scripts/check_b_t6_gate.sh:1-17]

Os documentos de homologação mencionam testes locais e validações manuais nas VMs; são instruções documentadas e não registros de execução. [frontend/docs/CLOUD-DEPLOYMENT.md:85-95] [backend/docs/CLOUD-DEPLOYMENT.md:62-76]

**UNVERIFIED:** pipelines CI, gatilhos, runners, matriz de validação, política de tags/versões, aprovação de release, promoção entre ambientes e automação de publicação.
