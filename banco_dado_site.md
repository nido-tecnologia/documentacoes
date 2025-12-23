# Documentação Técnica - Banco de Dados Nido

## Visão Geral

Este banco de dados é utilizado como repositório de dados exportados pelo **CRM Imobiliário** para alimentar o site da imobiliária. O CRM é responsável por manter e sincronizar todos os dados neste banco de dados.

## ⚠️ IMPORTANTE - Regras de Uso

### Responsabilidades do CRM

- **Alimentação de Dados**: O CRM alimenta todas as tabelas diretamente através de processos de exportação e sincronização
- **Manutenção da Estrutura**: O CRM ajusta a estrutura das tabelas quando necessário (adição de colunas, alteração de tipos, criação de índices, etc.)
- **Integridade dos Dados**: O CRM garante a consistência e integridade dos dados exportados

### Responsabilidades dos Consumidores (Site/API)

- **Somente Leitura**: As tabelas devem ser utilizadas **APENAS para consulta (SELECT)**
- **Proibido Modificar**: **NÃO** é permitido realizar operações de INSERT, UPDATE ou DELETE nas tabelas
- **Proibido Alterar Estrutura**: **NÃO** é permitido alterar a estrutura das tabelas (ALTER TABLE, CREATE INDEX, etc.)
- **Sincronização**: Os dados são atualizados automaticamente pelo CRM, não é necessário realizar sincronizações manuais

### Consequências do Não Cumprimento

Modificações nas tabelas podem causar:
- Conflitos com processos de sincronização do CRM
- Perda de dados durante atualizações automáticas
- Inconsistências entre o CRM e o banco de dados
- Falhas no sistema de exportação do CRM

---

## Estrutura do Banco de Dados

O banco de dados está organizado em três grupos principais de tabelas:

1. **Tabelas de Localização (`i_*`)**: Dados geográficos (país, estado, cidade, bairro, região)
2. **Tabelas do Website (`w_*`)**: Dados dos imóveis, profissionais, agências e configurações do site
3. **Tabelas de Controle (`nido_migration`)**: Controle de migrações do sistema

---

## Tabelas de Localização (Prefixo `i_`)

### `i_pais`
Armazena informações sobre países.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codpais` | int(11) | Código único do país (PK) |
| `descricao` | varchar(50) | Nome do país |
| `situacao` | varchar(10) | Situação do registro (ativo/inativo) |

**Relacionamentos**: Relacionado com `i_estado` através de `codpais`

---

### `i_estado`
Armazena informações sobre estados/províncias.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codestado` | int(11) | Código único do estado (PK) |
| `codpais` | int(11) | Código do país (FK → `i_pais.codpais`) |
| `descricao` | varchar(50) | Nome do estado |
| `sigla` | varchar(2) | Sigla do estado (ex: SP, RJ) |
| `situacao` | varchar(10) | Situação do registro |

**Relacionamentos**: 
- Relacionado com `i_pais` através de `codpais`
- Relacionado com `i_cidade` através de `codestado`

---

### `i_cidade`
Armazena informações sobre cidades.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codcidade` | int(11) | Código único da cidade (PK) |
| `codestado` | int(11) | Código do estado (FK → `i_estado.codestado`) |
| `descricao` | varchar(50) | Nome da cidade |
| `cep` | varchar(8) | CEP padrão da cidade |
| `situacao` | varchar(10) | Situação do registro |
| `siglaestado` | varchar(2) | Sigla do estado (redundante, para performance) |
| `imovel` | tinyint(1) | Flag indicando se há imóveis nesta cidade |

**Relacionamentos**: 
- Relacionado com `i_estado` através de `codestado`
- Relacionado com `i_bairro` através de `codcidade`
- Relacionado com `i_regiao` através de `codcidade`

---

### `i_bairro`
Armazena informações sobre bairros.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codbairro` | int(11) | Código único do bairro (PK) |
| `codcidade` | int(11) | Código da cidade (FK → `i_cidade.codcidade`) |
| `maxcep` | varchar(8) | CEP máximo do bairro |
| `descricao` | varchar(72) | Nome do bairro |
| `nomeuso` | varchar(72) | Nome alternativo/uso do bairro |
| `situacao` | varchar(10) | Situação do registro |

