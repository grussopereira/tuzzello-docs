# Ambientes

Finalidade: separar diferenças de ambiente comprovadas e identificar lacunas entre laboratório, homologação e produção.

> Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Laboratório local

Há evidência de uma rotina de gate executável localmente no backend, que varre artefatos e valida consistência de uma alteração SQL; o próprio script declara não ser enforcement de CI. [backend/scripts/check_b_t6_gate.sh:1-17]

**UNVERIFIED:** definição reproduzível de laboratório Podman. O arquivo `backend/podman` está vazio, e o inventário consultado não revelou compose, Containerfile ou Dockerfile.

## Homologação documentada

Os documentos de implantação descrevem uma homologação GCP, com uma VM web e outra de backend, e recomendam coordenação de branches específicas. Essas instruções não comprovam que o ambiente esteja provisionado ou acessível. [frontend/docs/CLOUD-DEPLOYMENT.md:1-4] [backend/docs/CLOUD-DEPLOYMENT.md:1-5]

## Produção

**UNVERIFIED:** provedor, topologia, endereços, diferenças de configuração, processo de promoção e relação com os ambientes de desenvolvimento/homologação. Não inferir produção GCP a partir de documentos de homologação.
