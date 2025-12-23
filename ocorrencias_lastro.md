# API Nido Adm

Manual de Integração

## Itens necessários

- \_\_ENDPOINT_CLIENTE__
  - Ex: https://app.nidoadm.com.br/sistemadocliente/index.php/Lastro/admin/
- \_\_TOKEN_CLIENTE__
  - Token JWT, Ex: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9uaWRvLmNvbS5iciIsImF1ZCI6Imh0dHA6XC9cL25pZG9pbW92ZWwuY29tLmJyIiwiaWF0IjoxNTQ2MzA4MDAwLCJuYmYiOjE1NDYzMDgwMDAsImRhdGEiOnsiY2xpZW50ZV9pZCI6IjMxIiwiY29kYWdlbmNpYSI6Ik5JIiwid2ViX2NvbmZpZ19pZCI6IjgyIn19.VARfNYjb9yIY7pB01HcUJipMC1HEnaG028307Elfz1s

## Métodos

- `GET {__ENDPOINT_CLIENTE__}/incidentTypes`, Exibe os tipos de ocorrência.
- `GET {__ENDPOINT_CLIENTE__}/users?document=CPF_CNPJ`, Busca o participante vinculado ao CPF ou CNPJ informado.
- `POST {__ENDPOINT_CLIENTE__}/incidents`, Cria com os dados do formulário a nova ocorrência.

Esta API utiliza **JWT Token** para autenticação e disponibiliza endpoints relacionados a **ocorrências** e **participantes**.

## Segurança

- **Token JWT**: obrigatório como método de autenticação
- **User-Agent**: obrigatório utilizar o agente **Nido** no cabeçalho da requisição

## Autenticação

Todos os endpoints requerem o envio de um **Bearer Token JWT** no cabeçalho:

Exemplo de cabeçalho:

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
User-Agent: Nido/1.0.0
```

---

## Métodos Disponíveis

**Endpoint:**

### 🔹 `GET {__ENDPOINT_CLIENTE__}/incidentTypes`

**Descrição:**
Retorna a lista de tipos de ocorrência disponíveis no sistema.

#### Exemplo de retorno:

```json
[
  {
    "id": "1",
    "name": "Problema técnico"
  },
  {
    "id": "2",
    "name": "Dúvida comercial"
  }
]
```

#### Exemplo CURL: | Retornar todos os tipos de ocorrências

**Exemplo de requisição:**

```shell
curl -X GET \
  '__ENDPOINT_CLIENTE__/incidentTypes' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0'
```

---

**Endpoint:**

### 🔹 `GET {__ENDPOINT_CLIENTE__}/users?document=CPF_CNPJ`

**Descrição:**
Busca o participante vinculado ao CPF ou CNPJ informado.

**Parâmetros de Query:**

- `document` *(string, obrigatório)* → CPF ou CNPJ do participante.

#### Exemplo de retorno:

```json
{
    "user_id": "XXX.XXX.XXX-XX",
    "name": "TESTE PARTICIPANTE JUNIOR",
    "document": "XXXXXXXXXXX",
    "email": "nidoexemplo@nido.com",
    "phone": "9XXXXXXXX"
}
```

#### Exemplo CURL: | Retornar o participante

**Exemplo de requisição:**

```shell
curl -X GET \
  '__ENDPOINT_CLIENTE__/users?document={cpfCnpj}' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0'
```

---

**Endpoint:**

### 🔹 `POST {__ENDPOINT_CLIENTE__}/incidents`

**Descrição:**
Permite a abertura de uma ocorrência vinculada a um participante.

### `A PEDIDO DA LASTRO, OS CAMPOS SUMMARY E DESCRIPTION FORAM TROCADOS`

O campo description que originalmente era usado para a descrição da ocorrência, passou a ser usado para o assunto (não obrigatório - preenchido automaticamente com "Ocorrência Lais") e o campo summary que originalmente era usado para o assunto, passou a ser usado para o preenchimento do conteúdo da ocorrência. 

**Parâmetros do corpo (JSON):**

- `user_id` *(string, obrigatório)* → ID do contrato.
- `incident_type_id` *(string, obrigatório)* → ID do tipo da ocorrência.
- `summary` *(string, obrigatório)* → Mensagem da ocorrência.
- `description` *(string)* → Assunto da ocorrência.

#### Exemplo de retorno:

```json
{
  "status": "registered",
  "message": "Ocorrência registrada com sucesso.",
  "occurrence_id": XXXXX
}
```

#### Exemplo CURL: | Abertura de uma ocorrência vinculada a um contrato.

**Exemplo de requisição:**

```bash
curl -X POST \
  '__ENDPOINT_CLIENTE__/incidents' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'User-Agent: Nido/1.0.0' \
  --data-urlencode 'user_id={user_id}' \
  --data-urlencode 'incident_type_id={incident_type_id}' \
  --data-urlencode 'summary={summary}'

```

---

## Erros Comuns

- **401 Unauthorized** → Token inválido ou não informado.
- **403 Forbidden** → Permissões insuficientes para acessar o endpoint.
- **400 Bad Request** → Parâmetros obrigatórios ausentes ou inválidos.
- **500 Internal Server Error** → Erro inesperado no servidor.

#### Retornos possíveis:

| Status | Descrição                                   |
| -----: | --------------------------------------------- |
|    200 | Sucesso                                       |
|    400 | Parâmetro inválido ou ausente               |
|    401 | Token de autenticação inválido ou expirado |
|    404 | Nenhum participante ou registro encontrado    |
|    50x | Problema de Token ou Problema Genérico       |

---

## Observações

- Todos os endpoints requerem **JWT válido**.
- O token deve ser renovado conforme a política de expiração definida pelo sistema.
- O campo `document` deve ser informado no formato aceito pelo sistema (com ou sem máscara).
