# ADR-005: Estratégia de Persistência e Banco de Dados Relacional (PostgreSQL)

* **Status:** Aceito
* **Data:** 27 de Julho de 2026
* **Decisores:** Time de Arquitetura e Engenharia Vortex Engine
* **Contexto Técnico:** Monólito Modular (ADR-001) / Arquitetura Hexagonal (ADR-002) / Spring Boot (ADR-003) / Module Map (ADR-004)

---

## 1. Contexto e Problema

A **Vortex Engine** atua no núcleo do processamento de pagamentos e liquidação financeira. Nesse domínio, a integridade dos dados e a confiabilidade das transações são requisitos não-negociáveis. A perda de uma transação, uma atualização inconsistente de saldo (*Lost Update*) ou a ausência de rastreabilidade contábil geram prejuízos financeiros diretos e inconformidades regulatórias.

Precisamos definir a estratégia oficial de banco de dados e persistência do sistema, respondendo às seguintes necessidades:

1. **Garantias Transacionais Estritas:** Operações de crédito e débito (*double-entry bookkeeping*) no módulo `ledger` exigem atomicidade e consistência absolutas.
2. **Concorrência Elevada sem Inconsistência:** Múltiplas requisições simultâneas tentando alterar pagamentos ou contas devem ser tratadas de forma segura e sem travamentos desnecessários (*deadlocks* ou gargalos de throughput).
3. **Evolução de Schema Controlada:** Alterações na estrutura do banco de dados devem ser auditáveis, versionadas e executadas de forma automatizada no pipeline de CI/CD.
4. **Isolamento entre Módulos:** Preservar as fronteiras de dados estabelecidas no *Module Map* (ADR-004), impedindo o acoplamento no nível do banco.

---

## 2. Decisão

Decidimos adotar o **PostgreSQL** como o banco de dados relacional único e primário da **Vortex Engine**, combinado com o **Flyway** para gestão e versionamento de migrations.

A estratégia de persistência é sustentada pelos seguintes pilares técnicos:

1. **Garantias ACID e MVCC:** Utilização nativa do modelo relacional e do *Multi-Version Concurrency Control* (MVCC) do PostgreSQL para suporte a transações financeiras puras.
2. **Flyway para Evolução de Schema:** Migrações DDL versionadas, imutáveis e executadas em tempo de boot/deploy.
3. **Chaves Primárias com UUIDv7:** Adoção universal de UUIDs ordenados por tempo (UUIDv7) para todas as tabelas.
4. **Controle de Concorrência Otimista (Optimistic Locking):** Uso de coluna de versão (`version`) em agregados críticos para prevenir atualizações concorrentes destrutivas sem travar o banco.
5. **Estratégia de Indexação Planejada:** Aplicação rigorosa de índices B-Tree, parciais e compostos focados nas rotas críticas de busca e ordenação.

---

## 3. Detalhamento Técnico e Justificativas

### 3.1. Garantias ACID e MVCC no PostgreSQL

Para um motor financeiro, os quatro pilares ACID são aplicados da seguinte forma no PostgreSQL:

* **Atomicidade:** Transações financeiras que envolvem múltiplos lançamentos contábeis são executadas em bloco (tudo ou nada). Se o débito ocorrer mas o crédito falhar, o PostgreSQL realiza o *rollback* completo do bloco transacional.
* **Consistência:** Garantida por restrições de integridade nativas (`FOREIGN KEY`, `CHECK`, `NOT NULL`, `UNIQUE`). Exemplo: contas financeiras não podem ter saldos negativos quando violam a regra de negócio definida em uma `CHECK constraint`.
* **Isolamento via MVCC (Multi-Version Concurrency Control):** O PostgreSQL gerencia acessos concorrentes através do MVCC. Leituras não bloqueiam escritas e escritas não bloqueiam leituras. Cada transação enxerga um *snapshot* dos dados consistente no tempo. O nível de isolamento padrão adota o `READ COMMITTED`, com escalonamento para `REPEATABLE READ` ou `SERIALIZABLE` em operações contábeis críticas de liquidação.
* **Durabilidade:** Garantida pelo *Write-Ahead Logging* (WAL). Toda transação é gravada em disco no log WAL antes de ser confirmada (*commit*), prevenindo perda de dados em caso de falha de hardware ou queda do processo.

### 3.2. Gerenciamento de Schema com Flyway

A evolução da estrutura de tabelas do banco de dados deve ser tratada como código (*Database-as-Code*):

* **Migrações Versionadas:** Todos os scripts DDL residem no repositório de código em `db/migration/V<Versao>__<descricao>.sql`.
* **Execução Automática e Idempotente:** O Flyway valida o *hash* dos scripts executados na tabela `flyway_schema_history` durante a inicialização da aplicação, impedindo que migrações alteradas retroativamente sejam aplicadas e garantindo paridade total entre ambientes (Desenvolvimento, Staging, Produção).
* **Separação por Escopo do Módulo:** Embora os módulos residam na mesma instância de banco de dados no monólito modular, cada módulo possui suas tabelas isoladas e identificadas explicitamente.

### 3.3. Chaves Primárias: UUID vs. Auto-Incremento (Adoção do UUIDv7)

Descartamos o uso de sequenciais auto-incrementais (`BIGSERIAL` / `BIGINT GENERATED ALWAYS AS IDENTITY`) em favor de identificadores únicos universais (**UUID**):

* **Segurança e Não-Enumerabilidade:** IDs auto-incrementais expõem dados sensíveis de negócio na API (ex: quantidade total de pagamentos processados via `/v1/payments/1042`). UUIDs eliminam esse vetor de raspagem e enumeração.
* **Geração Descentralizada:** Permite que a camada de aplicação/domínio gere o identificador único da entidade antes mesmo de persistir no banco de dados.
* **Por que UUIDv7 em vez de UUIDv4?**
  * **Problema do UUIDv4:** Sendo puramente aleatório, o UUIDv4 causa fragmentação severa de índices B-Tree no PostgreSQL. Inserções em tabelas grandes exigem reordenação de páginas de disco e aumentam significativamente o *bloat* e o consumo de I/O.
  * **Solução com UUIDv7:** O UUIDv7 combina um *timestamp* de milissegundos na primeira metade do valor com dados aleatórios na segunda metade. Isso torna o UUIDv7 **sequencial no tempo** (*time-ordered*). As inserções no índice B-Tree ocorrem sempre no final da árvore, preservando a localidade de referência (*locality of reference*), reduzindo a fragmentação de índices e otimizando buscas por faixa temporal.

### 3.4. Controle de Concorrência Otimista (Optimistic Locking)

Para proteger o estado dos agregados contra acessos concorrentes (ex: duas requisições tentando aprovar o mesmo pagamento simultaneamente), adotamos a estratégia de **Optimistic Locking**:

* **Implementação:** Toda tabela representativa de um Agregado de Domínio (`payments`, `accounts`) possui uma coluna `version BIGINT NOT NULL DEFAULT 0`.
* **Funcionamento:** Na alteração de um registro, a camada de persistência executa implicitamente a instrução:
  ```sql
  UPDATE payments 
  SET status = 'APPROVED', version = version + 1 
  WHERE id = '018f9d2a-7c30-7123-891a-bc0123456789' AND version = 2;
