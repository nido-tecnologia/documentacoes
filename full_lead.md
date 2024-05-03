# Integração de Leads
Manual de Integração

## Itens necessários
- \_\_ENDPOINT_CLIENTE__
   - Ex: http://sistemadocliente.nidoimovel.com.br/index.php/api
- \_\_TOKEN_CLIENTE__
   - Token JWT, Ex: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9uaWRvLmNvbS5iciIsImF1ZCI6Imh0dHA6XC9cL25pZG9pbW92ZWwuY29tLmJyIiwiaWF0IjoxNTQ2MzA4MDAwLCJuYmYiOjE1NDYzMDgwMDAsImRhdGEiOnsiY2xpZW50ZV9pZCI6IjMxIiwiY29kYWdlbmNpYSI6Ik5JIiwid2ViX2NvbmZpZ19pZCI6IjgyIn19.VARfNYjb9yIY7pB01HcUJipMC1HEnaG028307Elfz1s
   

## Métodos

- `GET {__ENDPOINT_CLIENTE__}/leadOrigin`, método que lista todas as origens possíveis para um novo cadastro de Lead
- `GET {__ENDPOINT_CLIENTE__}/SchedulingTypes`, método que lista todos os tipos de agendamentos possíveis para uma nova solicitação de Agendamento
- `POST {__ENDPOINT_CLIENTE__}/fullLead`, método para o cadastro de um novo Lead

## Segurança

- **Token JWT**, obrigatório utilização como autenticação
- **User-Agent**, obrigatório utilização do agente **Nido** no cabeçalho da requisição

## Campos do JSON para novo Lead
- **id**, identificador do lead no site
- **origin_id**, código da origem do Lead (Portal, Site, Rede Social)
- **created_at**, data de criação do lead, formato *YYYY-MM-DD HH:MM:SS*
- **name**, nome do cliente
- **email**, email do cliente
- **phone**, telefone do cliente
- **mobile**, celular do cliente
- **property_id**, referência do imóvel, ex: XX1234
- **message**, mensagem do lead
- **profile**, objeto contendo o perfil de busca (**EM TESTES**)

## Campos do JSON para nova solicitação de Agendamento
Repete os campos obrigatórios para um novo Lead e adiciona obrigatoriamente os campos listados abaixo.

- **scheduling_date**, data e hora da solicitação de agendamento,  *YYYY-MM-DD HH:MM:SS*
- **scheduling_type**, código do tipo de agendamento (Visita, Fotos, etc), opções são obtidas através do endpoint ***SchedulingTypes***

## Retornos da aplicação
|Status|Descrição|
|-:|-|
|201|Sucesso|
|40x|Problemas na validação dos dados enviados|
|50x|Problema de Token ou Problema Genérico|

## Exemplo CURL | Novo Lead
```shell
curl -X POST \
  __ENDPOINT_CLIENTE__ \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0' \
  -d '{
  "id": "1598457",
  "origin_id": "8",
  "created_at": "2019-01-01 15:59:59",
  "name": "João da Silva",
  "email": "joao@meusite.com.br",
  "phone": "+55 11 3030-2020",
  "mobile": "+55 11 98080-8080",
  "property_id": "NI1234",
  "message": "Olá vi o imóvel NI1234 no seu site",
}'
```
## Exemplo Javascript | Novo Lead
```javascript
var data = {
   "id": "1598457",
   "origin_id": "8",
   "created_at": "2019-01-01 15:59:59",
   "name": "João da Silva",
   "email": "joao@meusite.com.br",
   "phone": "+55 11 3030-2020",
   "mobile": "+55 11 98080-8080",
   "property_id": "NI1234",
   "message": "Olá vi o imóvel NI1234 no seu site",
};

var xhr = new XMLHttpRequest();
xhr.withCredentials = true;

xhr.addEventListener("readystatechange", function () {
  if (this.readyState === 4) {
    console.log(this.responseText);
  }
});

xhr.open("POST", "__ENDPOINT_CLIENTE__");
xhr.setRequestHeader("Authorization", "Bearer __TOKEN_CLIENTE__");
xhr.setRequestHeader("User-Agent", "Nido/1.0.0");
xhr.setRequestHeader("Content-Type", "application/json");

xhr.send(JSON.stringify(data));
```

## Exemplo PHP | Novo Lead
```php
$request = new HttpRequest();
$request->setUrl('__ENDPOINT_CLIENTE__');
$request->setMethod(HTTP_METH_POST);

$body = json_encode(array(
    "id" => "1598457",
    "origin_id" => "8",
    "created_at" => "2019-01-01 15:59:59",
    "name" => "João da Silva",
    "email" => "joao@meusite.com.br",
    "phone" => "+55 11 3030-2020",
    "mobile" => "+55 11 98080-8080",
    "property_id" => "NI1234",
    "message" => "Olá vi o imóvel NI1234 no seu site",
));

$request->setHeaders(array(
  'Content-Type' => 'application/json',
  'User-Agent' => 'Nido/1.0.0',
  'Authorization' => 'Bearer __TOKEN_CLIENTE__'
));

$request->setBody($body);

try {
  $response = $request->send();

  echo $response->getBody();
} catch (HttpException $ex) {
  echo $ex;
}
```

