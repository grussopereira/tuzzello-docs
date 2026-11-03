# Fluxo Git e limites de publicação

**Propósito:** descrever os limites Git verificados nas instruções dos repositórios e o estado de referência desta documentação.

**Data:** 2026-11-03. Este documento descreve o HEAD de ambos os repositórios nessa data: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`.

## Repositórios e estado consultado

| Repositório | Branch consultada | HEAD | Estado observado |
| --- | --- | --- | --- |
| Frontend `Tuzzello-Frontend-AI-Lab` | `main` | `c1bd75daff802b7ac97bfa32b47f45dd4608d574` | Sem alterações reportadas no status consultado |
| Backend `Tuzzello-Backend-AI-Lab` | `main` | `e53fb27c2d11e137eafba8096617a20105320026` | Havia itens não rastreados preexistentes; nenhum foi alterado |

O checkout frontend identifica `origin` como remoto gravável e `upstream` (repositório `GRP-MultiSeller-Addon-WCFM`) como somente leitura. As instruções desse repositório nomeiam `main` como branch de desenvolvimento e proíbem alterar/push para upstream ou produção (frontend: `.maestri/roles/f6c8d38c-4c74-4a25-9490-48beb15dd4b7/AGENTS.md:38-60`).

## Antes de qualquer edição de código

As instruções do projeto exigem conferir raiz Git, branch ativa, remotos e estado de trabalho antes de alterar arquivos; a branch frontend autorizada é `main` (frontend: `.maestri/roles/f6c8d38c-4c74-4a25-9490-48beb15dd4b7/AGENTS.md:38-49`). O backend foi tratado nesta tarefa como leitura somente, conforme o brief.

## Publicação e operações Git

- Commit, push, merge, rebase, exclusão de branch e deploy exigem autorização explícita (frontend: `.maestri/roles/f6c8d38c-4c74-4a25-9490-48beb15dd4b7/AGENTS.md:54-58`).
- Não usar `git reset --hard` nem `git clean -fd` sem autorização explícita, conforme as instruções do repositório citado acima.
- O procedimento oficial de criação de branches, formato de commits, abertura/revisão de pull request e promoção entre ambientes não está estabelecido pelas fontes consultadas: **UNVERIFIED**.

## Escopo desta entrega

Somente documentos em `/home/gilberto/tuzzello-specs/docs-desenvolvimento` foram criados para esta tarefa. Nenhum comando Git de escrita foi executado.
