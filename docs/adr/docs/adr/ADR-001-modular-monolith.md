# ADR-002 — Java 21

## Status

**Aceito**

---

## Contexto

A **Vortex** é uma engine de pagamentos e liquidação financeira projetada para executar operações de baixa latência, alta concorrência e tolerância a falhas. Para suportar esse ecossistema, necessitamos de um ambiente de execução (JDK) que ofereça forte suporte à modelagem conceitual do Domain-Driven Design (DDD), alta performance operacional de I/O e facilidade de manutenção de código.

A escolha da versão da plataforma Java influencia a expressividade das regras de negócio, o modelo de concorrência adotado e a compatibilidade com o ecossistema moderno (como Spring Boot 3.x, Hibernate 6.x e Testcontainers).

---

## Problema

A construção de um sistema financeiro moderno impõe três desafios principais à linguagem e ao runtime:

1. **Segurança no Modelo de Domínio:** Necessidade de garantir imutabilidade em Objetos de Valor (*Value Objects*) e exhaustiveness (verificação exaustiva pelo compilador) em transições de estado de pagamentos, evitando estados inválidos em tempo de execução.
2. **Concorrência de Alta Vazão sem Complexidade Reativa:** Integrações com adquirentes, bancos de dados e sistemas de mensageria geram muitas operações bloqueantes de I/O. Modelos reativos tradicionais (ex: Project Reactor / WebFlux) resolvem o throughput, mas aumentam dramaticamente a complexidade do código, a curva de aprendizado, a dificuldade de depuração e a perda de contexto de chamadas (*stack traces* difíceis e perda de `ThreadLocal`).
3. **Estabilidade de Longo Prazo:** Adoção de uma versão com suporte corporativo contínuo (LTS — *Long Term Support*), evitando atualizações constantes de ciclo curto que tragam instabilidade ao ambiente de produção.

---

## Decisão

Adotaremos o **Java 21 (LTS)** como a versão padrão da linguagem e plataforma de execução da **Vortex**.

Abaixo detalhamos como os recursos da linguagem e da plataforma foram associados às necessidades do projeto:

### 1. Recursos da Linguagem aplicados ao Domínio

* **Records (Value Objects e DTOs):**
  * *Aplicação:* Mapeamento de objetos imutáveis como `Money`, `Currency`, `AccountId`, `PaymentId` e DTOs de API/Eventos.
  * *Benefício:* Garante imutabilidade por padrão, elimina código *boilerplate* (`equals`, `hashCode`, `toString`, `getters`) e expressa claramente a semântica do domínio.
* **Sealed Classes e Interfaces:**
  * *Aplicação:* Restrição de hierarquias de domínio fechadas, como `PaymentStatus` (`Pending`, `Authorized`, `Settled`, `Failed`) ou tipos de `DomainEvent`.
  * *Benefício:* O compilador passa a conhecer todas as implementações possíveis de uma interface, impedindo extensões indevidas fora do módulo.
* **Pattern Matching para `switch` e Record Patterns:**
  * *Aplicação:* Processamento e despacho de eventos de domínio e maquina de estados de pagamentos.
  * *Benefício:* Permite desestruturar *Records* diretamente no `switch` com verificação exaustiva do compilador (dispensando o uso de blocos `default` genéricos e evitando falhas em tempo de execução ao adicionar novos estados).

### 2. Recursos do Runtime (JVM) aplicados à Concorrência

* **Virtual Threads (Project Loom):**
  * *Aplicação:* Execução de requisições síncronas HTTP, chamadas de repositórios JPA/JDBC e clientes de integração.
  * *Benefício:* Permite adotar o modelo intuitivo *thread-per-request* (código imperativo síncrono tradicional) enquanto atinge a capacidade de escala de I/O de arquiteturas reativas. Não exige reescrever a base de código em estilo reativo.

---

## Alternativas Consideradas

### Alternativa 1: Java 17 (LTS)

* **Pontos Fortes:** Versão LTS madura, amplamente adotada pela indústria e com total compatibilidade de bibliotecas terceiras.
* **Por que foi descartada:** Embora suporte *Records* e *Sealed Classes*, o Java 17 não possui **Virtual Threads** nativos de produção nem as evoluções finais de *Pattern Matching* para *Records*. Para atingir alta vazão de I/O no Java 17, seríamos forçados a usar frameworks reativos ou arcar com o custo de memória de *Platform Threads* tradicionais do sistema operacional.

