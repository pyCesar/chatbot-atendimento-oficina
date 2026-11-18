# Chatbot de atendimento para oficina mecânica

Chatbot com IA que atende clientes de uma oficina fictícia, a **AutoCerto Oficina**. Ele responde dúvidas sobre serviços e preços iniciais e, quando o cliente quer um orçamento, coleta nome e telefone e registra tudo numa planilha do Google Sheets, usando uma automação no Make.

> Projeto de estudo. A oficina, os serviços, os preços e os dados dos testes são fictícios.

<img width="694" height="642" alt="01_conversa_preco_confirmacao" src="https://github.com/user-attachments/assets/a3b5dd70-75d4-4b8d-9f9d-81f07773ca2e" />


## Como funciona

```
Cliente  ->  Botpress (agente + playbook)  ->  Ferramenta HTTP "Registrar lead"
         ->  Webhook do Make  ->  Google Sheets (aba leads\\\_chatbot)
```

1. O cliente conversa com o bot (testei pelo preview e pela simulação do Botpress).
2. O bot responde usando uma base de conhecimento com os serviços, preços e perguntas frequentes.
3. Se o cliente pede orçamento, o bot segue um playbook com passos fixos: descobrir o serviço, informar o preço inicial, **pedir confirmação**, pedir o nome, pedir o telefone e só então registrar.
4. A ferramenta HTTP envia `nome`, `telefone` e `servico` para um webhook do Make.
5. O Make grava uma linha na planilha, com data e hora no fuso de São Paulo.

<img width="910" height="552" alt="04_planilha_lead_registrado" src="https://github.com/user-attachments/assets/76376194-839a-4a66-ad49-e2a037e97e23" />


## O que o bot faz e o que não faz

**Faz**

* Responde sobre serviços, preços iniciais, tipos de veículo atendidos e formas de pagamento.
* Informa o preço do serviço que o cliente pediu e confirma se é esse mesmo.
* Valida o formato do telefone antes de registrar.
* Recusa assuntos que não são da oficina.

**Não faz**

* Não agenda nada. Ele avisa isso ao cliente.
* Não inventa preço, prazo ou disponibilidade. Se o serviço não está na base, ele diz que a oficina não oferece.
* Não negocia desconto.

O escopo completo está em [escopo.md](escopo.md) e o conteúdo da base em [base\_conhecimento.md](base_conhecimento.md).

## Dados registrados

|Coluna|Exemplo|
|-|-|
|data\_hora|17/11/2026 21:46:30|
|nome|Jeferso|
|telefone|21999990000|
|servico|revisão básica|

## Como testei

Montei uma tabela de testes ([testes.csv](testes.csv)) e fui anotando cada falha, a correção e o resultado depois. Resumo:

|Versão|Teste|Problema|Correção|
|-|-|-|-|
|v1|"Qual a capital da França?"|O bot respondia assunto fora da oficina|Regra de escopo e recusa de pedidos para ignorar regras|
|v2|"Tem desconto no Pix?"|Resposta ambígua|Entrada no FAQ: não há desconto fixo|
|v3|Orçamento de freios|Pedia nome e telefone juntos e repetia a base inteira|Playbook reescrito, um dado por vez|
|v4|Fim da conversa|Alternava "obrigado" e "obrigada"|Saudação e linguagem neutras|
|v5|Telefones `123` e `21 99 99`|Aceitava qualquer número|Regra de formato de telefone brasileiro|
|v6|Revisão básica do início ao fim|Pulava a confirmação do serviço e a mensagem final parecia agendamento|Novo fluxo com preço, confirmação e aviso de que não é agendamento|
|v7|"Quero funilaria"|Dizia que não tinha o serviço, mas pedia confirmação e registrava o lead|O fluxo para quando o serviço não está na base|

<img width="514" height="666" alt="02_telefone_invalido" src="https://github.com/user-attachments/assets/f2399300-42cc-493a-83e2-444ddc4eafc2" />

<img width="594" height="676" alt="03_funilaria_recusa" src="https://github.com/user-attachments/assets/fa51a42a-fb0f-496e-8f1c-7c20e65a7809" />


## O que a IA gerou e o que eu corrigi

A primeira versão das instruções foi escrita com ajuda de IA. Só lendo o texto ela parecia boa, e foi nos testes que apareceram os erros da tabela acima. O que aprendi com isso:

* Instrução geral demais não segura o bot. O playbook só ficou estável quando virou passos numerados com regras explícitas, como "peça SOMENTE o nome".
* Cada regra nova pode quebrar uma antiga. A regra "não repetir preços" (v3) atrapalhou o fluxo de confirmação (v6), então tive que reescrevê-la.
* Teste com casos fora do caminho feliz (serviço que não existe, telefone errado, cliente que muda de ideia) acha mais problema do que o teste normal.

## Limitações

* A validação do telefone é uma instrução para a IA, não código. Ela funcionou nos meus testes, mas um modelo de linguagem pode errar. Dá para reforçar com um filtro no Make.
* O bot não consulta agenda, estoque nem preço real. Tudo vem da base de conhecimento.
* Às vezes repete "Anotado!" em mensagens seguidas.
* Em pedidos de agendamento, a resposta pode soar contraditória: diz que não agenda por aqui e também recomenda agendar.
* Os testes cobrem os casos que pensei, não todos os possíveis.
* O registro de leads só funciona com o cenário do Make ligado.

## Ferramentas

* **Botpress**: agente de IA, base de conhecimento, playbook e ferramenta HTTP
* **Make**: webhook e módulo do Google Sheets
* **Google Sheets**: registro dos leads
* **IA generativa (Claude)**: apoio na escrita das instruções e na revisão dos testes

## Estrutura do projeto

```
chatbot-atendimento-oficina/
├── README.md
├── escopo.md
├── base\\\_conhecimento.md
├── prompts/
├── testes.csv
├── automacao/
└── prints/
```

## Autor

Cesar Barbosa  
[LinkedIn](https://www.linkedin.com/in/cesar-barbosa-013622303/) | [GitHub](https://github.com/pyCesar)

