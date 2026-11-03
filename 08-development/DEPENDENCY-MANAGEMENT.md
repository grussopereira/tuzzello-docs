# Gerenciamento de dependências

**Propósito:** descrever manifestos e dependências confirmados no HEAD; não prescrever atualizações não documentadas.

**Data:** 2026-11-03. Este documento descreve o HEAD de ambos os repositórios nessa data: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`.

## Backend

- `requirements.txt` fixa versões de FastAPI, Uvicorn, Pydantic, dotenv, autenticação JWT, conectores MySQL/PostgreSQL, `phpserialize`, multipart e EasyPost (backend: `requirements.txt:1-14`).
- `requirements-dev.txt` inclui o manifesto principal e fixa pytest e httpx (backend: `requirements-dev.txt:1-3`). `requirements-cloud.txt` repete as dependências runtime em vez de referenciar o principal (backend: `requirements-cloud.txt:1-14`).
- `README-BE.md` contém uma lista antiga não fixada e grafia divergente do nome do arquivo de requirements. Para versões presentes, o manifesto versionado é a fonte verificável; README histórico: frontend `README-BE.md:1-20`.
- O arquivo `.env.example` existe, mas os valores de ambiente não foram reproduzidos. `main.py` carrega dotenv e lê `CORS_ALLOWED_ORIGINS`; a fábrica de conexão consulta `DB_ENGINE` e parâmetros de conexão de MySQL/PostgreSQL (backend: `fastapi_app/main.py:1-7,26-32`; `fastapi_app/utils/database.py:8-30`).
- O comando de instalação aparece no README histórico (`pip install -r requirements.txt`, frontend: `README-BE.md:18-20`), mas um procedimento atual e completo de criação/ativação de ambiente isolado não foi confirmado: **UNVERIFIED**.

## Frontend WordPress

- Não foi encontrado `composer.json` ou `package.json` no checkout analisado. O código importa dependências via ambiente WordPress e plugins instalados; o plugin registra integração com WCFM e WooCommerce em código distribuído por módulos.
- `README-FE.md` enumera versões/requisitos de WordPress e plugins (frontend: `README-FE.md:11-25`), mas essa lista não é sustentada por um manifesto de dependências ou verificação automática no código atual. Compatibilidade mínima vigente: **UNVERIFIED**.
- `includes/training/grpms-training.php` enfileira player externo via URL e ativos locais com versão baseada em `filemtime` (frontend: `includes/training/grpms-training.php:129-163`). Essa referência não estabelece política geral de fornecedores ou atualização de ativos.

## Atualizações

Política de auditoria, atualização, lockfiles, resolução de vulnerabilidades e aprovação de atualizações: **UNVERIFIED**. Não foram executadas instalações nem atualizações durante a documentação.
