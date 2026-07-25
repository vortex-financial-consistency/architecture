# Required Skills

Este documento define o conjunto de competências técnicas que esta engine de pagamentos demonstra. O objetivo é refletir o nível de maturidade exigido para profissionais de Engenharia Backend (Pleno/Sênior) no mercado financeiro.

---

## Competências de Arquitetura

* **Modelagem de Domínio (DDD Estratégico e Tático):** Traduz regras financeiras complexas diretamente no código, mantendo o núcleo do negócio protegido contra mudanças tecnológicas.
* **Arquitetura Hexagonal (Ports & Adapters):** Garante que o domínio financeiro não dependa de frameworks ou bancos de dados, facilitando testes e trocas de componentes.
* **Monólito Modular (Modular Monolith):** Proporciona forte isolamento entre contextos de negócio com baixa complexidade operacional, permitindo evolução estruturada.

---

## Competências de Backend

* **Gerenciamento de Transações (Transaction Management):** Domínio amplo de propriedades ACID, níveis de isolamento, estratégias de commit/rollback e integridade financeira.
* **Controle de Concorrência:** Protege o saldo das contas contra requisições simultâneas no mesmo milissegundo, eliminando o risco de gasto duplo.
* **Design Orientado a Interfaces:** Desacopla contratos de uso de suas implementações reais, facilitando a criação de dublês de teste e simulação de dependências.
* **Versionamento de APIs:** Permite evoluir contratos de integração de pagamentos sem quebrar clientes ou parceiros que utilizam versões anteriores.

---

## Competências de Sistemas Distribuídos

* **Garantia de Idempotência:** Impede que requisições repetidas ou reenviadas por falhas de rede gerem novos débitos na conta do cliente.
* **Consistência Eventual:** Compreensão de quando substituir chamadas síncronas por comunicação assíncrona, mantendo a integridade do negócio ao longo do tempo.
* **Mensageria Orientada a Eventos:** Permite a comunicação assíncrona entre módulos, garantindo notificações confiáveis sem impactar o tempo de resposta síncrono.
* **Padrões de Resiliência:** Protege o sistema contra instabilidades externas usando Circuit Breakers, Timeouts e Fallbacks, evitando quedas em cascata.
* **Observabilidade e Rastreabilidade:** Permite monitorar a saúde do sistema e rastrear transações ponta a ponta com Correlation IDs para diagnósticos rápidos.

---

## Competências de Qualidade

* **Testes Unitários:** Validam regras de negócio isoladas em milissegundos, garantindo que cálculos e estados do domínio estejam sempre corretos.
* **Testes de Integração:** Validam a comunicação real entre a aplicação, banco de dados, mensageria e caches sob condições operacionais reais.
* **Testes de Resiliência:** Simulam cenários críticos de falha (queda de banco de dados ou lentidão em APIs) para validar a capacidade de recuperação do sistema.

---

## Competências de DevOps

* **Automação de CI/CD:** Garante que o código seja validado, testado e empacotado automaticamente a cada alteração, reduzindo falhas humanas no deploy.
* **Containerização e Ambientes Reprodutíveis:** Garante que o ambiente de execução seja idêntico em desenvolvimento e produção, facilitando testes e execução.
* **Infraestrutura como Código (IaC):** Gestão e provisionamento declarativo do ambiente de execução e suas dependências por código reproduzível.

---

## Competências de Engenharia

* **Documentação Arquitetural:** Mapeamento contínuo de visão de produto, escopo, princípios, diagramas estruturais e descobertas do domínio.
* **Documentação por ADRs (Architecture Decision Records):** Registra o histórico e as justificativas de cada decisão técnica relevante, facilitando a evolução do projeto.
* **Boas Práticas de Código (Clean Code):** Mantém a base de código legível, modular e simples de manter, reduzindo o custo cognitivo para a equipe.
* **Análise de Trade-offs:** Pondera prós e contras de cada escolha arquitetural (como priorizar consistência em vez de disponibilidade) com foco no risco financeiro.
