# Portfolio Map

---

## 1. Objetivo

Apresentar de forma clara e estruturada as **competências técnicas**, **padrões de arquitetura** e **soluções de engenharia** demonstrados neste projeto. 

Este documento serve como ponte para recrutadores, lideranças técnicas e engenheiros avaliarem o impacto e a maturidade das decisões aplicadas no desenvolvimento da engine.

---

## 2. Competências Demonstradas

* **Domain-Driven Design (DDD):** Isolamento do modelo de domínio e abstração das regras do negócio de pagamentos e ledger.
* **Arquitetura Hexagonal (Ports & Adapters):** Desacoplamento entre o núcleo de regras financeiras e os detalhes de infraestrutura (banco de dados, mensageria e APIs).
* **Consistência Transacional e Concorrência:** Tratamento de *Race Conditions* e prevenção do gasto duplo com bloqueios transacionais e concorrência no banco de dados.
* **Garantia de Idempotência:** Intercepção e controle de chamadas repetidas para prevenção de débitos duplicados.
* **Mensageria e Sincronização Assíncrona:** Implementação de mensageria com o padrão *Transactional Outbox* para eliminação do problema de *Dual-Write*.
* **Resiliência em Sistemas Distribuídos:** Aplicação de *Circuit Breakers*, *Timeouts* e *Fallbacks* para isolar falhas de serviços externos.
* **Observabilidade Nativa:** Uso de *Correlation IDs*, rastreamento distribuído e estruturação de logs e métricas desde o início do projeto.
* **Tomada de Decisão por ADRs:** Documentação de trade-offs técnicos usando *Architecture Decision Records*.

---

## 3. Problemas de Mercado Abordados

O projeto ataca diretamente dores críticas do setor de tecnologia financeira:

* **Cobrança Duplicada por Retries:** Proteção contra reenvios de requisições causados por instabilidade na rede.
* **Gasto Duplo em Execuções Simultâneas:** Proteção da integridade do saldo da conta sob alta demanda simultânea.
* **Perda de Notificações Financeiras:** Sincronização garantida entre o banco de dados e a fila de mensagens.
* **Cascada de Falhas por Terceiros:** Proteção da aplicação principal contra instabilidades em adquirentes e antifraude.

---

## 4. Tecnologias e Conceitos

### Conceitos e Padrões de Projeto
* Clean Architecture / Hexagonal Architecture
* Transactional Outbox Pattern
* Idempotency Pattern
* Resilience Patterns (Circuit Breaker, Rate Limiter, Retry)
* Concurrency Control (Pessimistic / Optimistic Locking)

### Tecnologias de Implementação
* **Linguagem & Framework:** Java 17 / Spring Boot 3
* **Persistência & Cache:** PostgreSQL / Redis
* **Mensageria:** Apache Kafka
* **Containerização & Testes:** Docker / Testcontainers
* **Observabilidade:** Prometheus / Grafana / OpenTelemetry

---

## 5. Diferenciais do Projeto

* **Orientado a Risco Financeiro:** Construído com base na mitigação de perdas e falhas operacionais.
* **Domínio Puro e Testável:** Regras de negócio 100% isoladas de bibliotecas e frameworks externos.
* **Documentação Arquitetural Completa:** Alinhamento de visão, escopo, princípios e tomadas de decisão estruturadas.
* **Testes de Resiliência:** Validação de cenários extremos de queda de banco de dados, falha de mensageria e lentidão de APIs.

---

## 6. Relação com Vagas de Mercado

Este projeto evidencia os conhecimentos mais exigidos em vagas de **Desenvolvimento Backend (Pleno/Sênior)** e **Engenharia de Software em Fintechs**:

| Requisito Comum de Vaga | Como o Projeto Demonstra a Competência |
| :--- | :--- |
| **"Experiência com microsserviços e sistemas distribuídos"** | Aplicação de desacoplamento, mensageria e comunicação assíncrona orientada a eventos. |
| **"Domínio de resiliência e alta disponibilidade"** | Proteção contra falhas em lote via Circuit Breakers e Dead Letter Queues (DLQ). |
| **"Garantia de consistência e integridade de dados"** | Implementação de idempotência e Transactional Outbox Pattern. |
| **"Conhecimento em DDD e Arquitetura Limpa"** | Isolamento total do núcleo de domínio em relação à infraestrutura. |
| **"Cultura de Observabilidade e Diagnóstico"** | Rastreabilidade total com Correlation ID e métricas expostas. |
