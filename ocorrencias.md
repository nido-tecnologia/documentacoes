# API Nido Adm

Manual de Integração

## Itens necessários

- \_\_ENDPOINT_CLIENTE__
  - Ex: https://app.nidoadm.com.br/sistemadocliente/index.php/apiConnector
- \_\_TOKEN_CLIENTE__
  - Token JWT, Ex: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9uaWRvLmNvbS5iciIsImF1ZCI6Imh0dHA6XC9cL25pZG9pbW92ZWwuY29tLmJyIiwiaWF0IjoxNTQ2MzA4MDAwLCJuYmYiOjE1NDYzMDgwMDAsImRhdGEiOnsiY2xpZW50ZV9pZCI6IjMxIiwiY29kYWdlbmNpYSI6Ik5JIiwid2ViX2NvbmZpZ19pZCI6IjgyIn19.VARfNYjb9yIY7pB01HcUJipMC1HEnaG028307Elfz1s

## Métodos

- `GET {__ENDPOINT_CLIENTE__}/tiposOcorrencia`, Exibe os tipos de ocorrência.
- `GET {__ENDPOINT_CLIENTE__}/contratos?cpfCnpj=CPF_CNPJ`, Exibe os contratos vinculados ao participante pelo numero do documento.
- `POST {__ENDPOINT_CLIENTE__}/aberturaOcorrencia`, Cria com os dados do formulário a nova ocorrência.

Esta API utiliza **JWT Token** para autenticação e disponibiliza endpoints relacionados a **ocorrências** e **contratos**.

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

### 🔹 `GET {__ENDPOINT_CLIENTE__}/tiposOcorrencia`

**Descrição:**
Retorna a lista de tipos de ocorrência disponíveis no sistema.

#### Exemplo de retorno:

```json
[
  {
    "id": 1,
    "name": "Problema técnico"
  },
  {
    "id": 2,
    "name": "Dúvida comercial"
  }
]
```

#### Exemplo CURL: | Retornar todos os tipos de ocorrências

**Exemplo de requisição:**

```shell
curl -X GET \
  '__ENDPOINT_CLIENTE__/tiposOcorrencia' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0'
```

---

**Endpoint:**

### 🔹 `GET {__ENDPOINT_CLIENTE__}/contratos?cpfCnpj=CPF_CNPJ`

**Descrição:**
Busca os contratos vinculados ao CPF ou CNPJ informado.

**Parâmetros de Query:**

- `cpfCnpj` *(string, obrigatório)* → CPF ou CNPJ do participante.

#### Exemplo de retorno:

```json
[
  {
    "id": "id-usuario-XXX",
    "name": "João XXXXX XXXXXX",
    "document": "1234567XXXX",
    "emails": "joao.XXXXX@exemplo.com",
    "phones": "XXXXX-XXXX",
    "contracts": [
      {
        "contract_id": "contract-XXX",
        "address": {
          "street": "Avenida XX XXXXX XXXXXX",
          "number": "XXX",
          "complement": "Apto XX",
          "neighborhood": "Vila XXXXXX",
          "city": "São Paulo",
          "state": "SP",
          "zip_code": "XXXXXXXX"
        },
        "contractDeadline": "XXXX-XX-XX"
      }
    ]
  }
]
```

#### Exemplo CURL: | Retornar todos os contratos do participante

**Exemplo de requisição:**

```shell
curl -X GET \
  '__ENDPOINT_CLIENTE__/contratos?cpfCnpj={cpfCnpj}' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0'
```

---

**Endpoint:**

### 🔹 `POST {__ENDPOINT_CLIENTE__}/aberturaOcorrencia`

**Descrição:**
Permite a abertura de uma ocorrência vinculada a um contrato.

**Parâmetros do corpo (x-www-form-urlencoded):**

- `contract_id` *(int, obrigatório)* → ID do contrato.
- `type_occurrence_id` *(int, obrigatório)* → ID do tipo da ocorrência.
- `subject` *(string, obrigatório)* → Assunto da ocorrência.
- `description` *(string, obrigatório)* → Descrição detalhada.

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
  '__ENDPOINT_CLIENTE__/aberturaOcorrencia' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'User-Agent: Nido/1.0.0' \
  --data-urlencode 'contract_id={contract_id}' \
  --data-urlencode 'type_occurrence_id={type_occurrence_id}' \
  --data-urlencode 'subject={subject}' \
  --data-urlencode 'description={description}'

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
- O campo `cpfCnpj` deve ser informado no formato aceito pelo sistema (com ou sem máscara).
