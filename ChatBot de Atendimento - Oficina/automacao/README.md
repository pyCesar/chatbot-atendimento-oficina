# Automação: registro de leads

O bot envia os dados do cliente para o Make, que grava uma linha no Google Sheets.

```
Botpress (ferramenta HTTP "Registrar lead") -> Webhook do Make -> Google Sheets
```

## 1\. Ferramenta HTTP no Botpress

Nome: **Registrar lead**
Método: POST
URL: `SEU\_WEBHOOK\_AQUI` (a URL do webhook do Make, não publicada por segurança)

Inputs (todos texto e obrigatórios): `nome`, `telefone`, `servico`

Corpo (JSON):

```json
{
  "nome": "{{nome}}",
  "telefone": "{{telefone}}",
  "servico": "{{servico}}"
}
```

A ferramenta só roda se estiver citada no playbook (aparece como uma etiqueta no passo 6).

## 2\. Cenário no Make

Dois módulos:

1. **Webhooks > Custom webhook**: recebe `nome`, `telefone` e `servico`.
2. **Google Sheets > Add a Row**: grava na aba `leads\_chatbot`.

Mapeamento das colunas:

|Coluna|Valor|
|-|-|
|A: data\_hora|`{{formatDate(now; "DD/MM/YYYY HH:mm:ss"; "America/Sao\_Paulo")}}`|
|B: nome|campo `nome` do webhook|
|C: telefone|campo `telefone` do webhook|
|D: servico|campo `servico` do webhook|

A data usa `formatDate` com o fuso de São Paulo, porque o Make entrega a hora em UTC.

## 3\. Pontos de atenção

* O cenário precisa estar **ativo** (chave no canto superior direito). Com ele desligado, os dados ficam na fila do webhook e não chegam na planilha.
* Se o Make mostrar erro de campo "Missing", confira se os nomes dos inputs no Botpress são iguais aos do webhook.
* Formate a coluna do telefone como texto simples, senão o Sheets trata como número.
* Melhoria possível: um filtro entre os dois módulos que só deixa passar telefones no formato brasileiro, como segunda trava além da instrução do playbook.

## Arquivos

* blueprint.json

