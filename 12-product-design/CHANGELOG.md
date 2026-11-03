# Registro de alterações

Propósito: resumir alterações recentes registradas no histórico Git dos dois repositórios.

> Documento de referência do HEAD dos repositórios frontend e backend em 2026-11-03.

## 2026-11-03

- **Backend (`e53fb27`)** — EasyPost exige identidade de vendedor assinada e checagem de ownership, conforme assunto do commit.

## 2026-11-02

- **Frontend (`c1bd75d`)** — ajuste visual de ribbon/badge na galeria de produtos.
- **Backend (`9e3c8df`)** — testes e referência de grants para alertas.
- **Backend (`4aa4708`)** — valida configuração de alerta no início do worker.
- **Backend (`55bcfa0`)** — alerta por e-mail para gerente da loja quando há falha de envio ao WooCommerce.
- **Backend (`9531fa1`, `a66903d`)** — segurança de destinos e envio de status para WooCommerce descritos nos assuntos dos commits.

## 2026-11-01

- **Frontend (`4cc8f6b`)** — gerenciamento de responsabilidade tributária e UI.
- **Frontend (`3ac4060`)** — interrompe o envio de “Order was done!” na conclusão.
- **Frontend (`d64d0e9`)** — exibe mensagem do servidor em vez de `undefined` na aprovação de pedidos.
- **Backend (`e67cec5`, `1e83f40`)** — commits sobre gate de conclusão automática desligado/não conectado e webhook de pedido assinado.
- **Backend (`ce5f85e`)** — ledger de reembolsos descrito como não conectado e restrito ao laboratório no assunto do commit.

Os resumos refletem apenas assuntos do `git log`; não substituem revisão de diff e não afirmam disponibilidade em produção.