**Relacionamentos**: 
- Relacionado com `i_cidade` através de `codcidade`
- Relacionado com `i_regiaobairro` através de `codbairro`
- Relacionado com `w_imovel` através de `codbairro`

---

### `i_regiao`
Armazena informações sobre regiões/zonas das cidades.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codregiao` | int(11) | Código único da região (PK) |
| `codcidade` | int(11) | Código da cidade (FK → `i_cidade.codcidade`) |
| `macrozona` | varchar(20) | Nome da macrozona |
| `zona` | varchar(20) | Nome da zona |
| `descricao` | varchar(50) | Descrição da região |
| `situacao` | varchar(10) | Situação do registro |

**Índices Únicos**: `(codcidade, descricao)` - Garante que não haja regiões duplicadas na mesma cidade

**Relacionamentos**: 
- Relacionado com `i_cidade` através de `codcidade`
- Relacionado com `i_regiaobairro` através de `codregiao`

---

### `i_regiaobairro`
Tabela de relacionamento entre regiões e bairros (N:N).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codregiao` | int(11) | Código da região (FK → `i_regiao.codregiao`) |
| `codbairro` | int(11) | Código do bairro (FK → `i_bairro.codbairro`) |
| `situacao` | varchar(10) | Situação do relacionamento |

**Chave Primária**: `(codregiao, codbairro)` - Garante relacionamento único

**Relacionamentos**: 
- Relacionado com `i_regiao` através de `codregiao`
- Relacionado com `i_bairro` através de `codbairro`

---

## Tabelas do Website (Prefixo `w_`)

