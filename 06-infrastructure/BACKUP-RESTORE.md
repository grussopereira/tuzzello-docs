# Backup e restauração

Finalidade: identificar dados e arquivos cuja proteção é mencionada nas instruções de implantação e apontar procedimentos não especificados.

> Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Orientação encontrada

O documento do backend orienta obter backup privado da instalação e dump consistente antes da implantação; também instrui a não restaurar o dump por cima de pedidos posteriores sem reconciliação. [backend/docs/CLOUD-DEPLOYMENT.md:22-32,78-83]

O documento do frontend também menciona backup do plugin, banco e configurações antes de publicação. [frontend/docs/CLOUD-DEPLOYMENT.md:50-52]

## Lacunas

**UNVERIFIED:** ferramenta e comandos de backup/restore, escopo de uploads e arquivos, armazenamento e criptografia, frequência, retenção, validação de integridade, volumes Podman e teste de restauração. Não foi executado SQL nem conexão com banco.
