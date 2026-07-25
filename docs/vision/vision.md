# Vision

## Contexto
Quando você clica no botão "Pagar" em um aplicativo, um gigante processo complexo começa a trabalhar nos bastidores: o sistema precisa validar suas informações, reservar o saldo na sua conta e avisar outros sistemas que a compra deu certo.

Em um cenário com milhões de transações diárias, falhas de rede, oscilações na nuvem e instabilidades em parceiros externos não são a exceção — são a regra. Se a arquitetura do sistema não for desenhada para lidar com o imprevisível, essas pequenas falhas do mundo digital viram prejuízos financeiros reais e insatisfação para o usuário.

## Problema
Sem os cuidados certos de engenharia, sistemas de pagamento sofrem com quatro dores clássicas:

1. **O "Clique Duplo" (Cobrança Duplicada):** A rede oscila, o app não recebe a confirmação a tempo e o cliente tenta de novo. Sem proteção, o sistema cobra o mesmo valor duas ou mais vezes.
2. **A Corrida pelo Saldo (Race Condition):** Duas transações tentam consumir o mesmo saldo no mesmo milissegundo. Sem controle de acesso simultâneo, a conta pode ficar negativa.
3. **O Esquecimento (Problema de Dual-Write):** O sistema tira o dinheiro da conta, mas a internet cai logo antes de enviar o comprovante ou notificar a contabilidade. O dinheiro sumiu e ninguém sabe o motivo.
4. **O Efeito Dominó (Falhas em Cadeia):** O serviço de antifraude fica lento ou cai. Em vez de isolar o problema, a aplicação inteira trava esperando por ele.

## Objetivo
Criar uma **engine de pagamentos blindada contra perdas financeiras e falhas operacionais**. 

O foco é garantir que:
- Toda cobrança seja processada **exatamente uma vez** (nem mais, nem menos).
- O saldo do cliente permaneça **sempre consistente**, mesmo com acessos simultâneos.
- Instabilidades de parceiros externos **não derrubem** a aplicação.
- Nenhuma transação ou notificação **seja perdida no caminho**.

## Público-alvo
- **Devs e Estudantes:** Um guia prático para entender como aplicar padrões de resiliência e consistência transacional na prática.
- **Lideranças e Recrutadores:** Uma demonstração clara de maturidade técnica e capacidade de resolver problemas sérios do setor financeiro.
- **Arquitetos de Solução:** Um exemplo de como separar bem as regras de negócio dos detalhes técnicos e de infraestrutura.

## Valor entregue
- **Tranquilidade Financeira:** Fim do prejuízo com estornos manuais e chamados no suporte por cobranças indevidas.
- **Sistemas Sempre No Ar:** A aplicação continua rodando mesmo quando serviços parceiros estão instáveis.
- **Transparência Total:** Rastreabilidade completa de cada centavo e de cada passo de uma transação.
- **Código Limpo e Mantenável:** Regras de negócio isoladas de frameworks, facilitando a evolução e os testes do projeto.

## Restrições
- **Foco em Pagamentos:** O projeto é uma engine de processamento e resiliência, não um banco completo (sem emissão de cartões, ERP ou gestão fiscal).
- **Domínio Livre de Frameworks:** As regras de negócio do núcleo não dependem de bibliotecas de terceiros.
- **Segurança de Dados em Primeiro Lugar:** Alterações de saldo exigem garantia total de consistência no banco de dados principal.
- **Processamento Assíncrono:** Notificações e tarefas secundárias não podem atrasar a resposta imediata para o usuário.

## Métricas de sucesso
- **Cobranças Duplicadas:** 0% (mesmo se o usuário ou cliente reenviar a mesma requisição várias vezes).
- **Saldo Negativo Indevido:** 0% de ocorrências em testes de estresse com chamadas simultâneas.
- **Perda de Mensagens:** 0% de divergência entre o que foi debitado no banco e o que foi notificado aos outros sistemas.
- **Contenção de Falhas:** Resposta imediata de isolamento quando um parceiro externo falhar, sem travar o restante da aplicação.
- **Rastreabilidade:** 100% das transações identificadas e auditáveis do início ao fim.
