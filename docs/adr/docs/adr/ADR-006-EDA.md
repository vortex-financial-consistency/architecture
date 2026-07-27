# ADR-006: Comunicação Assíncrona, Event-Driven Architecture, Transactional Outbox e Idempotência

---

## 1. Contexto e Problema

A **Vortex Engine** é um sistema de alta escala e relevância crítica para processamento de transações financeiras. O isolamento entre módulos (ADR-001/ADR-004) e a consistência eventual exigem que mudanças de estado em um contexto de negócio (ex: aprovação de um pagamento no módulo `payment`) sejam notificadas a outros contextos (ex: movimentação no módulo `ledger` ou envio de Webhooks) sem comprometer o tempo de resposta ou a integridade dos dados.

A implementação ingênua de publicação de eventos enfrenta dois grandes desafios distribuídos:

1. **O Problema da Dupla Escrita (*Dual-Write Problem*):** Tentar gravar o estado do agregado no banco de dados relacional e enviar uma mensagem para um broker (ex: Apache Kafka) dentro do mesmo caso de uso. Se o banco efetuar o *commit* mas a rede falhar ao comunicar com o broker, o evento é perdido. Se a mensagem for enviada ao broker mas a transação do banco sofrer *rollback*, o sistema publica um evento falso.
2. **Duplicação de Mensagens em Redes Distribuídas:** Em um ambiente de rede instável, retransmissões automáticas por falhas temporárias podem fazer com que o mesmo evento seja entregue mais de uma vez ao consumidor, correndo o risco de duplicar débitos, créditos ou notificações financeiras.

---

## 2. Decisão

Decidimos adotar a **Arquitetura Orientada a Eventos (*Event-Driven Architecture - EDA*)** como o padrão oficial para integração assíncrona e desacoplada entre módulos e sistemas externos.

A infraestrutura de comunicação assíncrona é respaldada pelas seguintes definições arquiteturais:

1. **Apache Kafka como Barramento de Eventos:** Utilizado como log distribuído de eventos e mensagens do sistema.
2. **Padrão Transactional Outbox:** Garantia atômica de que nenhum evento de domínio será perdido, eliminando o problema da dupla escrita.
3. **Garantia de Entrega At-Least-Once:** O ecossistema garante que toda mensagem será entregue *pelo menos uma vez* aos consumidores.
4. **Idempotência Obrigatória em Todos os Consumidores:** Todo consumidor de eventos deve implementar controle de idempotência explícito antes de aplicar qualquer efeito colateral.

---

## 3. Detalhamento Técnico e Justificativas

### 3.1. Event-Driven Architecture (EDA)
Eventos de Domínio (`DomainEvent`) representam fatos imutáveis ocorridos no passado (ex: `PaymentApprovedEvent`, `BalanceReservedEvent`). 

* **Desacoplamento Temporal:** O módulo emissor não precisa que o módulo receptor esteja online ou responda no mesmo milissegundo para concluir sua operação.
* **Auditabilidade e Reprocessamento:** Histórico completo das alterações de estado do sistema armazenado no broker.

### 3.2. Resolução do Dual-Write com Transactional Outbox Pattern

Para garantir atomicidade entre a alteração de dados no PostgreSQL e a emissão do evento para o Kafka, o caso de uso **nunca publica mensagens diretamente no Kafka**.

#### Funcionamento:
1. **Transação Única no Banco:** Durante a execução de um caso de uso, a alteração no Agregado e o registro do evento na tabela `outbox_events` são gravados dentro da **mesma transação ACID do PostgreSQL**.
   ```text
   BEGIN TRANSACTION;
   INSERT INTO payments (...) VALUES (...);
   INSERT INTO outbox_events (id, aggregate_type, payload, status) VALUES (...);
   COMMIT;
   ```
2. **Garantia de Persistência:** Se a transação falhar, nada é salvo e nenhum evento é gerado. Se tiver sucesso, o estado e o evento estão garantidos no disco de forma atômica.
3. **Message Relay (Módulo `messaging`):** Um worker assíncrono interno varre periodicamente a tabela `outbox_events` (utilizando índices parciais conforme ADR-005) ou consome a tabela via CDC/Polling, publica as mensagens no Apache Kafka e marca o registro como processado.

### 3.3. Escolha do Apache Kafka
O **Apache Kafka** foi escolhido como o broker da arquitetura devido às seguintes características:

* **Ordenamento Garantido por Partição:** Mensagens enviadas com a mesma chave de partição (`partition_key` = `account_id` ou `payment_id`) são estritamente ordenadas no mesmo *partition*, garantindo que eventos de um mesmo pagamento ou conta sejam processados na ordem cronológica exata.
* **Persistência Durável e Replay:** O Kafka funciona como um log de confirmação (*commit log*) persistido em disco. Caso um módulo passe por manutenção ou falha, ele pode reconectar e reprocessar eventos a partir do seu último *offset* confirmado.
* **Alto Throughput:** Capacidade de absorver picos de tráfego financeiro com baixíssima latência.

### 3.4. Garantia de Entrega: At-Least-Once (Pelo Menos Uma Vez)
Tentar garantir entrega estrita de "Exatamente Uma Vez" (*Exactly-Once*) no nível de infraestrutura de rede distribuída adiciona um custo computacional proibitivo e reduz a resiliência do sistema.