### Alternativa 2: Java 22 ou Versões Não-LTS posteriores

* **Pontos Fortes:** Acesso imediato a recursos mais recentes em pré-visualização (*Preview Features*).
* **Por que foi descartada:** Versões não-LTS possuem ciclo de vida curto (6 meses de suporte) e geram instabilidade e custo de manutenção contínuo para sistemas financeiros, não sendo recomendadas para ambientes de produção críticos.

---

## Diferenciação: Linguagem vs. Ecossistema

Para clareza da decisão, distinguimos os ganhos diretos do Java 21 em duas frentes:

| Âmbito | Benefícios Diretos no Projetos |
| :--- | :--- |
| **Linguagem (Java 21)** | • Tipagem forte, imutabilidade com *Records* e sintaxe mais limpa.<br>• *Pattern Matching* e *Sealed Classes* protegendo invariants do domínio.<br>• Redução drástica de linhas de código utilitário (*boilerplate*). |
| **Ecossistema & Runtime** | • **Virtual Threads** otimizando o uso de CPU e memória em operações de I/O.<br>• Compatibilidade nativa com Spring Boot 3.2+ / 3.x e Hibernate 6.x.<br>• Melhorias contínuas no Garbage Collector (ZGC) reduzindo *pausas de Stop-The-World*. |

---

## Consequências

### Consequências Positivas
* **Código de Domínio Expressivo e Seguro:** Domínio financeiro à prova de estados inconsistentes, validado durante a compilação.
* **Simplicidade de Debugging e Observabilidade:** Código imperativo fácil de depurar e rastrear stack traces ponta a ponta, permitindo a propagação simples de `Correlation-ID` via contexto de thread.
* **Desempenho com Baixa Pegada de Memória:** Milhares de *Virtual Threads* leves alocadas em memória sem o overhead de threads de SO.

### Consequências Negativas
* **Exigência de Tooling Atualizado:** Obriga o uso de JDK 21+ no ambiente de desenvolvimento do desenvolvedor, nas imagens de container Docker e nos agentes de execução de CI/CD.
* **Atenção a Bibliotecas Legadas:** Bibliotecas antigas que utilizem `synchronized` em blocos de I/O longo podem causar *pinning* de Virtual Threads (embora a maioria dos drivers modernos, como Postgres JDBC, já estejam adaptados).

---

## Trade-offs

| O que ganhamos | O que abrimos mão |
| :--- | :--- |
| **Alta vazão de I/O com estilo de código síncrono/imperativo** | Compatibilidade com sistemas legados ou infraestruturas presas a JDKs antigos (< 21) |
| **Segurança de compilação estrita em máquinas de estado financeiras** | Flexibilidade de criação de hierarquias abertas ou dinâmicas |
| **Aproveitamento máximo do suporte de longo prazo da Oracle/Comunidade** | Adoção de recursos *bleeding-edge* experimentais de versões não-LTS |

---

## Revisitando a Decisão no Futuro

Esta decisão poderá ser reavaliada nos seguintes cenários:

1. **Lançamento do Próximo Java LTS (ex: Java 25):** Para avaliar novos ganhos de performance ou funcionalidades de linguagem consolidadas.
2. **Incompatibilidade Crítica de Infraestrutura:** Se alguma dependência core de produção indispensável do ecossistema demonstrar incompatibilidade severa e não corrigível com a JVM 21.

---

## Relação com os Princípios Arquiteturais e Roadmap

* **Simplicidade Pragmática:** Adoção do modelo imperativo de *Virtual Threads* evita a complexidade acidental de código reativo sem perder escalabilidade.
* **Isolamento e Segurança do Domínio:** *Records* e *Sealed Classes* garantem que as regras de negócio mapeadas na **Fase 1 do Roadmap Técnico** permaneçam puras, imutáveis e isoladas.
* **Alinhamento com o Roadmap:** Prepara a fundação para os testes de integração e concorrência da **Fase 3**, assegurando que os mecanismos de concorrência funcionem sobre uma JVM moderna.
