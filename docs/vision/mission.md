# Mission

---

## Missão

Construir uma engine de pagamentos **à prova de falhas**, projetada para **eliminar perdas financeiras** causadas por:

* Erros de software
* Instabilidades de rede
* Quedas de serviços parceiros

---

## Visão de Longo Prazo

Tornar este projeto o **modelo de referência arquitetural** para o desenvolvimento de sistemas financeiros distribuídos. 

O objetivo final é entregar uma aplicação que sirva de estudo prático sobre como unir:

* **Resiliência**
* **Consistência de dados**
* **Independência de infraestrutura**

---

## Objetivos Estratégicos

* **Cobrança Única:** Processar cada transação exatamente uma vez.
* **Proteção de Saldo:** Bloquear débitos inconsistentes ou saldos negativos em acessos simultâneos.
* **Zero Perda de Eventos:** Manter o banco de dados e a mensageria sempre sincronizados.
* **Isolamento de Falhas:** Impedir que a lentidão de terceiros derrube o sistema.

---

## Princípios de Engenharia (Invioláveis)

Estes princípios orientam todas as decisões técnicas e **não serão negociados**:

1. **Corretude antes da velocidade**  
   O sistema precisa estar correto antes de ser rápido. Um atraso é aceitável; uma cobrança duplicada, não.

2. **Domínio isolado**  
   A lógica de negócio financeira não pode depender de frameworks, bancos de dados ou bibliotecas externas.

3. **Falha segura (Fail-Safe)**  
   Na dúvida ou em cenários imprevistos, o sistema deve interromper a operação e proteger o dinheiro.

4. **Observabilidade nativa**  
   Toda transação deve ser rastreável do início ao fim. Se não pode ser auditado, não vai para produção.

5. **Clareza sobre esperteza**  
   Código simples e legível sempre tem prioridade sobre soluções complexas e difíceis de manter.

---

## Critérios de Sucesso

O projeto será considerado concluído com sucesso quando atingir estes indicadores:

* **0% de cobranças duplicadas:** Nenhuma requisição reexecutada sob estresse.
* **0% de gasto duplo:** Saldo correto mesmo em chamadas concorrentes no mesmo milissegundo.
* **100% de eventos integrados:** Zero divergência entre o banco de dados e a fila de notificações.
* **Contenção de erros de terceiros:** Isolamento imediato de APIs parceiras em falha, sem travar a aplicação.
* **Auditabilidade total:** Capacidade de rastrear o histórico completo de qualquer transação por um ID único.
