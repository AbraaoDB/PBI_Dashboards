# Modelagem Dimensional — Dash_Sales (Dash_Ecommerce_Sales)

Data: 2026-10-03 · Modelo: `Dash_Sales.SemanticModel` · `compatibilityLevel: 1606` · cultura `pt-BR`

Complementa `docs/power-query/auditoria-2026-10-03.md`, que trata da camada de ETL.

---

## 1. Diagrama conceitual

```
                        ┌──────────────────┐
                        │   dCalendario    │  dia · 1.827 linhas
                        │  Data (isKey)    │  dataCategory: Time
                        └────────┬─────────┘
                                 │ 1
                                 │   OrderDate
                                 ▼ N
┌──────────────────┐  1      N ┌──────────────────┐
│    dCliente      ├───────────┤     fOrders      │  pedido · 138.116 linhas
│ ClienteSK(isKey) │ ClienteSK │  PedidoSK        │  métricas NÃO aditivas
│  25.000 linhas   │           │  + 13 atributos  │  (dimensão degenerada)
└──────────────────┘           └────────┬─────────┘
                                        │ 1
                                        │   PedidoSK
                                        ▼ N
┌──────────────────┐  1      N ┌──────────────────┐
│    dProduto      ├───────────┤   fOrderItems    │  item de pedido · 397.569 linhas
│ ProdutoSK(isKey) │ ProdutoSK │  PedidoSK        │  ÚNICA fonte de receita e lucro
│   1.175 linhas   │           │  ProdutoSK       │
└──────────────────┘           └──────────────────┘
```

Fora do modelo (não carregam): `caminhoDados`, `fxGeraCalendario`, `srcEcommerceSales`,
`srcOrderItems`, `srcCustomerMaster`, `srcProductCatalog`, `statsCargaCsv`,
`qaValidacaoCarga`.

---

## 2. Fatos e dimensões, com granularidade declarada

| Tabela | Tipo | Uma linha representa | Linhas | Modo |
|---|---|---|---:|---|
| `fOrderItems` | fato transacional | **um item de um pedido** | 397.569 | Import |
| `fOrders` | fato de cabeçalho + dimensão degenerada | **um pedido** | 138.116 | Import |
| `dCliente` | dimensão | um cliente | 25.000 | Import |
| `dProduto` | dimensão | um produto | 1.175 | Import |
| `dCalendario` | dimensão (tabela de datas) | um dia | 1.827 | Import |

### Separação de métricas por grão

Esta é a decisão central do modelo. As duas fontes traziam as **mesmas** métricas em grãos
diferentes, o que dava dois resultados para "qual foi a receita".

| Métrica | Onde vive | Por quê |
|---|---|---|
| `Quantity`, `GrossSales`, `DiscountAmount`, `TaxAmount`, `ShippingCost`, `NetSales`, `ProductCost`, `Profit` | **só `fOrderItems`** | aditivas; o grão do item é o mais fino e o único completo (ver §6) |
| `DeliveryDays`, `EstimatedDeliveryDays`, `CustomerRating` | só `fOrders` | existem por pedido, não por item; **não aditivas** — `summarizeBy: average` |
| `LoyaltyPointsEarned`, `LoyaltyPointsRedeemed` | só `fOrders` | aditivas, mas o grão real é o pedido |
| `CustomerLifetimeValue`, `CustomerOrderCount` | só `fOrders` | atributos do cliente repetidos a cada pedido — **somar dá número sem sentido** |

`profit_margin_percentage` foi descartada: é razão, e razão não se agrega. Calcule
`DIVIDE(SUM(Profit), SUM(NetSales))` em DAX sobre `fOrderItems`.

---

## 3. Relacionamentos

Todos **muitos-para-um, filtro de direção única** (dimensão → fato). Zero bidirecional,
zero muitos-para-muitos, zero inativo.

| De (N) | Para (1) | Chave | Tipo |
|---|---|---|---|
| `fOrderItems` | `fOrders` | `PedidoSK` (Int64) | fato → cabeçalho |
| `fOrderItems` | `dProduto` | `ProdutoSK` (Int64) | fato → dimensão |
| `fOrders` | `dCliente` | `ClienteSK` (Int64) | fato → dimensão |
| `fOrders` | `dCalendario` | `OrderDate` → `Data` | fato → tabela de datas |