Adotamos a garantia **At-Least-Once**:
* O *Message Relay* garante que toda linha da tabela `outbox_events` será enviada ao Kafka. Em caso de dúvida ou timeout de rede, a mensagem é reenviada.
* O consumidor do Kafka confirma a leitura (*commit de offset*) somente após processar o evento com sucesso. Se o processo do consumidor cair antes da confirmação, o Kafka entregará a mensagem novamente quando o serviço reiniciar.

### 3.5. Resolução da Duplicação: Idempotência Obrigatória
Como a garantia *At-Least-Once* implica que uma mesma mensagem pode ser entregue mais de uma vez em cenários de falha de rede ou rebalanceamento de consumidores, **a responsabilidade de evitar duplicidades pertence ao consumidor**.

#### Estratégia Oficial de Idempotência:
1. **Identificador Único Universal do Evento:** Todo `DomainEvent` possui um `event_id` único gerado na sua origem (UUIDv7).
2. **Tabela de Mensagens Processadas (`processed_messages`):** Antes de executar qualquer regra de negócio, o consumidor tenta registrar o `event_id` na tabela de controle de idempotência do seu módulo.
3. **Chave Primária e Restrição UNIQUE:** Se o `event_id` já existir no banco de dados do consumidor, a tentativa de inserção gera uma violação de chave única (`UniqueConstraintException`), e o consumidor descarta o processamento imediatamente, reconhecendo a mensagem no Kafka sem executar efeitos colaterais duplicados.

---

## 4. Fluxo de Execução da Mensageria

```text
================================================================================
CAMADA DE APLICAÇÃO (ex: módulo payment)
================================================================================
HTTP Request
   │
   ▼
[ PaymentUseCase ]
   │
   ├── (1) Altera estado do Agregado Payment
   ├── (2) Registra evento na tabela `payments`
   └── (3) Registra evento na tabela `outbox_events`
   │
   ▼ (COMMIT ÚNICO NO POSTGRESQL - ACID)

================================================================================
INFRAESTRUTURA DE MENSAGERIA (módulo messaging - assíncrono)
================================================================================
[ Message Relay Worker ]
   │
   ├── (4) Lê registros pendentes da tabela `outbox_events`
   ├── (5) Publica mensagem no Apache Kafka (Partition Key: payment_id)
   └── (6) Atualiza status na tabela `outbox_events` para PROCESSED

================================================================================
CONSUMIDOR DE EVENTOS (ex: módulo ledger)
================================================================================
[ Kafka Consumer Listener ]
   │
   ├── (7) Recebe mensagem do Kafka
   ├── (8) Checa/Insere `event_id` na tabela `processed_messages`
   │       ├── SE JÁ EXISTE: Ignora evento e faz Acknowledge no Kafka
   │       └── SE NÃO EXISTE: Prossegue
   ├── (9) Executa regra de negócio contábil
   └── (10) Realiza Acknowledge (Commit Offset) no Kafka
```

---

## 5. Anti-Patterns Proibidos

1. **Invocação Direta do Kafka no UseCase:** É estritamente proibido injetar `KafkaTemplate` ou publicar mensagens diretamente nos casos de uso do domínio. Todo evento transacional deve passar pela `OutboxPort` / `DomainEventPublisherPort`.
2. **Consumidores Sem Verificação de Idempotência:** É proibido implementar consumidores de eventos do Kafka que alterem estado do banco de dados ou realizem chamadas externas sem o mecanismo de validação por `event_id` / chave de idempotência.
3. **Dependência de Lógica Temporal sem Event ID:** Não confiar em ordenação por *timestamp* da mensagem sem validar o identificador único do evento.
4. **Ignorar Erros de Processamento sem DLQ:** Caso uma mensagem falhe repetidamente por erros não-recuperáveis (ex: erro de *parse* do payload), ela deve ser encaminhada para uma *Dead Letter Queue* (DLQ) para análise posterior, impedindo o travamento da partição do Kafka.

---

## 6. Consequências

### Positivas
* **Zero Perda de Eventos:** O padrão Transactional Outbox garante que nenhum evento publicado pelo domínio seja perdido em falhas de rede ou quedas do broker.
* **Isolamento e Escala:** Módulos e consumidores operam no seu próprio ritmo, absorvendo picos de carga sem derrubar outros componentes.
* **Segurança Financeira:** A obrigatoriedade de idempotência impede pagamentos ou lançamentos duplicados na ocorrência de retransmissões do Kafka.

### Negativas e Limitações
* **Consistência Eventual:** O estado entre diferentes módulos não é atualizado no mesmo milissegundo, exigindo que a interface do usuário e as APIs lidem com estados intermediários (`PENDING`, `PROCESSING`).
* **Complexidade e Overhead de Armazenamento:** Exige a manutenção da tabela `outbox_events` (que necessita de rotinas de limpeza/expurgo de eventos antigos) e tabelas de controle de idempotência `processed_messages` em cada módulo consumidor.
* **Latência de Publicação Assíncrona:** A varredura do *Message Relay* adiciona um pequeno intervalo de latência (milissegundos) entre o *commit* no banco e a chegada da mensagem ao tópico do Kafka.
