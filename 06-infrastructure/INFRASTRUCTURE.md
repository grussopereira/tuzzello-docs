# Infraestrutura

Finalidade: registrar a topologia, redes e pontos de entrada de infraestrutura que podem ser comprovados nos repositórios.

> Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Ambientes descritos no código e na documentação

A documentação da aplicação descreve uma homologação GCP com VMs nomeadas para web e backend, um domínio público de homologação e uma URL interna privada para a comunicação PHP→FastAPI. Isso prova que esses valores são documentados para uma implantação proposta; não prova que os recursos estejam ativos, nem que correspondam à produção atual. [frontend/docs/CLOUD-DEPLOYMENT.md:1-4,42-50] [backend/docs/CLOUD-DEPLOYMENT.md:1-4]

O cliente PHP usa configuração server-side, restringe rotas a uma lista fixa e desativa redirecionamentos ao chamar o backend. [frontend/includes/api/grpms-backend-client.php:27-54]

## DNS, TLS e proxy

**UNVERIFIED:** provedor DNS, zonas e registros efetivamente publicados, certificados TLS, terminação TLS, topologia de Nginx, regras de firewall e endereços atuais. A documentação cita bloqueios Nginx e URLs de homologação como instruções de implantação, sem configuração de servidor versionada para validar o estado real. [backend/docs/CLOUD-DEPLOYMENT.md:30-42]

**UNVERIFIED:** produção em Google Cloud. Os repositórios contêm documentação de homologação GCP, não prova de uma arquitetura de produção ativa.
