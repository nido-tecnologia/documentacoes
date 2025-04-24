# API Nido Adm
Manual de Integração

## Itens necessários
- \_\_ENDPOINT_CLIENTE__
   - Ex: https://app.nidoadm.com.br/sistemadocliente/index.php/apiConnector/
- \_\_TOKEN_CLIENTE__
   - Token JWT, Ex: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9uaWRvLmNvbS5iciIsImF1ZCI6Imh0dHA6XC9cL25pZG9pbW92ZWwuY29tLmJyIiwiaWF0IjoxNTQ2MzA4MDAwLCJuYmYiOjE1NDYzMDgwMDAsImRhdGEiOnsiY2xpZW50ZV9pZCI6IjMxIiwiY29kYWdlbmNpYSI6Ik5JIiwid2ViX2NvbmZpZ19pZCI6IjgyIn19.VARfNYjb9yIY7pB01HcUJipMC1HEnaG028307Elfz1s
   

## Métodos

- `GET {__ENDPOINT_CLIENTE__}/retornarBoletos?document=CPF_CNPJ`, Retorna todos os boletos disponíveis associados ao CPF/CNPJ informado do participante.
- `GET {__ENDPOINT_CLIENTE__}/visualizarBoleto?boletoId=ID&cpfCnpj=CPF_CNPJ`, Exibe o boleto em PDF para visualização ou impressão.

## Segurança
- **Token JWT**: obrigatório como método de autenticação  
- **User-Agent**: obrigatório utilizar o agente **Nido** no cabeçalho da requisição

## Métodos Disponíveis

### 🔹 `GET {__ENDPOINT_CLIENTE__}/retornarBoletos?document=CPF_CNPJ`  
Retorna todos os boletos disponíveis associados ao CPF/CNPJ informado do participante.

#### Parâmetros:
- `document` (string): CPF ou CNPJ do cliente

#### Retornos possíveis:
| Status | Descrição |
|--------:|-----------|
| 200 | Sucesso |
| 400 | Parâmetro inválido ou ausente |
| 401 | Token de autenticação inválido ou expirado |
| 404 | Nenhum cliente ou boleto encontrado |
| 50x | Problema de Token ou Problema Genérico |

#### Exemplo CURL: | Retornar todos os boletos
```shell
curl -X GET \
  '__ENDPOINT_CLIENTE__/retornarBoletos?document={cpf_cnpj}' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0'
```

## Exemplo Javascript | Retornar todos os boletos
```javascript
// WARNING: For GET requests, body is set to null by browsers.
// AVISO: Para solicitações GET, o corpo é definido como nulo pelos navegadores.

var xhr = new XMLHttpRequest();
xhr.withCredentials = true;

xhr.addEventListener("readystatechange", function() {
  if(this.readyState === 4) {
    console.log(this.responseText);
  }
});

xhr.open("GET", "__ENDPOINT_CLIENTE__/retornarBoletos?document={cpf_cnpj}");
xhr.setRequestHeader("Authorization", "Bearer __TOKEN_CLIENTE__");
xhr.setRequestHeader("Content-Type", "application/json");
xhr.setRequestHeader("User-Agent", "Nido/1.0.0");

xhr.send();
```

## Exemplo PHP | Retornar todos os boletos
```php
require_once 'HTTP/Request2.php';
$request = new HTTP_Request2();
$request->setUrl('__ENDPOINT_CLIENTE__/retornarBoletos?document={cpf_cnpj}');
$request->setMethod(HTTP_Request2::METHOD_GET);
$request->setConfig(array(
  'follow_redirects' => TRUE
));
$request->setHeader(array(
  'Authorization' => 'Bearer __TOKEN_CLIENTE__',
  'Content-Type' => 'application/json',
  'User-Agent' => 'Nido/1.0.0'
));

try {
  $response = $request->send();
  
  if (isset($response)) {
      echo $response->getBody();
  } else {
      echo 'Unexpected HTTP status: ' . $response->getStatus() . ' ' . $response->getReasonPhrase();
  }
}
catch(HTTP_Request2_Exception $e) {
  echo 'Error: ' . $e->getMessage();
}
```