### Integridade validada contra os dados de origem

| Verificação | Resultado |
|---|---|
| `dCliente[ClienteSK]` único | 25.000 de 25.000 |
| `dProduto[ProdutoSK]` único | 1.175 de 1.175 |
| `fOrders[PedidoSK]` único | 138.116 de 138.116 |
| `fOrders[ClienteSK]` nulo | 0 de 138.116 |
| `fOrderItems[PedidoSK]` nulo | 0 de 397.569 |
| `fOrderItems[ProdutoSK]` nulo | 0 de 397.569 |
| Datas de pedido fora de `dCalendario` | 0 (range derivado dos próprios dados) |

**Nenhuma relação vai gerar linha em branco.**

### Como o filtro propaga

`dCliente` e `dCalendario` filtram `fOrderItems` **através de `fOrders`**, em dois saltos:

```
dCalendario → fOrders → fOrderItems
dCliente    → fOrders → fOrderItems
```

Funciona porque os dois saltos são muitos-para-um de direção única. "Receita por mês" e
"receita por região" filtram `fOrders`, que filtra os itens. É a exceção declarada ao
estrela puro — justificativa em §4.

---

## 4. Decisões de modelagem registradas

### 4.1 Por que `fOrders` não virou uma dimensão separada de atributos

`order-items.csv` não traz data nem cliente. Para `dCalendario` e `dCliente` filtrarem os
itens direto (estrela puro de dois fatos conformados), os atributos descritivos do pedido
teriam de virar uma dimensão relacionada aos dois fatos. Medi a viabilidade de uma
*junk dimension* com os 13 atributos:

| Atributo | Distintos | | Atributo | Distintos |
|---|---:|---|---|---:|
| `warehouse` | 19 | | `payment_status` | 4 |
| `marketing_channel` | 10 | | `shipping_method` | 4 |
| `return_reason` | 9 | | `delivery_status` | 4 |
| `payment_method` | 7 | | `review_sentiment` | 4 |
| `currency` | 7 | | `customer_type` | 3 |
| `order_status` | 4 | | `return_status` | 2 |
| `sales_channel` | 4 | | | |

**102.392 combinações distintas em 138.116 pedidos — 74,1%.** Uma junk dimension com 74%
das linhas do fato não é dimensão, é cópia: não economiza memória e adiciona um salto.

**Decisão:** os atributos ficam em `fOrders` como **dimensão degenerada**, e `fOrderItems`
se relaciona a `fOrders`. Exceção consciente ao estrela puro, com dois ganhos: nenhuma
tabela redundante, e nenhum merge de 397 mil linhas por refresh para denormalizar data e
cliente no grão do item.

**Quando reavaliar:** se `fOrderItems` crescer uma ordem de magnitude, ou se surgir um
terceiro fato que precise dos mesmos atributos de pedido. Aí denormalizar `OrderDate` e
`ClienteSK` em `fOrderItems` (um merge) passa a valer mais que o salto extra.

### 4.2 Chaves surrogadas inteiras

As chaves nativas são texto: `CUST-000001`, `PROD-000001`, `ORD-301242`. Só em
`fOrderItems` isso somava **8.348.949 caracteres** em colunas de chave. Substituídas por
`ClienteSK`, `ProdutoSK` e `PedidoSK` em `Int64`, geradas com `Table.AddIndexColumn` na
camada de staging e distribuídas por join.

- As chaves de texto foram **removidas das tabelas do modelo**, mas continuam no staging —
  é por elas que o join acontece.
- `OrderId` legível permanece visível em `fOrders` como dimensão degenerada, para contagem
  distinta e rastreio de um pedido específico.
- `CustomerId` e `ProductId` legíveis permanecem nas dimensões.
- SKs são `isHidden` + `isAvailableInMdx: false`: não servem para arrastar em visual e não
  precisam ficar disponíveis para cliente MDX.
- `isKey` nas SKs de `dCliente` e `dProduto` (regra BPA `MARK_PRIMARY_KEYS`).

**Atenção à estabilidade:** o índice é posicional, derivado da ordem do CSV. Se a ordem das
linhas da fonte mudar, as SKs mudam. Não há problema enquanto a carga é full refresh e
nada externo persiste essas chaves — mas **se um dia houver Incremental Refresh ou
exportação que guarde a SK, troque o índice por uma chave estável derivada do id natural.**

