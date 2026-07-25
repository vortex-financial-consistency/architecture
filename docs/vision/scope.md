
# Escopo da Engine de Pagamentos (SCOPE)

---

## 1. Processamento e Orquestração de Pagamentos

* **Orquestração de Fluxo:** Gerenciamento do ciclo de vida completo do pagamento (Criado $\rightarrow$ Validado $\rightarrow$ Autorizado $\rightarrow$ Liquidado ou Recusado).
* **Validação de Entrada:** Checagem de dados obrigatórios da requisição antes de iniciar o processamento.
* **Simulação de Antifraude:** Integração com serviço simulado para validação de risco com limites de tempo de resposta.

---

## 2. Integridade Financeira e Ledger

* **Validação de Saldo:** Verificação de saldo disponível na conta antes de aprovar qualquer débito.
* **Garantia de Não-Concorrência:** Bloqueio de conta durante a transação para evitar saldo negativo por chamadas simultâneas (Gasto Duplo).
* **Ledger (Livro-Razão):** Registro imutável de cada movimentação (crédito e débito) garantindo a consistência das contas.

---

## 3. Resiliência e Idempotência

* **Controle de Idempotência:** Intercepção de requisições repetidas usando chave única (`Idempotency-Key`), retornando o resultado original sem reprocessar o débito.
* **Isolamento de Falhas:** Uso de *Circuit Breaker* e *Timeouts* para interromper chamadas a parceiros externos simulados quando estiverem lentos ou indisponíveis.

---

## 4. Mensageria e Notificação Assíncrona

* **Garantia de Publicação:** Sincronização entre a alteração no banco de dados e o envio da mensagem para a fila (sem perda de eventos).
* **Publicação de Eventos:** Envio de notificações sobre mudanças de estado do pagamento (`PagamentoAprovado`, `PagamentoRecusado`).
* **Tratamento de Mensagens Com Falha:** Encaminhamento de eventos não processados para uma fila de recuperação (*Dead Letter Queue*).

---

## 5. Auditoria e Observabilidade

* **Identificador Único:** Atribuição de um código de rastreio (`Correlation ID`) para acompanhar o pagamento do início ao fim.
* **Trilha de Auditoria:** Registro histórico com data, hora e motivo de cada mudança de estado da transação.
* **Métricas do Sistema:** Exposição de dados de saúde da aplicação (quantidade de pagamentos, taxa de erros e tempo de resposta).
