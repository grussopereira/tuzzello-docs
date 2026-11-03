# Rollback

Finalidade: registrar as reversões descritas no repositório e os limites de reversão de migrações incompatíveis.

> Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Rollback de aplicação em homologação

A documentação orienta restaurar a versão anterior da VM nova, preservando configurações e chaves locais; também alerta para não restaurar dump sobre pedidos posteriores sem reconciliação. É orientação escrita, não procedimento verificado em execução. [backend/docs/CLOUD-DEPLOYMENT.md:78-83] [frontend/docs/CLOUD-DEPLOYMENT.md:97-98]

## Migração B-T6

O SQL de reversão documenta precondições, bloqueio de tráfego, verificação de duplicidades e risco de retorno de defeito funcional. Exige aprovação externa e declara não executado; não rodar como parte desta documentação. [backend/sql/b_t6/003_rollback_restore_uq_order_product.sql:1-35]

O script de gate é parte das condições descritas para essa migração, mas não se apresenta como CI. [backend/scripts/check_b_t6_gate.sh:1-17]

**UNVERIFIED:** procedimento geral de rollback de releases, retenção de artefatos e restauração automatizada.