### 4.3 Modo de armazenamento

**Import** nas cinco tabelas. Os dados são CSV em disco: não há OneLake para Direct Lake e
DirectQuery sobre arquivo local não se aplica. Volume total (~563 mil linhas) é confortável
para Import.

### 4.4 Tabela de datas

- `dataCategory: Time` + `isKey` em `Data` — a marcação exigida, confirmada na abertura do
  projeto.
- `__PBI_TimeIntelligenceEnabled = 0`, e as tabelas `LocalDateTable_*` /
  `DateTableTemplate_*` foram removidas.
- Gerada por `fxGeraCalendario` no Power Query, não por `CALENDAR` em DAX.
- Range derivado de `List.Min`/`List.Max` de `order_date`, arredondado para anos inteiros:
  hoje 2021-01-01 a 2025-12-31, 1.827 dias contínuos sem lacuna.
- Uma única tabela calendário. Não há data secundária relacionada: a data de entrega existe
  como `DeliveryDays` (duração), não como data. Se um dia virar data, use relacionamento
  inativo + `USERELATIONSHIP` em vez de uma segunda `dCalendario`.

### 4.5 Nulos estruturais preservados

`DeliveryDays`, `EstimatedDeliveryDays` e `CustomerRating` são nulos em 24.557 linhas —
exatamente os pedidos `Returned`, `Cancelled` e `Pending`, e zero dos `Completed`. O nulo é
"não se aplica" e **não** foi substituído por zero. As três colunas estão com
`summarizeBy: average` e `///` avisando para não dividir pelo total de linhas.

### 4.6 Tabela de estatísticas fora do modelo

`statsCargaCsv` (ex-`dataset_statistics`) é snapshot congelado de KPIs em texto e divergia
das medidas. Virou expressão não carregada, disponível no editor para conferência.

---

## 5. Dicionário de dados

### fOrderItems — grão: item de pedido (397.569)

| Coluna | Tipo | summarizeBy | Visível | Observação |
|---|---|---|---|---|
| `PedidoSK` | int64 | none | não | FK → `fOrders` |
| `ProdutoSK` | int64 | none | não | FK → `dProduto` |
| `Quantity` | int64 | sum | sim | |
| `UnitPrice` | decimal | sum | sim | |
| `DiscountPercentage` | double | average | sim | fração (0,34 = 34%) |
| `DiscountAmount` | decimal | sum | sim | |
| `GrossSales` | decimal | sum | sim | |
| `TaxAmount` | decimal | sum | sim | |
| `ShippingCost` | decimal | sum | sim | |
| `NetSales` | decimal | sum | sim | **fonte oficial de receita** |
| `ProductCost` | decimal | sum | sim | |
| `Profit` | decimal | sum | sim | **fonte oficial de lucro** |

Identidade conferida no cabeçalho: `gross − desconto + imposto + frete = net`.

### fOrders — grão: pedido (138.116)

| Coluna | Tipo | summarizeBy | Visível | Observação |
|---|---|---|---|---|
| `PedidoSK` | int64 | none | não | chave, lado 1 de `fOrderItems` |
| `ClienteSK` | int64 | none | não | FK → `dCliente` |
| `OrderId` | string | none | sim | dimensão degenerada |
| `OrderDate` | dateTime | none | sim | FK → `dCalendario` |
| `OrderTime` | dateTime | none | sim | hora separada da data |
| `OrderStatus`, `SalesChannel`, `CustomerType`, `PaymentMethod`, `PaymentStatus`, `Currency`, `ShippingMethod`, `Warehouse`, `DeliveryStatus`, `ReturnStatus`, `ReturnReason`, `ReviewSentiment`, `MarketingChannel` | string | none | sim | 13 atributos degenerados |
| `DeliveryDays` | int64 | average | sim | nulo fora de `Completed` |
| `EstimatedDeliveryDays` | int64 | average | sim | nulo fora de `Completed` |
| `CustomerRating` | double | average | sim | nulo fora de `Completed` |
| `LoyaltyPointsEarned` | int64 | sum | sim | |
| `LoyaltyPointsRedeemed` | int64 | sum | sim | |
| `CustomerLifetimeValue` | decimal | average | sim | não somar |
| `IsRepeatCustomer` | boolean | none | sim | |
| `CustomerOrderCount` | int64 | average | sim | não somar |