### `w_agencia`
Armazena informações sobre as agências imobiliárias.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codagencia` | varchar(2) | Código único da agência (PK) |
| `nomeempresa` | varchar(50) | Nome da empresa |
| `nome` | varchar(50) | Nome comercial |
| `razaosocial` | varchar(70) | Razão social |
| `cep` | varchar(8) | CEP |
| `endereco` | varchar(70) | Endereço |
| `numero` | int(11) | Número do endereço |
| `bairro` | varchar(50) | Bairro |
| `complemento` | varchar(50) | Complemento |
| `cidade` | varchar(50) | Cidade |
| `estado` | varchar(2) | Estado (sigla) |
| `ddd1`, `ddd2`, `dddfax` | varchar(2) | DDD dos telefones |
| `telefone1`, `telefone2` | varchar(10) | Números de telefone |
| `fax` | varchar(10) | Número do fax |
| `email` | varchar(50) | E-mail |
| `creci` | varchar(20) | CRECI da agência |
| `situacao` | varchar(10) | Situação da agência |

**Relacionamentos**: 
- Relacionado com `w_imovel` através de `codagencia`
- Relacionado com `w_profissional` através de `codagencia`
- Relacionado com `web_site` através de `codagencia`

---

### `w_imovel`
**Tabela principal** - Armazena todas as informações dos imóveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência única do imóvel (PK) |
| `dataatualizacao` | datetime | Data da última atualização |
| `datacadastro` | datetime | Data de cadastro |
| `dataalteracaofoto` | datetime | Data da última alteração de foto |
| `dataalteracaofoto_empreendimento` | datetime | Data da última alteração de foto do empreendimento |
| `datahora_pub` | datetime | Data/hora de publicação |
| `codagencia` | varchar(2) | Código da agência responsável |
| `codagencia_publicacao` | varchar(2) | Código da agência de publicação |
| `codtipoimovel` | int(11) | Código do tipo de imóvel (FK → `w_tipoimovel`) |
| `codtiposimplificado` | int(11) | Código do tipo simplificado (FK → `w_tiposimplificado`) |
| `tipoimovel` | varchar(64) | Descrição do tipo de imóvel |
| `tiposimplificado` | varchar(64) | Descrição do tipo simplificado |
| `codtiponegocio` | varchar(1) | Código do tipo de negócio (V=Venda, L=Locacao, etc) |
| `codtipoutilizacao` | char(1) | Código do tipo de utilização |
| `residencial`, `comercial`, `rural`, `industrial` | char(1) | Flags de tipo de uso |
| `codtipoorigem` | int(11) | Código da origem (FK → `w_tipoorigem`) |
| `lancamento` | int(1) | Flag de lançamento (1=Sim, 0=Não) |
| `cep` | varchar(8) | CEP do imóvel |
| `logradouro` | varchar(25) | Tipo de logradouro |
| `endereco` | varchar(512) | Endereço completo |
| `numero` | blob | Número do endereço |
| `codcidade` | int(11) | Código da cidade (FK → `i_cidade`) |
| `cidade` | varchar(64) | Nome da cidade |
| `estado` | char(2) | Sigla do estado |
| `regiao` | varchar(72) | Região |
| `bairro` | varchar(72) | Bairro |
| `codbairro` | int(10) | Código do bairro (FK → `i_bairro`) |
| `codregiao1`, `codregiao2`, `codregiao3` | varchar(72) | Códigos de regiões |
| `valor` | double | Valor geral |
| `disponivelvenda` | tinyint(1) | Disponível para venda (1=Sim, 0=Não) |
| `valorvenda` | double | Valor de venda |
| `valorvendam2` | float | Valor de venda por m² |
| `disponivellocacao` | tinyint(1) | Disponível para locação |
| `valorlocacao` | double | Valor de locação |
| `periodolocacao` | varchar(32) | Período de locação |
| `valorlocacaom2` | float | Valor de locação por m² |
| `disponiveltemporada` | tinyint(1) | Disponível para temporada |
| `valortemporada` | double | Valor da temporada |
| `qtdepessoas` | int(11) | Quantidade de pessoas (temporada) |
| `qtdelevadores` | int(11) | Quantidade de elevadores |
| `condominio` | double | Valor do condomínio |
| `iptu` | double | Valor do IPTU |
| `tipoiptu` | varchar(10) | Tipo de IPTU |
| `dormitorios` | smallint(6) | Quantidade de dormitórios |
| `suites` | smallint(6) | Quantidade de suítes |
| `vagas` | smallint(6) | Quantidade de vagas |
| `areatotal` | int(11) | Área total (m²) |
| `areautil` | int(11) | Área útil (m²) |
| `metragem` | varchar(20) | Metragem |
| `promocao` | text | Texto promocional |
| `titulo` | text | Título do imóvel |
| `detunidade` | text | Detalhes da unidade |
| `detcondominio` | text | Detalhes do condomínio |
| `tag` | text | Tags do imóvel |
| `comfoto` | tinyint(1) | Possui foto |
| `quantidade_fotos` | int(10) | Quantidade de fotos |
| `comtexto` | tinyint(1) | Possui texto |
| `comfinanciamento` | tinyint(1) | Aceita financiamento |
| `valorentrada` | double | Valor de entrada |
| `valorparcela` | double | Valor da parcela |
| `situacao` | int(1) | Situação do imóvel |
| `situacao_imovel` | varchar(20) | Situação do imóvel (texto) |
| `salas` | smallint(6) | Quantidade de salas |
| `codprofissional` | varchar(6) | Código do profissional responsável |
| `empreendimento` | varchar(150) | Nome do empreendimento |
| `complemento` | varchar(150) | Complemento do endereço |
| `video` | varchar(1024) | URL do vídeo |
| `latitude` | double | Latitude (coordenada geográfica) |
| `longitude` | double | Longitude (coordenada geográfica) |
| `condpagamento` | text | Condições de pagamento |
| `imediacoes` | varchar(50) | Informações sobre imediações |
| `codclassificacao` | int(3) | Código da classificação (FK → `w_classificacao`) |
| `banheiros` | int(11) | Quantidade de banheiros |
| `exclusividade` | tinyint(1) | Exclusividade |
| `plantao` | tinyint(1) | Em plantão |
| `remanescente` | tinyint(1) | Remanescente |
| `aceitapermuta` | tinyint(1) | Aceita permuta |
| `possuirenda` | tinyint(1) | Possui renda |
| `edificio` | varchar(150) | Nome do edifício |
| `situacao_mobilia` | varchar(32) | Situação da mobília |
| `codigo_anterior` | varchar(20) | Código anterior do imóvel |
| `textoimpressao` | varchar(2048) | Texto para impressão |
| `regra_sistema` | varchar(3) | Regra do sistema |
| `imovel_desabilitado` | tinyint(1) | Imóvel desabilitado (default: 0) |
| `caracteristica_unidade` | varchar(2048) | Características da unidade |
| `caracteristica_condominio` | varchar(2048) | Características do condomínio |
| `dataatualizacao_pregao` | datetime | Data de atualização do pregão |
| `zoneamento` | varchar(32) | Zoneamento |

**Índices Principais**:
- `codtipoimovel` - Para filtros por tipo
- `codclassificacao` - Para filtros por classificação
- `disponivelvenda` - Para filtros de disponibilidade
- `disponivellocacao` - Para filtros de disponibilidade
- `codbairro` - Para filtros por bairro
- `situacao` - Para filtros por situação
- `datacadastro` - Para ordenação por data

**Relacionamentos**: 
- Relacionado com `w_foto` através de `ref`
- Relacionado com `w_imovelprofissional` através de `ref`
- Relacionado com `w_imovel_pois` através de `ref`
- Relacionado com `w_imovel_poi_distancia` através de `ref`
- Relacionado com `w_link` através de `ref`
- Relacionado com `w_pregao` através de `ref_imovel`

---

### `w_foto`
Armazena as fotos dos imóveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência do imóvel (FK → `w_imovel.ref`) |
| `seq` | int(11) | Sequência da foto (ordem) |
| `titulo` | varchar(30) | Título da foto |
| `foto` | longblob | Dados binários da foto |
| `destaque` | tinyint(1) | Foto de destaque (1=Sim, 0=Não) |

**Chave Primária**: `(ref, seq)` - Permite múltiplas fotos por imóvel

**Relacionamentos**: Relacionado com `w_imovel` através de `ref`

---

### `w_imovelprofissional`
Tabela de relacionamento entre imóveis e profissionais (N:N).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência do imóvel (FK → `w_imovel.ref`) |
| `relacao` | varchar(60) | Tipo de relação (ex: "Proprietário", "Corretor", etc) |
| `codprofissional` | varchar(20) | Código do profissional (FK → `w_profissional.codprofissional`) |
| `codprofagencia` | varchar(20) | Código da agência do profissional |
| `codprofequipe` | varchar(20) | Código da equipe do profissional |
| `ordem` | int(11) | Ordem de exibição |

**Chave Primária**: `(ref, relacao, codprofissional)` - Permite múltiplos profissionais por imóvel

**Relacionamentos**: 
- Relacionado com `w_imovel` através de `ref`
- Relacionado com `w_profissional` através de `codprofissional`

---

### `w_profissional`
Armazena informações sobre os profissionais (corretores).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codprofissional` | varchar(6) | Código único do profissional (PK) |
| `codagencia` | varchar(2) | Código da agência (FK → `w_agencia.codagencia`) |
| `nomeuso` | varchar(40) | Nome de uso/apresentação |
| `foto` | mediumblob | Foto do profissional |
| `obs` | text | Observações |
| `ddd1`, `ddd2`, `dddcelular` | varchar(3) | DDD dos telefones |
| `fone1`, `fone2` | varchar(10) | Números de telefone |
| `celular` | varchar(10) | Número do celular |
| `email1`, `email2`, `email3` | varchar(50) | E-mails |
| `codequipe` | int(3) | Código da equipe |
| `equipenome` | varchar(50) | Nome da equipe |
| `nextel_id` | varchar(20) | ID Nextel |
| `creci` | varchar(10) | CRECI do profissional |
| `ramal` | varchar(5) | Ramal |
| `idiomas` | varchar(50) | Idiomas falados |
| `nivel` | varchar(40) | Nível do profissional |