## Exemplo C | Novo Lead
```C#
var client = new RestClient("__ENDPOINT_CLIENTE__");
var request = new RestRequest(Method.POST);

request.AddHeader("Content-Type", "application/json");
request.AddHeader("User-Agent", "Nido/1.0.0");
request.AddHeader("Authorization", "Bearer __TOKEN_CLIENTE__");

var body = new {
   "id" = "1598457",
   "origin_id" = "8",
   "created_at" = "2019-01-01 15:59:59",
   "name" = "João da Silva",
   "email" = "joao@meusite.com.br",
   "phone" = "+55 11 3030-2020",
   "mobile" = "+55 11 98080-8080",
   "property_id" = "NI1234",
   "message" = "Olá vi o imóvel NI1234 no seu site"
};

request.AddParameter("undefined", JsonConvert.SerializeObject(body), ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

## Exemplo CURL | Nova solicitação Agendamento
```shell
curl -X POST \
  __ENDPOINT_CLIENTE__ \
  -H 'Authorization: Bearer __TOKEN_CLIENTE__' \
  -H 'Content-Type: application/json' \
  -H 'User-Agent: Nido/1.0.0' \
  -d '{
  "id": "1598457",
  "origin_id": "8",
  "created_at": "2019-01-01 15:59:59",
  "name": "João da Silva",
  "email": "joao@meusite.com.br",
  "phone": "+55 11 3030-2020",
  "mobile": "+55 11 98080-8080",
  "property_id": "NI1234",
  "message": "Olá vi o imóvel NI1234 no seu site",
  "scheduling_date": "2022-01-10 10:15:00",
  "scheduling_type": "1"
}'
```
## Exemplo Javascript | Nova solicitação Agendamento
```javascript
var data = {
   "id": "1598457",
   "origin_id": "8",
   "created_at": "2019-01-01 15:59:59",
   "name": "João da Silva",
   "email": "joao@meusite.com.br",
   "phone": "+55 11 3030-2020",
   "mobile": "+55 11 98080-8080",
   "property_id": "NI1234",
   "message": "Olá vi o imóvel NI1234 no seu site",
   "scheduling_date": "2022-01-10 10:15:00",
   "scheduling_type": "1"
};

var xhr = new XMLHttpRequest();
xhr.withCredentials = true;

xhr.addEventListener("readystatechange", function () {
  if (this.readyState === 4) {
    console.log(this.responseText);
  }
});

xhr.open("POST", "__ENDPOINT_CLIENTE__");
xhr.setRequestHeader("Authorization", "Bearer __TOKEN_CLIENTE__");
xhr.setRequestHeader("User-Agent", "Nido/1.0.0");
xhr.setRequestHeader("Content-Type", "application/json");

xhr.send(JSON.stringify(data));
```

## Exemplo PHP | Nova solicitação Agendamento
```php
$request = new HttpRequest();
$request->setUrl('__ENDPOINT_CLIENTE__');
$request->setMethod(HTTP_METH_POST);

$body = json_encode(array(
    "id" => "1598457",
    "origin_id" => "8",
    "created_at" => "2019-01-01 15:59:59",
    "name" => "João da Silva",
    "email" => "joao@meusite.com.br",
    "phone" => "+55 11 3030-2020",
    "mobile" => "+55 11 98080-8080",
    "property_id" => "NI1234",
    "message" => "Olá vi o imóvel NI1234 no seu site",
    "scheduling_date": "2022-01-10 10:15:00",
    "scheduling_type": "1"
));

$request->setHeaders(array(
  'Content-Type' => 'application/json',
  'User-Agent' => 'Nido/1.0.0',
  'Authorization' => 'Bearer __TOKEN_CLIENTE__'
));

$request->setBody($body);

try {
  $response = $request->send();

  echo $response->getBody();
} catch (HttpException $ex) {
  echo $ex;
}
```

## Exemplo C | Nova solicitação Agendamento
```C#
var client = new RestClient("__ENDPOINT_CLIENTE__");
var request = new RestRequest(Method.POST);

request.AddHeader("Content-Type", "application/json");
request.AddHeader("User-Agent", "Nido/1.0.0");
request.AddHeader("Authorization", "Bearer __TOKEN_CLIENTE__");

var body = new {
   "id" = "1598457",
   "origin_id" = "8",
   "created_at" = "2019-01-01 15:59:59",
   "name" = "João da Silva",
   "email" = "joao@meusite.com.br",
   "phone" = "+55 11 3030-2020",
   "mobile" = "+55 11 98080-8080",
   "property_id" = "NI1234",
   "message" = "Olá vi o imóvel NI1234 no seu site",
   "scheduling_date": "2022-01-10 10:15:00",
   "scheduling_type": "1"
};

request.AddParameter("undefined", JsonConvert.SerializeObject(body), ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