23 colunas visíveis — abaixo do limite de ~30 que indicaria desnormalização excessiva.

### dCliente — grão: cliente (25.000)

| Coluna | Tipo | summarizeBy | Visível |
|---|---|---|---|
| `ClienteSK` | int64 | none | não (`isKey`) |
| `CustomerId` | string | none | sim |
| `CustomerName` | string | none | sim |
| `CustomerAge` | int64 | none | sim |
| `Gender`, `CustomerSegment`, `CustomerCity`, `CustomerState`, `CustomerCountry`, `Region` | string | none | sim |
| `CustomerPostalCode` | string | none | sim |
| `CustomerAcquisitionCost` | decimal | sum | sim |

`CustomerPostalCode` é texto de propósito: identificador, não número.

### dProduto — grão: produto (1.175)

| Coluna | Tipo | summarizeBy | Visível |
|---|---|---|---|
| `ProdutoSK` | int64 | none | não (`isKey`) |
| `ProductId` | string | none | sim |
| `ProductName`, `ProductCategory`, `ProductSubcategory`, `Brand`, `Supplier` | string | none | sim |
| `UnitPrice` | decimal | sum | sim |
| `ProductCost` | decimal | sum | sim |
| `ProductRating` | double | average | sim |

`UnitPrice` aqui é preço de tabela; o preço praticado está em `fOrderItems[UnitPrice]`.

### dCalendario — grão: dia (1.827)

| Coluna | Tipo | sortByColumn | Visível |
|---|---|---|---|
| `Data` | dateTime | — | sim (`isKey`) |
| `Ano` | int64 | — | sim |
| `TrimestreNum` | int64 | — | não |
| `Trimestre` | string | `TrimestreNum` | sim |
| `MesNum` | int64 | — | não |
| `MesNome` | string | `MesNum` | sim |
| `MesAbrev` | string | `MesNum` | sim |
| `AnoMesNum` | int64 | — | não |
| `AnoMes` | string | `AnoMesNum` | sim |
| `DiaMes` | int64 | — | sim |
| `SemanaAno` | int64 | — | sim |
| `DiaSemanaNum` | int64 | — | não |
| `DiaSemanaNome` | string | `DiaSemanaNum` | sim |
| `FimDeSemana` | boolean | — | sim |

Toda coluna de texto temporal tem ordenação numérica. As auxiliares `*Num` ficam ocultas
mas **mantêm `isAvailableInMdx` padrão**, porque são alvo de `sortByColumn`.

---

## 6. Alerta sobre a fonte: o cabeçalho subconta 7,9%

Registrado em detalhe na auditoria de ETL (§"Achado crítico novo"), repetido aqui porque
condiciona a leitura de qualquer número do modelo:

O `net_sales` do cabeçalho agrega **no máximo 5 itens por pedido** e ignora o excedente —
0% de divergência em pedido de 1 item, 100% em pedido com 6 ou mais, e em 11.226 de 11.226
casos divergentes o cabeçalho equivale à soma de um prefixo dos itens. O detalhe é que
está completo.

| Fonte | Receita |
|---|---:|
| `SUM(fOrderItems[NetSales])` — **o modelo** | **192.398.877,29** |
| `net_sales` do cabeçalho (coluna removida) | 177.134.263,74 |
| `Total Revenue` do CSV de estatísticas | 177.134.263,74 |

`SUM(fOrderItems[NetSales])` **não deve** bater com o `Total Revenue` do CSV. Se bater,
algo voltou a ler o cabeçalho.

---

## 7. Próximo passo

O modelo está pronto para medidas. Nada de DAX foi escrito ainda — a ordem é
partições → colunas e tipos → relacionamentos → **medidas**, e as três primeiras etapas
estão fechadas e validadas.

Sugestão de primeira entrega, tudo sobre `fOrderItems` com `dCalendario` já marcada:
receita, lucro, margem, ticket médio, pedidos distintos (`DISTINCTCOUNT` em
`fOrders[OrderId]`), e as de prazo e satisfação com `AVERAGE` sobre `fOrders`.

Crie as medidas numa tabela dedicada (`Medidas`), não penduradas nas fatos.