**Relacionamentos**: 
- Relacionado com `w_agencia` através de `codagencia`
- Relacionado com `w_imovelprofissional` através de `codprofissional`

---

### `w_tipoimovel`
Armazena os tipos de imóveis disponíveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codtipoimovel` | int(11) | Código único do tipo (PK) |
| `descricao` | varchar(50) | Descrição do tipo |
| `descricao_slug` | varchar(50) | Slug da descrição (URL-friendly) |
| `codtiposimplificado` | int(10) | Código do tipo simplificado |
| `validacao` | char(1) | Flag de validação |
| `situacao` | varchar(10) | Situação do tipo |
| `publicavel` | tinyint(4) | Pode ser publicado (1=Sim, 0=Não) |

**Relacionamentos**: Relacionado com `w_imovel` através de `codtipoimovel`

---

### `w_tiposimplificado`
Armazena os tipos simplificados de imóveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codtiposimplificado` | int(10) | Código único do tipo simplificado (PK) |
| `descricao` | varchar(50) | Descrição do tipo |
| `descricao_slug` | varchar(50) | Slug da descrição |
| `situacao` | varchar(10) | Situação do tipo |

**Relacionamentos**: Relacionado com `w_imovel` através de `codtiposimplificado`

---

### `w_tipoorigem`
Armazena os tipos de origem dos imóveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codtipoorigem` | int(10) | Código único da origem (PK) |
| `descricao` | varchar(50) | Descrição da origem |
| `situacao` | varchar(10) | Situação do tipo |

**Relacionamentos**: Relacionado com `w_imovel` através de `codtipoorigem`

---

### `w_classificacao`
Armazena as classificações dos imóveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codclassificacao` | int(11) | Código único da classificação (PK) |
| `descricao` | varchar(100) | Descrição da classificação |
| `situacao` | varchar(10) | Situação da classificação |

