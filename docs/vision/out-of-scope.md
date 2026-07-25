# Out of Scope (Fora do Escopo)

---

## 1. Funcionalidades Não Contempladas

* **Internet Banking Completo:** Telas de usuário, extratos consolidados, pagamento de boletos e gestão de conta corrente.
* **Emissão e Gestão de Cartões:** Criação de cartões físicos ou virtuais, controle de limite e alteração de senha.
* **Gestão de Clientes (KYC):** Cadastro de usuários, envio de documentos, validação biométrica e checagem de antecedentes.
* **Investimentos e Câmbio:** Compra e venda de ativos, conversão de moedas e acompanhamento de rentabilidade.
* **Gestão Fiscal e ERP:** Emissão de notas fiscais, cálculo de impostos e relatórios contábeis complexos.

---

## 2. Integrações Não Previstas

* **PIX Real (Banco Central):** Sem conexão com a rede do SPI (Sistema de Pagamentos Instantâneos) ou DICT do BACEN.
* **Adquirentes de Produção:** Sem comunicação real com credenciadoras de cartão (Cielo, Rede, Stone) ou bandeiras (Visa, Mastercard).
* **Autenticação Multifator (MFA):** Sem envio de códigos via SMS, WhatsApp ou aplicativos autenticadores.

---

## 3. Justificativas

* **Preservação do Foco Técnico:** O objetivo único é resolver problemas de resiliência, idempotência e consistência transacional no processamento de pagamentos.
* **Redução de Complexidade Regulatória:** Integrações reais com o Banco Central ou adquirentes exigem burocracias e licenças que desviam a atenção do núcleo de engenharia.
* **Proteção do Domínio:** Evitar que regras periféricas (como cadastro de telas ou cálculo de impostos) poluam a arquitetura da engine.

---

## 4. Possíveis Evoluções Futuras

* **Simulação de Chaves PIX:** Criação de um mock interno para simular liquidação em tempo real via PIX.
* **Motor de Antifraude Dinâmico:** Módulo configurável para análise de risco com base em regras de histórico transacional.
* **Ledger Multimoeda:** Suporte a diferentes moedas dentro do livro-razão sem dependência de serviços externos de câmbio.
