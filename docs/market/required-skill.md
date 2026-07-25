# Skills to Features

Este documento conecta cada competência técnica mapeada no projeto **Vortex** à sua aplicação prática, identificando o módulo correspondente e o problema real do mercado financeiro que ela resolve.

---

## 1. Arquitetura

| Competência | Funcionalidade Relacionada | Módulo / Contexto | Problema Real Resolvido |
| :--- | :--- | :--- | :--- |
| **Modelagem de Domínio (DDD)** | Isolamento de Entidades, Objetos de Valor e Bounded Contexts de Pagamento e Ledger | `domain` | Regras financeiras dispersas e código espaguete com vazamento de lógica de negócio. |
| **Arquitetura Hexagonal** | Separação estrita entre Casos de Uso (Ports) e Adaptadores de Infraestrutura (Adapters) | `application` / `infrastructure` | Acoplamento rígido de regras de negócio a frameworks, bancos de dados e bibliotecas. |
| **Monólito Modular** | Módulos internos bem delimitados e desacoplados (`payment`, `ledger`, `messaging`) | Estrutura de pacotes da aplicação | Complexidade operacional desnecessária de microsserviços mantendo alto isolamento de código. |

---

## 2. Backend

| Competência | Funcionalidade Relacionada | Módulo / Contexto | Problema Real Resolvido |
| :--- | :--- | :--- | :--- |
| **Gerenciamento de Transações** | Controle de transações ACID com rollback automático na reserva e débito de saldo | `ledger` / `payment` | Transações parciais ou dados corrompidos após falhas no meio do processamento. |
| **Controle de Concorrência** | Bloqueio pessimista/otimista no saldo da conta em requisições paralelas | `ledger` | Gasto duplo (*Race Condition*) quando múltiplos débitos ocorrem no mesmo milissegundo. |
| **Design Orientado a Interfaces** | Abstração de contratos para gateways, repositórios e serviços externos | `domain / ports` | Dificuldade para criar dublês de testes e dependência direta de fornecedores específicos. |
| **Versionamento de APIs** | Prefixos e rotas explícitas de versão nos endpoints de entrada (ex: `/v1/payments`) | `entrypoint / api` | Quebra de contratos em produção com clientes e parceiros ao evoluir a API. |

---

## 3. Sistemas Distribuídos

| Competência | Funcionalidade Relacionada | Módulo / Contexto | Problema Real Resolvido |
| :--- | :--- | :--- | :--- |
| **Garantia de Idempotência** | Interceptador de requisições com validação de cabeçalho `Idempotency-Key` | `payment / idempotency` | Cobrança em duplicidade causada por retries de rede ou cliques repetidos do usuário. |
| **Consistência Eventual** | Processamento e notificação assíncrona desacoplados do fluxo crítico de resposta | `messaging / outbox` | Retenção e bloqueio síncrono do usuário aguardando etapas secundárias. |
| **Mensageria Orientada a Eventos** | Publicação garantida de eventos (`PaymentApproved`, `PaymentFailed`) via Transactional Outbox | `messaging` | Perda de notificações e inconsistência entre o banco de dados e a fila (*Dual-Write*). |
| **Padrões de Resiliência** | Proteção com Circuit Breaker, Timeout e Rate Limiter nas chamadas de Antifraude e Adquirente | `infrastructure / client` | Queda do sistema em cascata devido a instabilidade ou lentidão em APIs parceiras. |
| **Observabilidade e Rastreabilidade** | Interceptador de `Correlation-ID` propagado em logs, traces e chamadas assíncronas | `observability` | Dificuldade e demora no diagnóstico de erros e rastreio de transações em produção. |

---

## 4. Qualidade

| Competência | Funcionalidade Relacionada | Módulo / Contexto | Problema Real Resolvido |
| :--- | :--- | :--- | :--- |
| **Testes Unitários** | Testes de isolamento das regras de saldo, cálculos e transições de estado do domínio | `domain / test` | Regras de negócio quebradas silenciosamente após refatorações no código. |
| **Testes de Integração** | Validação de fluxos reais usando containers de Banco de Dados e Mensageria via Testcontainers | `infrastructure / test` | Incompatibilidade entre o comportamento do código e o comportamento real do banco/fila. |
| **Testes de Resiliência** | Simulação de indisponibilidade e latência na rede e serviços externos | `test / resilience` | Falhas inesperadas em cenários de estresse e indisponibilidade de infraestrutura. |

---

## 5. DevOps

| Competência | Funcionalidade Relacionada | Módulo / Contexto | Problema Real Resolvido |
| :--- | :--- | :--- | :--- |
| **Automação de CI/CD** | Pipeline de build, verificação de linter, segurança e execução automatizada de suíte de testes | `.github/workflows` | Envio acidental de código sem testes ou quebrado para os ambientes de integração/produção. |
| **Containerização** | Padronização dos ambientes de desenvolvimento e execução com Docker | `Dockerfile` | O problema clássico do *"funciona na minha máquina, mas não em produção"*. |
| **Infraestrutura como Código (IaC)** | Declaração e subida automatizada dos serviços de suporte (Kafka, Postgres, Redis) via Docker Compose | `docker-compose.yml` | Configurações manuais, lentas e propensas a falhas humanas no ambiente de infraestrutura. |

---

## 6. Engenharia

| Competência | Funcionalidade Relacionada | Módulo / Contexto | Problema Real Resolvido |
| :--- | :--- | :--- | :--- |
| **Documentação Arquitetural** | Documentos vivos de Visão, Escopo, Princípios e Mapeamento de Portfólio | `docs/` | Desalinhamento técnico da equipe e falta de contexto do propósito do software. |
| **Documentação por ADRs** | Registros de Decisão de Arquitetura com justificativas, alternativas e trade-offs | `docs/adr/` | Perda do histórico de motivos e razões que levaram a determinada escolha técnica. |
| **Boas Práticas de Código** | Escrita baseada em legibilidade, baixíssima complexidade ciclomática e responsabilidade única | Todo o projeto | Código difícil de ler, rígido para alterar e com alto custo cognitivo de manutenção. |
| **Análise de Trade-offs** | Decisão deliberada de priorizar consistência sobre disponibilidade no Ledger | `architecture-principles.md` | Escolhas tecnológicas orientadas a moda ou impulso, sem avaliação do risco de negócio. |