**Relacionamentos**: Relacionado com `w_imovel` através de `codclassificacao`

---

### `w_tag`
Armazena as tags disponíveis para os imóveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codtag` | int(11) | Código único da tag (PK) |
| `codtipotag` | int(11) | Código do tipo de tag (FK → `w_tipotag`) |
| `titulo` | varchar(100) | Título da tag |
| `descricao` | text | Descrição da tag |
| `palavras_chave` | text | Palavras-chave associadas |
| `slug` | varchar(100) | Slug da tag (URL-friendly) |
| `situacao` | varchar(10) | Situação da tag |

**Relacionamentos**: Relacionado com `w_tipotag` através de `codtipotag`

---

### `w_tipotag`
Armazena os tipos de tags.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `codtipotag` | int(11) | Código único do tipo de tag (PK) |
| `descricao` | text | Descrição do tipo |
| `slug` | varchar(100) | Slug do tipo |
| `situacao` | varchar(10) | Situação do tipo |

**Relacionamentos**: Relacionado com `w_tag` através de `codtipotag`

---

### `w_link`
Armazena links relacionados aos imóveis (vídeos, tours virtuais, etc).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência do imóvel (FK → `w_imovel.ref`) |
| `ordem` | int(11) | Ordem de exibição |
| `titulo` | varchar(64) | Título do link |
| `tipo` | varchar(64) | Tipo do link |
| `url` | varchar(20148) | URL do link |

**Chave Primária**: `(ref, ordem)` - Permite múltiplos links por imóvel

**Relacionamentos**: Relacionado com `w_imovel` através de `ref`

---

### `w_imovel_pois`
Armazena pontos de interesse (POIs) dos imóveis em formato binário.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência do imóvel (FK → `w_imovel.ref`) |
| `pois` | blob | Dados binários dos POIs (provavelmente JSON serializado) |

**Relacionamentos**: Relacionado com `w_imovel` através de `ref`

---

### `w_imovel_poi_distancia`
Armazena distâncias entre imóveis e pontos de interesse.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência do imóvel (FK → `w_imovel.ref`) |
| `tipo` | varchar(48) | Tipo do POI |
| `nome` | varchar(72) | Nome do POI |
| `distancia` | double | Distância em metros/quilômetros |

**Chave Primária**: `(ref, tipo, nome)` - Permite múltiplos POIs por imóvel

**Índices**: 
- `tipo` - Para filtros por tipo de POI
- `nome` - Para filtros por nome
- `tipo_nome` - Índice composto para consultas combinadas

**Relacionamentos**: Relacionado com `w_imovel` através de `ref`

---

### `w_pregao`
Armazena informações sobre pregões/leilões de imóveis.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência única do pregão (PK) |
| `ref_imovel` | varchar(13) | Referência do imóvel relacionado |
| `tipo_negocio` | varchar(10) | Tipo de negócio |
| `valorpregao` | double | Valor do pregão |
| `texto` | text | Texto descritivo |
| `datahora` | datetime | Data/hora do pregão |
| `validade` | datetime | Data de validade |

**Relacionamentos**: Relacionado com `w_imovel` através de `ref_imovel`

---

### `w_pregaoprofissional`
Armazena informações sobre profissionais relacionados aos pregões.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `ref` | varchar(16) | Referência do pregão (FK → `w_pregao.ref`) |
| `codprofissional` | varchar(20) | Código do profissional |
| `codprofagencia` | varchar(20) | Código da agência do profissional |
| `codprofequipe` | varchar(20) | Código da equipe do profissional |

**Relacionamentos**: Relacionado com `w_pregao` através de `ref`

---

### `w_lead`
Armazena leads gerados pelo site (contatos de interesse).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | int(11) | ID único do lead (PK, AUTO_INCREMENT) |
| `tipo_lead` | enum('Unidade','Empreendimento') | Tipo do lead |
| `referencia` | char(50) | Referência do imóvel relacionado |
| `codagencia_publicacao` | varchar(2) | Código da agência de publicação |
| `tipoimovel` | int(11) | Tipo de imóvel |
| `codtipoutilizacao` | char(1) | Tipo de utilização |
| `codregiao` | varchar(72) | Código da região |
| `disponivelvenda` | tinyint(4) | Interesse em venda |
| `valorvenda` | double | Valor de venda desejado |
| `disponivellocacao` | tinyint(4) | Interesse em locação |
| `valorlocacao` | double | Valor de locação desejado |
| `cliente` | varchar(100) | Nome do cliente |
| `email` | varchar(200) | E-mail do cliente |
| `telefone` | char(20) | Telefone do cliente |
| `celular` | char(20) | Celular do cliente |
| `gostaria_de` | varchar(200) | O que o cliente gostaria |
| `tipouso` | varchar(200) | Tipo de uso desejado |
| `mensagem` | text | Mensagem do cliente |
| `datahora` | datetime | Data/hora do lead |
| `status` | enum('Pendente','Processado','Erro') | Status do processamento |
| `data_agendamento` | datetime | Data de agendamento |

**Índices**: 
- `datahora` - Para ordenação por data
- `status` - Para filtros por status

**Nota**: Esta tabela pode ser escrita pelo site, mas o processamento é feito pelo CRM.

---

### `w_comunicacao`
Armazena comunicações/contatos recebidos pelo site.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | int(11) | ID único da comunicação (PK, AUTO_INCREMENT) |
| `tipocomunicao` | tinyint(4) | Tipo de comunicação |
| `dados_contato` | blob | Dados do contato (provavelmente JSON serializado) |
| `status` | enum('Pendente','Processado','Erro') | Status do processamento |
| `datahora` | datetime | Data/hora da comunicação |

**Índices**: 
- `datahora` - Para ordenação por data
- `status` - Para filtros por status
- `tipocomunicao` - Para filtros por tipo

**Nota**: Esta tabela pode ser escrita pelo site, mas o processamento é feito pelo CRM.

---

### `web_site`
Armazena configurações e conteúdos do site.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | int(11) | ID único (PK, AUTO_INCREMENT) |
| `codagencia` | char(2) | Código da agência (FK → `w_agencia.codagencia`) |
| `modelo_site` | int(11) | Modelo do site |
| `texto_home` | mediumtext | Texto da página inicial |
| `texto_empresa` | mediumtext | Texto sobre a empresa |
| `texto_faleconosco` | text | Texto da página fale conosco |
| `texto_lgpd` | text | Texto sobre LGPD |
| `titulo_home` | varchar(64) | Título da página inicial |
| `informacoes_principais` | blob | Informações principais (JSON) |
| `endereco` | blob | Dados de endereço (JSON) |
| `contatos` | blob | Dados de contatos (JSON) |
| `informacoes_promocionais` | blob | Informações promocionais (JSON) |
| `paginas_extra` | longblob | Páginas extras (JSON) |
| `informacoes_seo` | blob | Informações de SEO (JSON) |
| `informacoes_google` | blob | Informações do Google (JSON) |
| `image_path` | varchar(255) | Caminho das imagens |
| `favicon` | varchar(255) | Caminho do favicon |
| `banner` | varchar(4092) | URL do banner |
| `imagem_contato` | varchar(255) | Caminho da imagem de contato |
| `informacoes_lais` | blob | Informações LAIS (JSON) |

**Relacionamentos**: Relacionado com `w_agencia` através de `codagencia`

---

## Tabelas de Controle

### `nido_migration`
Tabela de controle de migrações do sistema.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | int(11) | ID único (PK, AUTO_INCREMENT) |
| `migration` | varchar(255) | Nome/identificador da migração |
| `date` | datetime | Data da migração |

**Índices**: `date` - Para ordenação por data

**Nota**: Esta tabela é gerenciada pelo sistema de migrações do CRM.

---

## Convenções e Padrões

### Nomenclatura

- **Prefixo `i_`**: Tabelas de localização geográfica (país, estado, cidade, bairro, região)
- **Prefixo `w_`**: Tabelas do website (imóveis, profissionais, agências, etc)
- **Prefixo `web_`**: Tabelas de configuração do site

### Tipos de Dados Comuns

- **`tinyint(1)`**: Flags booleanas (0 ou 1)
- **`varchar(n)`**: Strings de tamanho variável
- **`text`**: Textos longos
- **`blob`**: Dados binários (geralmente JSON serializado)
- **`datetime`**: Data e hora
- **`double`**: Valores monetários e decimais
- **`int(11)`**: Valores inteiros

### Campos de Situação

Muitas tabelas possuem um campo `situacao` do tipo `varchar(10)` que geralmente contém valores como:
- `"Ativo"` ou `"A"` - Registro ativo
- `"Inativo"` ou `"I"` - Registro inativo

### Campos de Data

- `datacadastro`: Data de criação do registro
- `dataatualizacao`: Data da última atualização
- `datahora`: Data e hora de um evento específico

---

## Relacionamentos Principais

### Hierarquia Geográfica
```
i_pais
  └── i_estado
      └── i_cidade
          ├── i_bairro
          └── i_regiao
              └── i_regiaobairro (N:N)
