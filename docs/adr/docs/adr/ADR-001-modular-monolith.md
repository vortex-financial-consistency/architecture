# ADR-001 — Modular Monolith

## Status

**Aceito**

---

## Contexto

A **Vortex** é uma engine de pagamentos e liquidação financeira projetada para garantir altíssimo nível de consistência, prevenção contra gasto duplo e resiliência em transações. No estágio atual da aplicação, as fronteiras do domínio e as interações entre os contextos delimitados (como *Payment*, *Ledger* e *Messaging*) estão sendo consolidadas e validadas.

A escolha da topologia arquitetural inicial determina diretamente a velocidade de desenvolvimento, a complexidade operacional, os custos de infraestrutura e a facilidade de refatoração do código.

---

## Problema

Precisamos de uma arquitetura que ofereça:

1. **Forte Isolamento de Domínio:** Garantia de desacoplamento entre diferentes contextos de negócio para evitar que o código se torne um monólito acoplado (*Big Ball of Mud*).
2. **Baixa Complexidade Operacional:** Minimização de falhas distribuídas (latência de rede, serialização, particionamento) durante o amadurecimento das regras de negócio.
3. **Garantia de Consistência Transacional:** Capacidade de executar operações atômicas no banco de dados local com facilidade quando a regra de negócio exigir.
4. **Facilidade de Refatoração:** Flexibilidade para redefinir limites de módulos à medida que o conhecimento sobre o domínio financeiro evolui.

O desafio consiste em escolher uma arquitetura que ofereça a disciplina e a modularidade dos microsserviços sem arcar imediatamente com sua complexidade de infraestrutura e redes.

---

## Decisão

Adotaremos a arquitetura de **Monólito Modular (*Modular Monolith*)**.

A aplicação será construída e empacotada como uma **única unidade de deploy** (processo único), porém internamente estruturada em **módulos independentes e estritamente delimitados** (`payment`, `ledger`, `messaging`, etc.).

### Regras de Convivência entre Módulos:
* **Comunicação por Contratos Explícitos:** Um módulo não pode acessar diretamente tabelas ou entidades de outro módulo.
* **Isolamento de Persistência:** Cada módulo possui seu próprio conjunto de tabelas/esquemas organizados logicamente.
* **Invocação via Interfaces ou Eventos:** A comunicação síncrona entre módulos é feita exclusivamente por interfaces de serviços (In-Memory Ports), enquanto a assíncrona utiliza eventos de domínio.

---

## Alternativas Consideradas

### Alternativa 1: Arquitetura de Microsserviços Distribuídos

Divisão imediata da aplicação em múltiplos serviços independentes (ex: *Payment Service*, *Ledger Service*, *Outbox Service*), cada um com seu próprio ciclo de deploy e banco de dados isolado.

* **Pontos Fortes:**
  * Escalabilidade independente por módulo.
  * Autonomia total de deploy para equipes distintas.
  * Isolamento de falhas em nível de processo/infraestrutura.
* **Por que foi descartada:**
  * **Overhead Operacional Prematuro:** Exige infraestrutura complexa (serviço de descoberta, mTLS, tracing distribuído, observabilidade em malha).
  * **Complexidade Transacional:** Exige o uso do padrão Saga e consistência eventual para cenários onde a transação atômica local ainda é o requisito mais seguro.
  * **Custo de Refatoração:** Alterar fronteiras de domínio incorretas entre microsserviços exige mudanças em múltiplos repositórios, contratos de rede e pipelines de CI/CD.

### Alternativa 2: Monólito Tradicional Camado (Layered Monolith)

Organização do sistema em camadas globais (ex: `controllers`, `services`, `repositories`) compartilhadas por toda a aplicação.

* **Pontos Fortes:**
  * Extrema simplicidade inicial de implementação.
  * Facilidade de navegação em projetos pequenos.
* **Por que foi descartada:**
  * **Risco de Acoplamento Descontrolado:** Sem barreiras rígidas de código, regras de *Ledger* e *Payment* terminam misturadas nos mesmos serviços e tabelas.
  * **Dificuldade de Testes:** Impossibilidade de testar módulos em isolamento total.

---

## Consequências

### Consequências Positivas
* **Simplicidade de Deploy e Execução:** Apenas um artefato para construir, testar, containerizar e implantar.
* **Latência Mínima:** A comunicação entre sub-domínios ocorre em memória (chamadas de método), sem overhead de rede ou serialização HTTP/gRPC.
* **Evolução Segura do Domínio:** Permite ajustar limites entre sub-domínios via refatoração simples no código antes de fixar contratos de rede.
* **Transições Transacionais Flexíveis:** Possibilita manter transações ACID locais nos fluxos onde a consistência imediata é mandatória para a segurança financeira.

### Consequências Negativas
* **Escalabilidade Unificada:** Não é possível dimensionar recursos (CPU/Memória) de forma isolada para um único módulo de alta demanda; toda a aplicação deve ser escalada junto.
* **Acoplamento de Runtime:** Um erro crítico em tempo de execução (ex: *Out Of Memory*) em um módulo afeta a disponibilidade do processo inteiro.
* **Risco de Permeabilidade:** Sem o uso de ferramentas de enforcement de arquitetura (como ArchUnit ou visibilidade de pacotes do Java), desenvolvedores podem quebrar a modularidade acidentalmente.

---

## Trade-offs

| O que ganhamos | O que abrimos mão |
| :--- | :--- |
| **Simplicidade operacional e baixo custo de infraestrutura** | Escalabilidade e deploy independentes por funcionalidade |
| **Refatoração rápida de regras de domínio e chamadas em memória** | Isolamento de falhas em tempo de execução por processo |
| **Facilidade de testes de integração ponta a ponta** | Diversidade tecnológica livre por módulo |

---

## Revisitando a Decisão no Futuro

Esta decisão arquitetural não é definitiva e deve ser reavaliada caso o projeto atinja os seguintes gatilhos de mudança:

1. **Divergência de Escalabilidade:** Quando um módulo específico (ex: consulta de extrato/ledger) exigir volume de escala 10x maior que os demais, justificando a separação física.
2. **Crescimento Organizacional:** Quando o projeto for mantido por múltiplas equipes autônomas que necessitem de ciclos de release e pipelines de deploy totalmente desvinculados.
3. **Maturidade de Domínio:** Quando as fronteiras dos contextos delimitados estiverem 100% consolidadas e comprovadas em produção.

*Nota: Graças ao isolamento estrito imposto pelo Monólito Modular, a eventual extração de um módulo para um microsserviço no futuro será uma tarefa cirúrgica, e não uma reescrita total.*

---

## Relação com os Princípios Arquiteturais

* **Priorização da Consistência sobre Disponibilidade:** O Monólito Modular viabiliza o controle estrito de transações ACID no banco relacional local antes do disparo de eventos assíncronos.
* **Isolamento de Domínio:** Preserva a independência conceitual dos contextos delimitados do DDD dentro do mesmo repositório.
* **Simplicidade Pragmática:** Evita a sobrecarga de sistemas distribuídos enquanto a equipe e o produto ainda estão validando os requisitos fundamentais do negócio.
