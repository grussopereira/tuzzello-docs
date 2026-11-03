# Acessibilidade

Propósito: inventariar evidências de suporte a acessibilidade no frontend e lacunas que exigem verificação dedicada.

> Documento de referência do HEAD dos repositórios frontend e backend em 2026-11-03.

## Evidências no código

- Os gráficos são renderizados em elementos `<canvas>` com dimensões, mas alternativas textuais equivalentes não são demonstradas pelo trecho do template PHP. (`frontend/includes/dashboard/grpms-dashboard.php:22`, `:35`, `:48`) Alternativa textual: **UNVERIFIED**.
- O seletor de período tem rótulos associados visualmente e estilo de foco visível para input em foco. (`frontend/includes/dashboard/css/dashboard.css:30-50`, `:416`)
- O CTA de dashboard vazio possui estilo `:focus-visible`. (`frontend/includes/dashboard/css/dashboard.css:429`)
- O template de preview de produto usa `aria-label`, `aria-hidden`, papéis de tab/tablist/tabpanel e estados selecionados. (`frontend/templates/product-public-preview.php:30-64`)
- O CSS do assistente desativa animação de indicador de digitação com `prefers-reduced-motion: reduce`. (`frontend/includes/general/grpms-general.php:317`)

## Lacunas

Conformidade WCAG, navegação integral por teclado, contraste medido, anúncio de erros, testes com leitores de tela, zoom/reflow e auditoria de todos os fluxos: **UNVERIFIED**. Os exemplos acima não comprovam conformidade global.