## Exemplo C# | Retornar todos os boletos
```C#
using var client = new HttpClient();
client.DefaultRequestHeaders.Add("Authorization", "Bearer __TOKEN_CLIENTE__");
client.DefaultRequestHeaders.Add("User-Agent", "Nido");

var url = "__ENDPOINT_CLIENTE__/retornarBoletos?document={cpf_cnpj}";
var response = await client.GetAsync(url);
var body = await response.Content.ReadAsStringAsync();

Console.WriteLine(body);
```

---

### 🔹 `GET {__ENDPOINT_CLIENTE__}/visualizarBoleto?boletoId=ID&cpfCnpj=CPF_CNPJ`  
Exibe o boleto em PDF para visualização ou impressão.

#### Parâmetros:
- `boletoId` (int): ID do boleto
- `cpfCnpj` (string): CPF ou CNPJ do cliente

#### Retornos possíveis:
| Status | Descrição |
|--------:|-----------|
| 200 | Sucesso |
| 400 | Parâmetros obrigatórios ausentes ou inválidos |
| 401 | Token de autenticação inválido ou inesperado |
| 404 | Boleto ou cliente não encontrado |
| 500 | Erro interno (HTML vazio, falha ao salvar arquivo, etc.) |
| 50x | Problema de Token ou Problema Genérico |

#### Exemplo CURL: | Exibir o boleto
```shell
curl -X GET \
  '__ENDPOINT_CLIENTE__/visualizarBoleto?boletoId={boleto_id}&cpfCnpj={cpf_cnpj}' \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0'
```

## Exemplo Javascript | Exibir o boleto
```javascript
// WARNING: For GET requests, body is set to null by browsers.
// AVISO: Para solicitações GET, o corpo é definido como nulo pelos navegadores.

var xhr = new XMLHttpRequest();
xhr.withCredentials = true;

xhr.addEventListener("readystatechange", function() {
  if(this.readyState === 4) {
    console.log(this.responseText);
  }
});

xhr.open("GET", "__ENDPOINT_CLIENTE__/visualizarBoleto?boletoId={boleto_id}&cpfCnpj={cpf_cnpj}");
xhr.setRequestHeader("Authorization", "Bearer __TOKEN_CLIENTE__");
xhr.setRequestHeader("Content-Type", "application/json");
xhr.setRequestHeader("User-Agent", "Nido/1.0.0");

xhr.send();
```

## Exemplo PHP | Exibir o boleto
```php
require_once 'HTTP/Request2.php';

$request = new HTTP_Request2();

$request->setUrl('__ENDPOINT_CLIENTE__/visualizarBoleto?boletoId={boleto_id}&cpfCnpj={cpf_cnpj}');
$request->setMethod(HTTP_Request2::METHOD_GET);

$request->setConfig(array(
  'follow_redirects' => TRUE
));

$request->setHeader(array(
  'Authorization' => 'Bearer __TOKEN_CLIENTE__',
  'Content-Type' => 'application/json',
  'User-Agent' => 'Nido/1.0.0'
));

try {
  $response = $request->send();
  
  if (isset($response)) {
      echo $response->getBody();
  } else {
      echo 'Unexpected HTTP status: ' . $response->getStatus() . ' ' . $response->getReasonPhrase();
  }
}
catch(HTTP_Request2_Exception $e) {
  echo 'Error: ' . $e->getMessage();
}
```

## Exemplo C# | Exibir o boleto
```C#
using var client = new HttpClient();
client.DefaultRequestHeaders.Add("Authorization", "Bearer __TOKEN_CLIENTE__");
client.DefaultRequestHeaders.Add("User-Agent", "Nido");

var url = "__ENDPOINT_CLIENTE__/visualizarBoleto?boletoId={boleto_id}&cpfCnpj={cpf_cnpj}";
var response = await client.GetAsync(url);
var body = await response.Content.ReadAsStringAsync();

Console.WriteLine(body);
```