```

### Estrutura de Imóveis
```
w_imovel (tabela principal)
  ├── w_foto (1:N)
  ├── w_imovelprofissional (N:N)
  ├── w_imovel_pois (1:1)
  ├── w_imovel_poi_distancia (1:N)
  ├── w_link (1:N)
  └── w_pregao (1:N)
      └── w_pregaoprofissional (1:1)
```

### Estrutura de Agência
```
w_agencia
  ├── w_profissional (1:N)
  ├── w_imovel (1:N)
  └── web_site (1:1)
```

### Tabelas de Classificação
```
w_tipoimovel ──┐
w_tiposimplificado ──┐
w_tipoorigem ──┐
w_classificacao ──┐
                └── w_imovel
```

---

## Boas Práticas de Consulta

### Performance

1. **Use índices**: Sempre que possível, filtre por campos que possuem índices:
   - `w_imovel.codtipoimovel`
   - `w_imovel.disponivelvenda`
   - `w_imovel.disponivellocacao`
   - `w_imovel.codbairro`
   - `w_imovel.situacao`

2. **Evite SELECT ***: Selecione apenas os campos necessários

3. **Use LIMIT**: Em listagens, sempre use LIMIT para evitar consultas muito grandes

4. **JOINs eficientes**: Use INNER JOIN quando possível e certifique-se de que as chaves estrangeiras estão indexadas

### Exemplos de Consultas Úteis

```sql
-- Listar imóveis disponíveis para venda em um bairro específico
SELECT ref, titulo, valorvenda, bairro, cidade
FROM w_imovel
WHERE disponivelvenda = 1
  AND codbairro = ?
  AND situacao = 1
ORDER BY datacadastro DESC
LIMIT 20;

-- Buscar fotos de um imóvel ordenadas por sequência
SELECT seq, titulo, foto, destaque
FROM w_foto
WHERE ref = ?
ORDER BY seq ASC;

-- Listar profissionais relacionados a um imóvel
SELECT p.nomeuso, p.email1, p.fone1, ip.relacao
FROM w_imovelprofissional ip
INNER JOIN w_profissional p ON ip.codprofissional = p.codprofissional
WHERE ip.ref = ?
ORDER BY ip.ordem;
```

---

## Atualizações e Manutenção

### Processo de Atualização

1. O CRM realiza exportações periódicas dos dados
2. As atualizações são feitas automaticamente pelo CRM
3. Não é necessário (e não é recomendado) realizar atualizações manuais

### Monitoramento

- Monitore a tabela `nido_migration` para acompanhar atualizações de estrutura
- Use os campos `dataatualizacao` e `datacadastro` para identificar registros recentes
- Verifique o campo `situacao` para filtrar registros ativos

---

**Última atualização**: Esta documentação reflete a estrutura do banco de dados atualizada em 25/11/2025
