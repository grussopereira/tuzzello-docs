# Observabilidade

Finalidade: descrever métricas, logs, tracing e alertas operacionais declarados em configuração versionada.

> Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Evidência disponível

As instruções de homologação dizem para verificar logs e uma resposta HTTP local após reiniciar o backend; isso é uma etapa de validação manual, não uma definição de monitoramento. [backend/docs/CLOUD-DEPLOYMENT.md:54-60]

A aplicação monta routers FastAPI e configura CORS, mas isso não define por si só métricas, tracing ou alertas. [backend/fastapi_app/main.py:23-48]

## Lacunas

**UNVERIFIED:** plataforma e retenção de logs, métricas, tracing, healthchecks, dashboards, alertas, SLOs e escalonamento de incidentes.
