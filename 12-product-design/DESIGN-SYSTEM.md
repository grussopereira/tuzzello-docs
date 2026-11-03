# Sistema visual

Propósito: documentar somente estilos e componentes reutilizáveis observados no CSS e templates frontend.

> Documento de referência do HEAD dos repositórios frontend e backend em 2026-11-03.

## Paleta observada

O painel declara fundo claro `#f5f7fa`, texto `#333`, acento `#47767E` e acento hover `#f35b04`; links e ícones usam o acento. (`frontend/includes/dash/css/dash.css:13-32`) A folha do dashboard de métricas também usa `#47767E` em títulos e controles e `#f35b04` em detalhes do hero. (`frontend/includes/dashboard/css/dashboard.css:15-17`, `:187-205`) Não há evidência nesses trechos de uma especificação de paleta global única; os valores descrevem os estilos locais.

## Tipografia

O painel define `font-family: 'Poppins', sans-serif`. (`frontend/includes/dash/css/dash.css:40-46`) A folha do dashboard também aplica Poppins aos rótulos do seletor de período. (`frontend/includes/dashboard/css/dashboard.css:40-45`) Fonte efetivamente carregada e pesos disponibilizados: **UNVERIFIED**.

## Layout responsivo

- A largura da sidebar usa `min(20vw, 275px)`. (`frontend/includes/dash/css/dash.css:13-19`)
- A grade de gráficos vira uma coluna única em telas até 900px. (`frontend/includes/dashboard/css/dashboard.css:168-172`)
- O painel declara no CSS comportamento mobile que oculta o menu lateral e expande o conteúdo; também há regras específicas em breakpoints próximos de 980px e 1180px. (`frontend/includes/dash/css/dash.css:1-8`, `:2517-2523`)

## Componentes reutilizáveis do dashboard

- Topbar fixa com marca e área de ações. (`frontend/includes/dash/css/dash.css:49-90`)
- Container de estatísticas em cartões flexíveis com quebra de linha. (`frontend/includes/dashboard/css/dashboard.css:53-69`)
- Hero do dashboard com fundo em gradiente, borda e conteúdo flexível. (`frontend/includes/dashboard/css/dashboard.css:187-200`)
- Shortcodes de gráficos de vendas, produtos e ganhos de afiliado. (`frontend/includes/dashboard/grpms-dashboard.php:12-49`)

Este inventário não define tokens ou componentes fora desses arquivos CSS/templates.
