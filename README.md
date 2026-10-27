# KDEFY SMS API

API REST para integração com serviços de números virtuais e recebimento de códigos de verificação por SMS. Permite consultar países, serviços, operadoras, preços e disponibilidade, realizar compras individuais ou em lote e gerenciar ativações por meio de requisições HTTP.

Todas as respostas são retornadas em JSON. Os valores financeiros são expressos em reais (BRL).

- **Base URL:** <https://kdefy-sms.com/api/v1>
- **Documentação interativa:** <https://kdefy-sms.com/api/v1/docs>
- **Especificação OpenAPI 3:** <https://kdefy-sms.com/api/v1/openapi.json>

## Sumário

- [Visão geral](#visão-geral)
- [Início rápido](#início-rápido)
- [Autenticação](#autenticação)
- [Formato das respostas](#formato-das-respostas)
- [Endpoints disponíveis](#endpoints-disponíveis)
- [Consulta de países, serviços e ofertas](#consulta-de-países-serviços-e-ofertas)
- [Compra de números](#compra-de-números)
- [Recebimento de códigos SMS](#recebimento-de-códigos-sms)
- [Estados da ativação](#estados-da-ativação)
- [Ativações e transações](#ativações-e-transações)
- [Idempotência](#idempotência)
- [Limites de requisição](#limites-de-requisição)
- [Tratamento de erros](#tratamento-de-erros)
- [Exemplos de integração](#exemplos-de-integração)
- [Recursos adicionais](#recursos-adicionais)
- [Suporte](#suporte)

## Visão geral

A KDEFY SMS API permite integrar as funcionalidades da plataforma a aplicações, sistemas e serviços externos.

Entre os principais recursos disponíveis estão:

- Consulta de países, serviços e operadoras.
- Verificação de preços e disponibilidade antes da compra.
- Compra de números virtuais individuais ou em lote.
- Acompanhamento de ativações e recebimento de códigos SMS.
- Cancelamento e finalização de ativações conforme as regras da operação.
- Consulta de saldo, movimentações financeiras e histórico de ativações.
- Controle de preço máximo por compra.
- Proteção contra requisições duplicadas por meio de idempotência.

## Início rápido

Para começar a utilizar a API, siga estas etapas.

### 1. Crie sua conta

Acesse <https://kdefy-sms.com> e cadastre-se na plataforma. Adicione saldo à carteira utilizando uma das formas de pagamento disponíveis, como PIX, cartão ou criptomoedas.

### 2. Gere sua chave de API

Acesse <https://kdefy-sms.com/app/api> e gere seu token de acesso. A chave utiliza o prefixo `kdf_` e é enviada ao seu e-mail.

### 3. Faça sua primeira requisição

O exemplo abaixo consulta o saldo e os totais financeiros da conta.

```bash
curl https://kdefy-sms.com/api/v1/balance \
  -H "X-API-Key: $KDEFY_KEY"
```

**Exemplo de resposta:**

```json
{
  "status": "success",
  "balance_brl": 25.5,
  "total_spent_brl": 12.3,
  "total_recharged_brl": 37.8,
  "currency": "BRL"
}
```

Nos exemplos deste documento, `KDEFY_KEY` representa sua chave de API armazenada em uma variável de ambiente.

## Autenticação

Todas as requisições autenticadas devem incluir uma chave de API válida. A autenticação pode ser realizada por meio de um dos formatos abaixo.

| Método | Formato |
|---|---|
| `X-API-Key` | `X-API-Key: kdf_seu_token` |
| `Authorization` | `Authorization: Bearer kdf_seu_token` |
| Query string | `?api_key=kdf_seu_token` |

O uso dos headers é recomendado. Enviar a chave pela URL pode expô-la em registros de servidores, proxies e outros sistemas intermediários.

A geração de um novo token invalida o anterior.

### Idioma das mensagens

O idioma das mensagens pode ser definido pelo header `Accept-Language` ou pelo parâmetro `lang`.

Idiomas disponíveis: `pt`, `en`, `de`, `es` e `fr`.

O campo `error` deve ser utilizado para identificar erros na aplicação. O texto de `message` pode variar de acordo com o idioma selecionado.

**Segurança:** mantenha sua chave de API em um ambiente seguro. Não a inclua em repositórios públicos, código frontend ou aplicações distribuídas diretamente aos usuários.

## Formato das respostas

As respostas da API seguem uma estrutura JSON. O campo `status` indica se a operação foi concluída com sucesso ou se ocorreu um erro.

### Resposta de sucesso

```json
{
  "status": "success"
}
```

Os demais campos dependem do endpoint solicitado.

### Resposta de erro

```json
{
  "status": "error",
  "error": "saldo_insuficiente",
  "message": "Saldo insuficiente. Recarregue no site."
}
```

Sempre que possível, trate os erros utilizando o campo `error`, evitando depender exclusivamente do texto de `message`.

Além do conteúdo JSON, considere o código HTTP retornado pela requisição.

## Endpoints disponíveis

Todos os caminhos abaixo são relativos à Base URL `https://kdefy-sms.com/api/v1`.

| Método | Endpoint | Descrição |
|---|---|---|
| `GET` | `/balance` | Consulta saldo e totais financeiros |
| `GET` | `/profile` | Consulta informações da conta e saldo |
| `GET` | `/countries` | Lista os países disponíveis |
| `GET` | `/services` | Lista serviços disponíveis em um país |
| `GET` | `/operators` | Consulta operadoras de um país |
| `GET` | `/offers` | Consulta preços e disponibilidade de ofertas |
| `GET`, `POST` | `/number/get` | Compra um ou vários números |
| `GET` | `/number/status` | Consulta o estado de uma ativação e seus códigos |
| `GET`, `POST` | `/number/cancel` | Solicita o cancelamento de uma ativação |
| `GET`, `POST` | `/number/finish` | Finaliza uma ativação |
| `GET` | `/activations` | Lista ativações da conta |
| `GET` | `/transactions` | Consulta o extrato da carteira |

Para consultar os parâmetros aceitos, os formatos de resposta e os detalhes de cada operação, acesse a [documentação interativa](https://kdefy-sms.com/api/v1/docs).

## Consulta de países, serviços e ofertas

Antes de realizar uma compra, consulte os países e serviços disponíveis. Em seguida, utilize o endpoint de ofertas para verificar os preços e o estoque.

### Consultar países

```bash
curl "https://kdefy-sms.com/api/v1/countries" \
  -H "X-API-Key: $KDEFY_KEY"
```

### Consultar serviços de um país

```bash
curl "https://kdefy-sms.com/api/v1/services?country=73" \
  -H "X-API-Key: $KDEFY_KEY"
```

### Consultar ofertas

```bash
curl "https://kdefy-sms.com/api/v1/offers?country=73&service=tg" \
  -H "X-API-Key: $KDEFY_KEY"
```

**Exemplo de resposta de serviços:**

```json
{
  "status": "success",
  "country": 73,
  "services": [
    {
      "code": "tg",
      "name": "Telegram",
      "available": 412,
      "from_price_brl": 1.9
    }
  ]
}
```

**Exemplo de resposta de ofertas:**

```json
{
  "status": "success",
  "country": 73,
  "service": "tg",
  "offers": [
    {
      "price_brl": 1.9,
      "available": 120
    },
    {
      "price_brl": 2.4,
      "available": 292
    }
  ]
}
```

O campo `from_price_brl` representa o menor preço informado para o serviço no país. Para verificar os valores e a disponibilidade das ofertas antes de comprar, utilize `/offers`.

Os exemplos utilizam `country=73` e `service=tg` apenas para demonstração. Consulte as respostas da API para obter os identificadores válidos.

## Compra de números

O endpoint `/number/get` permite comprar números virtuais para um país e serviço específicos.

São aceitos os métodos `GET` e `POST`. Para novas integrações, prefira `POST` nas operações de compra.

### Parâmetros

| Parâmetro | Obrigatório | Descrição |
|---|---|---|
| `country` | Sim | ID do país, obtido em `/countries` |
| `service` | Sim | Código do serviço, obtido em `/services` |
| `max_price_brl` | Não | Limite máximo de preço aceito |
| `operator` | Não | Operadora desejada; por padrão, qualquer operadora |
| `quantity` | Não | Quantidade de números, de 1 a 50 |
| `request_id` | Não | Chave de idempotência, com 8 a 80 caracteres |

Quando `max_price_brl` é informado, a API procura uma oferta cujo preço não ultrapasse o limite especificado. Se nenhuma oferta for compatível, a compra poderá retornar `preco_abaixo_do_minimo`.

### Exemplo de compra

```bash
curl -X POST https://kdefy-sms.com/api/v1/number/get \
  -H "X-API-Key: $KDEFY_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "country": 73,
    "service": "tg",
    "max_price_brl": 3.00
  }'
```

**Exemplo de resposta:**

```json
{
  "status": "success",
  "id": 981245,
  "service": "tg",
  "service_name": "Telegram",
  "country": 73,
  "country_name": "Brasil",
  "phone": "5511999999999",
  "state": "waiting",
  "state_label": "Aguardando código",
  "code": null,
  "codes": [],
  "price_brl": 1.9,
  "seconds_left": 1200,
  "cancel_in_seconds": 120,
  "balance_brl": 23.6
}
```

Os valores acima são ilustrativos. A disponibilidade, o preço, o saldo e os identificadores dependem da resposta real da API.

### Compra em lote

O parâmetro `quantity` permite solicitar vários números na mesma requisição, com limite de 50 por chamada.

```bash
curl -X POST https://kdefy-sms.com/api/v1/number/get \
  -H "X-API-Key: $KDEFY_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "country": 73,
    "service": "tg",
    "quantity": 10
  }'
```

A resposta de uma compra em lote inclui os campos `requested`, `purchased`, `failed` e `numbers`.

Compras parciais são possíveis. Apenas os números efetivamente adquiridos são cobrados, conforme as regras da API.

## Recebimento de códigos SMS

Após adquirir um número, utilize `/number/status` para consultar o estado da ativação e verificar se um código foi recebido.

```bash
curl "https://kdefy-sms.com/api/v1/number/status?id=981245" \
  -H "X-API-Key: $KDEFY_KEY"
```

**Exemplo de resposta após o recebimento:**

```json
{
  "status": "success",
  "id": 981245,
  "phone": "5511999999999",
  "state": "code_received",
  "code": "482913",
  "codes": [
    {
      "code": "482913",
      "received_at": "2026-10-26T03:40:12.000Z"
    }
  ]
}
```

O campo `id` identifica a ativação. Armazene esse valor para realizar consultas posteriores, finalizar a operação ou solicitar o cancelamento quando permitido.

### Acompanhamento da ativação

Para acompanhar o recebimento do SMS:

1. Consulte `/number/status` utilizando o ID da ativação.
2. Verifique o campo `state`.
3. Enquanto a ativação estiver aguardando, realize novas consultas respeitando um intervalo adequado.
4. Quando o código for recebido, leia o campo `code` ou a lista `codes`.
5. Após utilizar o número, solicite a finalização da ativação.

Um intervalo de 3 a 5 segundos entre consultas costuma ser suficiente para esse fluxo.

### Cancelamento

Se nenhum código for recebido, consulte as condições de cancelamento e solicite o encerramento pelo endpoint `/number/cancel`.

O cancelamento está sujeito às regras da ativação. Conforme o comportamento documentado, a solicitação só é aceita após os dois minutos iniciais da compra, e o campo `cancel_in_seconds` informa o tempo restante até que ela possa ser solicitada.

Não presuma que uma ativação pode ser cancelada depois que um código tiver sido recebido ou a operação tiver sido encerrada.

## Estados da ativação

O campo `state` informa a situação atual da ativação.

| Estado | Descrição |
|---|---|
| `waiting` | Aguardando o recebimento do código |
| `code_received` | Código SMS recebido |
| `completed` | Ativação finalizada |
| `cancelled` | Ativação cancelada e reembolsada |
| `expired` | Ativação expirada sem recebimento de código |

Utilize esses estados para determinar quando continuar consultando a ativação e quando interromper as requisições.

Os estados `completed`, `cancelled` e `expired` representam situações de encerramento. O comportamento exato de cada operação deve seguir a resposta retornada pela API.

## Ativações e transações

A API disponibiliza endpoints para consultar o histórico de ativações e as movimentações da carteira.

### Listar ativações

O endpoint `/activations` aceita os seguintes parâmetros:

| Parâmetro | Descrição |
|---|---|
| `state` | Filtro opcional pelo estado da ativação, como `active` |
| `limit` | Quantidade de registros por resposta, de 1 a 100; padrão 50 |

**Exemplo:**

```bash
curl "https://kdefy-sms.com/api/v1/activations?state=active&limit=50" \
  -H "X-API-Key: $KDEFY_KEY"
```

### Consultar transações

O endpoint `/transactions` permite consultar o extrato da carteira.

| Parâmetro | Descrição |
|---|---|
| `limit` | Quantidade de registros por resposta, de 1 a 200; padrão 50 |

**Exemplo:**

```bash
curl "https://kdefy-sms.com/api/v1/transactions?limit=50" \
  -H "X-API-Key: $KDEFY_KEY"
```

## Idempotência

A idempotência permite repetir uma requisição de compra sem criar uma nova operação quando a solicitação anterior já tiver sido processada com a mesma chave.

Esse recurso é especialmente importante quando ocorre um timeout ou quando a conexão é interrompida antes que a aplicação receba a resposta.

A chave pode ser enviada pelo header `Idempotency-Key` ou pelo parâmetro `request_id`.

### Exemplo

```bash
curl -X POST https://kdefy-sms.com/api/v1/number/get \
  -H "X-API-Key: $KDEFY_KEY" \
  -H "Idempotency-Key: pedido-2026-10-26-0001" \
  -H "Content-Type: application/json" \
  -d '{
    "country": 73,
    "service": "tg"
  }'
```

### Comportamento

- Repetir a requisição com a mesma chave e os mesmos parâmetros retorna a resposta original, com o header `Idempotent-Replay: true`.
- Reutilizar a chave com parâmetros diferentes retorna `request_id_reutilizada`, com HTTP `422`.
- Repetir a requisição enquanto a primeira ainda estiver em processamento pode retornar `requisicao_em_andamento`, com HTTP `409`.

Em caso de resultado incerto, reutilize a mesma chave. Não gere uma nova chave imediatamente para a mesma compra. Se necessário, consulte `/activations` para verificar as ativações registradas.

## Limites de requisição

A API aplica limites de utilização por conta para controlar o volume de requisições e compras.

| Limite | Valor |
|---|---|
| Requisições por minuto, por conta | 120 |
| Números comprados por minuto, por conta | 50 |
| Quantidade máxima por compra em lote | 50 |
| Tentativas com token inválido | 30 por minuto |

Ao exceder os limites, a API pode retornar HTTP `429`.

Os códigos de erro correspondentes são:

- `limite_de_requisicoes`: limite geral de requisições excedido.
- `limite_de_numeros`: limite de compras de números excedido.

No caso de `limite_de_numeros`, o campo `retry_in_seconds` informa quando uma nova tentativa poderá ser realizada.

Respeite os limites retornados pela API e evite realizar consultas desnecessárias.

## Tratamento de erros

Quando uma operação não pode ser concluída, a API retorna uma resposta de erro com os campos `status`, `error` e `message`.

A tabela abaixo apresenta os códigos documentados e suas condições de ocorrência.

| HTTP | Código | Descrição |
|---|---|---|
| 400 | `pais_invalido` | País ausente ou desconhecido |
| 400 | `servico_invalido` | Serviço ausente ou inválido |
| 400 | `quantidade_invalida` | Quantidade fora do intervalo de 1 a 50 |
| 400 | `preco_invalido` | Valor de `max_price_brl` inválido |
| 400 | `operador_invalido` | Operadora inválida |
| 400 | `request_id_invalido` | Chave de idempotência fora do formato permitido |
| 400 | `id_invalido` | ID da ativação ausente ou inválido |
| 401 | `token_nao_informado` | Chave de API não enviada |
| 401 | `token_invalido` | Chave inválida ou revogada |
| 402 | `saldo_insuficiente` | Saldo insuficiente para a operação |
| 403 | `conta_bloqueada` | Conta bloqueada |
| 403 | `token_suspenso` | Acesso à API suspenso |
| 404 | `sem_numeros_disponiveis` | Não há números disponíveis para o serviço e país |
| 404 | `oferta_indisponivel` | Oferta indisponível no momento da compra |
| 404 | `ativacao_nao_encontrada` | Ativação não encontrada na conta |
| 404 | `rota_nao_encontrada` | Endpoint inexistente |
| 409 | `ativacao_nao_cancelavel` | Ativação não pode ser cancelada |
| 409 | `cancelamento_muito_cedo` | Intervalo mínimo para cancelamento ainda não atingido |
| 409 | `ativacao_nao_finalizavel` | Ativação ainda não pode ser finalizada |
| 409 | `requisicao_em_andamento` | Requisição idempotente ainda em processamento |
| 422 | `preco_abaixo_do_minimo` | Nenhuma oferta atende ao preço máximo informado |
| 422 | `request_id_reutilizada` | Chave de idempotência utilizada com outros parâmetros |
| 429 | `limite_de_requisicoes` | Limite de requisições excedido |
| 429 | `limite_de_numeros` | Limite de compras excedido |
| 500 | `erro_interno` | Erro interno inesperado |
| 502 | `falha_temporaria` | Falha temporária durante o processamento |
| 503 | `provedor_ocupado` | Provedor temporariamente ocupado |

### Recomendações para tratamento de erros

- Verifique o código HTTP e o campo `error`.
- Utilize `message` apenas como informação complementar para exibição ou diagnóstico.
- Em erros temporários, aplique retentativas com espera progressiva (*exponential backoff*).
- Respeite o header `Retry-After` quando retornado.
- Em operações de compra com resultado incerto, utilize a mesma chave de idempotência nas retentativas.

De acordo com o comportamento documentado, falhas `502` durante compras não geram cobrança. Ainda assim, em qualquer resultado incerto, evite criar uma nova compra sem antes verificar a operação anterior.

## Exemplos de integração

Os exemplos a seguir demonstram como autenticar requisições, comprar números e acompanhar ativações utilizando linguagens comuns em aplicações web e serviços backend.

### Python

**Requisitos:** Python 3.9 ou superior e a biblioteca `requests`.

**Instalação:**

```bash
pip install requests
```

Defina a variável de ambiente `KDEFY_KEY` com sua chave de API antes de executar o programa.

```python
import os
import time
import requests

BASE = "https://kdefy-sms.com/api/v1"
API_KEY = os.environ["KDEFY_KEY"]

session = requests.Session()
session.headers.update({
    "X-API-Key": API_KEY,
    "Accept": "application/json",
})


def call(method, path, *, params=None, json=None):
    response = session.request(
        method,
        f"{BASE}{path}",
        params=params,
        json=json,
        timeout=20,
    )

    try:
        data = response.json()
    except ValueError:
        response.raise_for_status()
        raise RuntimeError("A API retornou uma resposta que não é JSON.")

    if not response.ok or data.get("status") != "success":
        error = data.get("error", "erro_desconhecido")
        message = data.get("message", "Não foi possível concluir a operação.")
        raise RuntimeError(f"{error}: {message}")

    return data


order = call(
    "POST",
    "/number/get",
    json={
        "country": 73,
        "service": "tg",
        "max_price_brl": 3.00,
    },
)

print("Número:", order["phone"])

activation_id = order["id"]

for _ in range(120):
    info = call(
        "GET",
        "/number/status",
        params={"id": activation_id},
    )

    if info["state"] == "code_received":
        print("Código:", info["code"])
        call(
            "POST",
            "/number/finish",
            params={"id": activation_id},
        )
        break

    if info["state"] in {"completed", "cancelled", "expired"}:
        print("Ativação encerrada:", info["state"])
        break

    time.sleep(5)
else:
    print("Tempo de espera esgotado. Verifique a ativação antes de agir.")
```

O exemplo pressupõe que os endpoints aceitam os métodos e os parâmetros utilizados. Ajuste o método de finalização conforme o contrato efetivo da API.

### Node.js

**Requisitos:** Node.js 18 ou superior, com suporte nativo a `fetch`.

Defina a variável de ambiente `KDEFY_KEY` antes de executar o programa.

```javascript
const BASE = "https://kdefy-sms.com/api/v1";
const API_KEY = process.env.KDEFY_KEY;

if (!API_KEY) {
  throw new Error("Defina a variável de ambiente KDEFY_KEY.");
}

async function call(method, path, { params = {}, body } = {}) {
  const url = new URL(`${BASE}${path}`);

  for (const [key, value] of Object.entries(params)) {
    url.searchParams.set(key, String(value));
  }

  const headers = {
    "X-API-Key": API_KEY,
    "Accept": "application/json",
  };

  const options = { method, headers };

  if (body !== undefined) {
    headers["Content-Type"] = "application/json";
    options.body = JSON.stringify(body);
  }

  const response = await fetch(url, options);
  const data = await response.json();

  if (!response.ok || data.status !== "success") {
    throw new Error(
      `${data.error ?? "erro_desconhecido"}: ` +
      `${data.message ?? "Não foi possível concluir a operação."}`
    );
  }

  return data;
}

const order = await call("POST", "/number/get", {
  body: {
    country: 73,
    service: "tg",
    max_price_brl: 3.00,
  },
});

console.log("Número:", order.phone);

for (let i = 0; i < 120; i++) {
  const info = await call("GET", "/number/status", {
    params: { id: order.id },
  });

  if (info.state === "code_received") {
    console.log("Código:", info.code);

    await call("POST", "/number/finish", {
      params: { id: order.id },
    });

    break;
  }

  if (["completed", "cancelled", "expired"].includes(info.state)) {
    console.log("Ativação encerrada:", info.state);
    break;
  }

  await new Promise(resolve => setTimeout(resolve, 5000));
}
```

Os exemplos são pontos de partida para integração. Em aplicações de produção, implemente também o tratamento de falhas de rede, timeouts, respostas inesperadas e retentativas seguras.

### Compra em lote com cURL

O exemplo abaixo solicita dez números em uma única requisição.

```bash
curl -X POST https://kdefy-sms.com/api/v1/number/get \
  -H "X-API-Key: $KDEFY_KEY" \
  -H "Idempotency-Key: lote-0001" \
  -H "Content-Type: application/json" \
  -d '{
    "country": 73,
    "service": "tg",
    "quantity": 10
  }'
```

Consulte os campos `requested`, `purchased`, `failed` e `numbers` para identificar o resultado da operação em lote.

## Recursos adicionais

A plataforma disponibiliza outros formatos de documentação para consulta e integração com ferramentas externas.

| Recurso | Descrição |
|---|---|
| [Documentação interativa](https://kdefy-sms.com/api/v1/docs) | Referência navegável dos endpoints |
| [`/docs.json`](https://kdefy-sms.com/api/v1/docs.json) | Documentação em formato JSON |
| [`/docs.txt`](https://kdefy-sms.com/api/v1/docs.txt) | Documentação em texto |
| [OpenAPI 3](https://kdefy-sms.com/api/v1/openapi.json) | Especificação da API |
| [`/ai.txt`](https://kdefy-sms.com/api/v1/ai.txt) | Resumo da API para uso em ferramentas de IA |

## Suporte

Em caso de dúvidas sobre autenticação, integração, compras ou utilização dos endpoints, utilize um dos canais oficiais.

- **Site:** <https://kdefy-sms.com>
- **Canal oficial no Telegram:** <https://t.me/kdefysms>
- **Bot oficial:** <https://t.me/SmsKdefyBot>
- **Suporte no Telegram:** <https://t.me/SuporteKdefy>
- **Canal WhatsApp:** <https://whatsapp.com/channel/0029VbDbqD22ER6aphR0Vm0R>
- **E-mail:** <support@kdefy-sms.com>

Para facilitar o atendimento, informe o endpoint utilizado, o código HTTP recebido e o campo `error` da resposta, quando disponível. Nunca envie sua chave de API ou outros dados de autenticação.
