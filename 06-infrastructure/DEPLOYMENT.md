# Implantação

Finalidade: resumir o procedimento de implantação descrito nos documentos versionados, sem executá-lo.

> Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Homologação GCP documentada

A documentação descreve deploy coordenado do plugin WordPress e do backend em VMs novas, exportação do código de uma revisão específica e preservação separada das configurações privadas. Trata-se de procedimento documentado para homologação; não confirma implantação atual nem autoriza sua execução. [frontend/docs/CLOUD-DEPLOYMENT.md:42-83] [backend/docs/CLOUD-DEPLOYMENT.md:30-60]

O frontend chama operações backend por URL interna configurada server-side, com cabeçalho de autenticação e rotas permitidas explícitas. Valores de configuração sensíveis não são reproduzidos aqui. [frontend/includes/api/grpms-backend-client.php:27-54]

## Desenvolvimento local

**UNVERIFIED:** comandos completos e procedimento suportado para implantar frontend e backend no laboratório local. Não há manifesto de contêiner identificado neste HEAD.

## Produção

**UNVERIFIED:** destino, processo de aprovação e promoção para produção. As instruções citadas são de homologação e não comprovam produção GCP.
