# Diretrizes para agentes Maestri

**Propósito:** resumir responsabilidades e critérios de revisão registrados nas instruções de agentes do repositório.

**Data:** 2026-11-03. Este documento descreve o HEAD de ambos os repositórios nessa data: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`.

## Papéis e responsabilidades

- **Code Analyst / Technical Writer:** investigação direcionada somente leitura, coleta de evidências com caminho/linha, separação de evidência de hipótese e produção de documentação quando solicitada; análise não substitui validação arquitetural (frontend: `.maestri/roles/ba67a20f-145f-4977-8880-7e71ef52359c/AGENTS.md:42-47`).
- **Software Architect:** avalia arquitetura, integrações, interfaces, autenticação/autorização, operações financeiras e compatibilidade; produz ADRs para decisões significativas e distingue arquitetura existente de proposta (frontend: `.maestri/roles/dca422d4-8391-4fb0-97c4-70788021f145/AGENTS.md:4-24`).
- **Software Engineer:** implementa correções aprovadas pelo arquiteto, executa verificações/testes pertinentes e fornece diff/resultados; mudança material de abordagem deve retornar ao arquiteto (frontend: `.maestri/roles/ba67a20f-145f-4977-8880-7e71ef52359c/AGENTS.md:53-56`).
- **Senior Code Reviewer:** revisão independente do diff real, alinhamento arquitetural, casos extremos, regressões, segurança, comportamento visível e evidência de testes (frontend: `.maestri/roles/ba67a20f-145f-4977-8880-7e71ef52359c/AGENTS.md:57-60`).

## Critérios e formato da revisão

As instruções do revisor pedem verificação de defeitos/regressões/segurança, ciclo WordPress e WooCommerce, preservação de lógica de negócio, contratos API, estruturas de banco e comportamento multimoeda, além de cobertura, casos extremos e compatibilidade retroativa (frontend: `.maestri/roles/efe57e53-ad68-47c4-a7ca-18f36fea159b/AGENTS.md:6-12`). O resultado requerido é um veredito `APPROVED`, `CHANGES REQUESTED` ou `INSUFFICIENT EVIDENCE`, achados ordenados por severidade, caminhos/linhas, avaliação de regressão/segurança/testes e ações necessárias (frontend: `.maestri/roles/efe57e53-ad68-47c4-a7ca-18f36fea159b/AGENTS.md:23-29`). Outra instrução de coordenação também admite `BLOCKED` em sua lista de vereditos para revisão independente (frontend: `.maestri/roles/ba67a20f-145f-4977-8880-7e71ef52359c/AGENTS.md:57-60`).

## Limites

O analista/escritor não emite aprovação arquitetural. O revisor deve ser independente do autor da alteração e seu parecer deve avaliar o diff e as evidências de fato, não presumir aprovação a partir da conclusão do implementador (fontes acima). As responsabilidades e critérios nesta página refletem instruções locais de agentes; não são apresentados como política geral de engenharia independente dessas instruções.

Procedimento de seleção/configuração de modelos e processo completo de criação/encerramento de agentes: **UNVERIFIED** neste resumo, pois não são necessários para os papéis e critérios técnicos solicitados.
