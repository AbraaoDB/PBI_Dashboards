# Medidas DAX — Dash_Sales

Data: 2026-10-03 · Arquivo: `Dash_Sales.SemanticModel/definition/tables/Medidas.tmdl`
Origem dos requisitos: `docs/levantamento-requisitos/`

**63 medidas** numa tabela calculada dedicada `Medidas`, distribuídas em 12 `displayFolder`.
Nenhuma medida pendurada em tabela fato ou dimensão.

## Convenções aplicadas

- **Medidas base antes das derivadas.** `[Receita Bruta]` é a única que soma `NetSales`; as
  outras 12 que dependem de receita a referenciam. Nenhuma lógica duplicada.
- **`DIVIDE()`** em toda razão, nunca `/` — nenhuma delas está dentro de iterador.
- **`KEEPFILTERS`** como forma de predicado em `CALCULATE`, conforme a skill.
- **`formatString` sem símbolo de moeda:** `#,0.00`, não `R$ #,0.00`. Os valores da fonte
  estão em sete moedas não convertidas (ressalva §6.1 do levantamento); rotular como real
  seria afirmar algo falso. Percentuais em `0.00%` — duas casas, para conferir contra os
  critérios de aceite.
- **Descrição `///`** em todas as 63, com a regra de negócio em uma frase.
- **Variáveis `__`** onde há expressão reaproveitada (`[Variacao de Preco %]`,
  `[Entregas Atrasadas]`, `[Receita Media Diaria]`).
- **Nomes sem acento.** Decisão deliberada: evita problema de encoding na edição direta de
  TMDL e em referência cruzada entre medidas. Os rótulos acentuados ficam para o visual.

---

## Valores conferidos

Cada medida foi reproduzida sobre os CSVs de origem. **A coluna "Esperado" é o valor que a
medida deve retornar no período completo, sem nenhum filtro.** Se o Desktop devolver outro
número, a medida está errada — ou a carga está.

### 01. Vendas

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Receita Bruta` | `SUM ( fOrderItems[NetSales] )` | 192.398.877,29 |
| `Receita Realizada` | `[Receita Bruta]` com `OrderStatus = "Completed"` | 156.796.214,63 |
| `Receita Perdida` | `[Receita Bruta]` com status `Cancelled` ou `Returned` | 26.457.549,67 |
| `Receita em Aberto` | `[Receita Bruta]` com `OrderStatus = "Pending"` | 9.145.112,99 |
| `% Receita Perdida` | `DIVIDE ( [Receita Perdida], [Receita Bruta] )` | 13,75% |
| `Venda Bruta antes de Desconto` | `SUM ( fOrderItems[GrossSales] )` | 206.343.698,09 |
| `Desconto Concedido` | `SUM ( fOrderItems[DiscountAmount] )` | 35.649.166,70 |
| `Desconto Medio %` | `AVERAGE ( fOrderItems[DiscountPercentage] )` | 14,71% |
| `Imposto` | `SUM ( fOrderItems[TaxAmount] )` | 18.361.184,20 |
| `Frete` | `SUM ( fOrderItems[ShippingCost] )` | 3.343.161,70 |

Soma de controle: `Realizada + Perdida + Em Aberto = 192.398.877,29` = `Receita Bruta`. ✔

### 02. Volume

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Pedidos` | `DISTINCTCOUNT ( fOrders[OrderId] )` | 138.116 |
| `Itens Vendidos` | `COUNTROWS ( fOrderItems )` | 397.569 |
| `Unidades` | `SUM ( fOrderItems[Quantity] )` | 842.366 |
| `Itens por Pedido` | `DIVIDE ( [Itens Vendidos], [Pedidos] )` | 2,88 |
| `Unidades por Pedido` | `DIVIDE ( [Unidades], [Pedidos] )` | 6,10 |

### 03. Margem

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Custo do Produto` | `SUM ( fOrderItems[ProductCost] )` | 106.357.677,04 |
| `Lucro` | `SUM ( fOrderItems[Profit] )` | 82.698.038,55 |
| `Margem %` | `DIVIDE ( [Lucro], [Receita Bruta] )` | 42,98% |
| `Lucro sem Imposto` | `[Lucro] - [Imposto]` | 64.336.854,35 |
| `Margem sem Imposto %` | `DIVIDE ( [Lucro] - [Imposto], [Receita Bruta] - [Imposto] )` | 36,97% |
| `Lucro Realizado` | `[Lucro]` com `OrderStatus = "Completed"` | 65.164.085,25 |

As duas margens existem por decisão registrada (pendência Q3): a de 42,98% é fiel à fonte e
conta imposto como lucro; a de 36,97% é a leitura econômica.

### 04. Ticket Médio

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Ticket Medio` | `DIVIDE ( [Receita Bruta], [Pedidos] )` | 1.393,02 |
| `Ticket Medio Realizado` | `DIVIDE ( [Receita Realizada], [Pedidos Concluidos] )` | 1.380,75 |
| `Receita por Item` | `DIVIDE ( [Receita Bruta], [Itens Vendidos] )` | 483,94 |
| `Preco Medio Praticado` | `AVERAGE ( fOrderItems[UnitPrice] )` | 245,19 |
| `Preco Medio de Tabela` | `AVERAGE ( dProduto[UnitPrice] )` | 245,52 |
| `Variacao de Preco %` | `DIVIDE ( praticado - tabela, tabela )` | −0,14% |

O desvio de preço de −0,14% é efetivamente zero: na fonte, o preço praticado não guarda
relação com o de catálogo. A medida está certa; a conclusão de negócio não existe.

### 05. Status do Pedido

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Pedidos Concluidos` | `[Pedidos]` com `Completed` | 113.559 |
| `Pedidos Devolvidos` | `[Pedidos]` com `Returned` | 9.462 |
| `Pedidos Cancelados` | `[Pedidos]` com `Cancelled` | 8.398 |
| `Pedidos Pendentes` | `[Pedidos]` com `Pending` | 6.697 |
| `Taxa de Conclusao %` | `DIVIDE ( [Pedidos Concluidos], [Pedidos] )` | 82,22% |
| `Taxa de Devolucao %` | `DIVIDE ( [Pedidos Devolvidos], [Pedidos] )` | 6,85% |
| `Taxa de Cancelamento %` | `DIVIDE ( [Pedidos Cancelados], [Pedidos] )` | 6,08% |

As três taxas batem com o `dataset-statistics.csv` (6,85% e 6,08%) — os indicadores de
contagem da fonte estão corretos; só os monetários não.

### 06. Tempo

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Receita Acumulada no Ano` | `TOTALYTD ( [Receita Bruta], dCalendario[Data] )` | varia pelo contexto |
| `Receita Realizada Acumulada no Ano` | `TOTALYTD ( [Receita Realizada], dCalendario[Data] )` | varia pelo contexto |
| `Receita Media Diaria` | `DIVIDE ( [Receita Bruta], DISTINCTCOUNT ( fOrders[OrderDate] ) )` | 105.366,31 |

Sem filtro, `Receita Media Diaria` usa os 1.826 dias com pedido. A medida respeita o período
filtrado — não fixei o denominador no histórico inteiro, que daria um número constante e
enganoso ao fatiar.

**Não há medida de crescimento YoY**, e isso é deliberado: a receita é plana nos cinco anos
(variação de 1,6% entre o maior e o menor). A medida funcionaria e exibiria ~0%, o que seria
lido como erro. Diferida com esse motivo no levantamento, §4 bloco D.

### 07. Operação e Entrega

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Prazo Medio de Entrega` | `AVERAGE ( fOrders[DeliveryDays] )` | 4,6 |
| `Prazo Medio Estimado` | `AVERAGE ( fOrders[EstimatedDeliveryDays] )` | 4,3 |
| `Desvio de Prazo` | `[Prazo Medio de Entrega] - [Prazo Medio Estimado]` | +0,3 |
| `Entregas Atrasadas` | `COUNTROWS` de `FILTER` com `DeliveryDays > Estimated` | 16.886 |
| `Entregas no Prazo` | `[Pedidos Concluidos] - [Entregas Atrasadas]` | 96.673 |
| `Aderencia ao Prazo %` | `DIVIDE ( [Entregas no Prazo], [Pedidos Concluidos] )` | 85,13% |
| `Frete Medio por Pedido` | `DIVIDE ( [Frete], [Pedidos] )` | 24,21 |

**`Entregas Atrasadas` é a única que precisa de `FILTER`.** Comparar duas colunas
(`DeliveryDays > EstimatedDeliveryDays`) não é um predicado válido em `CALCULATE` — exige
iteração. O `NOT ISBLANK` é obrigatório: as duas colunas são nulas fora de pedido concluído.

`Aderencia ao Prazo %` usa a comparação das duas colunas, **não** `DeliveryStatus`. A coluna
`DeliveryStatus` mistura `Early` com `On Time` e marca não-entrega como `Cancelled`
(24.557 linhas), que não é o mesmo que `OrderStatus = "Cancelled"` (8.398). Regra R8/R13 do
levantamento.

### 08. Qualidade

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Nota Media` | `AVERAGE ( fOrders[CustomerRating] )` | 3,68 |
| `Avaliacoes Negativas` | `[Pedidos]` com `ReviewSentiment = "Negative"` | 792 |
| `% Avaliacoes Negativas` | `DIVIDE ( [Avaliacoes Negativas], [Pedidos Concluidos] )` | 0,70% |
| `Nota Media por Produto` | `AVERAGE` com `CROSSFILTER ( …, BOTH )` | 3,68 |

### 09. Cliente

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Clientes Ativos` | `DISTINCTCOUNT ( fOrders[ClienteSK] )` | 24.911 |
| `Clientes Cadastrados` | `COUNTROWS ( dCliente )` | 25.000 |
| `Receita por Cliente` | `DIVIDE ( [Receita Bruta], [Clientes Ativos] )` | 7.723,45 |
| `Pedidos por Cliente` | `DIVIDE ( [Pedidos], [Clientes Ativos] )` | 5,54 |
| `CLV Medio` | `AVERAGE ( fOrders[CustomerLifetimeValue] )` | 8.971,64 |

89 clientes cadastrados nunca compraram — daí os dois contadores diferentes.
`CLV Medio` usa `AVERAGE` porque o valor se repete em cada pedido do cliente; somar daria
número sem significado (regra R10).

### 10. Marketing e Aquisição

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Custo de Aquisicao` | `SUM ( dCliente[CustomerAcquisitionCost] )` | 1.053.993,76 |
| `CAC Medio` | `DIVIDE ( [Custo de Aquisicao], [Clientes Cadastrados] )` | 42,16 |
| `Razao CLV sobre CAC` | `DIVIDE ( [CLV Medio], [CAC Medio] )` | 212,8 |

⚠️ **`Razao CLV sobre CAC` = 212,8× não sustenta conclusão de negócio.** Em e-commerce real,
3× a 5× é saudável. Na fonte, CLV e CAC foram gerados independentes um do outro. A medida
está correta em relação ao dado; o dado é que não representa a realidade. Ressalva §6.5 do
levantamento.

### 11. Fidelidade

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Pontos Ganhos` | `SUM ( fOrders[LoyaltyPointsEarned] )` | 14.362.828 |
| `Pontos Resgatados` | `SUM ( fOrders[LoyaltyPointsRedeemed] )` | 7.172.934 |
| `Pontos em Aberto` | `[Pontos Ganhos] - [Pontos Resgatados]` | 7.189.894 |
| `Taxa de Resgate %` | `DIVIDE ( [Pontos Resgatados], [Pontos Ganhos] )` | 49,94% |

### 12. Pagamento

| Medida | DAX (resumo) | Esperado |
|---|---|---:|
| `Pedidos com Pagamento Falho` | `[Pedidos]` com `PaymentStatus = "Failed"` | 10.429 |
| `Taxa de Falha de Pagamento %` | `DIVIDE ( [Pedidos com Pagamento Falho], [Pedidos] )` | 7,55% |
| `Pedidos Pagos` | `[Pedidos]` com `PaymentStatus = "Paid"` | 104.250 |

Os 10.429 com falha são os 8.398 cancelados mais 2.031 pendentes (regra R4).

---

## O ponto de atenção do modelo: filtro de produto não sobe até o pedido

O relacionamento `fOrderItems → fOrders` é **muitos-para-um**. Filtro propaga do lado "um"
para o lado "muitos", nunca o contrário. Consequência prática:

| Direção | Funciona? |
|---|---|
| `dProduto` → `fOrderItems` | ✔ direto |
| `fOrders` (status, canal, CD…) → `fOrderItems` | ✔ o pedido é o lado "um" |
| `dCliente` / `dCalendario` → `fOrderItems` | ✔ via `fOrders`, dois saltos |
| **`dProduto` → `fOrders`** | ✘ **não propaga** |

Então **qualquer medida de `fOrders` fatiada por produto ignora o filtro em silêncio** — e
é exatamente o que a pergunta P17 pede ("a satisfação varia por categoria?").

Solução adotada, sem tornar o relacionamento bidirecional no modelo:

```dax
Nota Media por Produto =
CALCULATE (
    AVERAGE ( fOrders[CustomerRating] ),
    CROSSFILTER ( fOrderItems[PedidoSK], fOrders[PedidoSK], BOTH )
)
```

`CROSSFILTER` liga a direção **só durante essa medida**. Marcar o relacionamento como
bidirecional no modelo resolveria também, mas abriria caminho de filtro ambíguo para as
outras 62 medidas — custo alto para resolver um caso.

**Regra de uso:** ao cruzar com `dProduto`, use `[Nota Media por Produto]`. Em qualquer outro
contexto, `[Nota Media]`. As duas devolvem 3,68 sem filtro; divergem ao fatiar por produto.

Se mais medidas de `fOrders` precisarem de corte por produto (pedidos por categoria, taxa de
devolução por marca), o padrão é o mesmo. Não criei preventivamente: a matriz de
rastreabilidade só pede essa.

---

## Checklist de revisão

| Item | Situação |
|---|---|
| Nasce de requisito registrado | ✔ 52 itens da matriz rastreados à medida |
| Reaproveita medidas base | ✔ só `[Receita Bruta]` soma `NetSales`; 12 derivadas a referenciam |
| `DIVIDE` fora de iteradores | ✔ nenhuma razão dentro de iterador |
| `formatString` e `displayFolder` | ✔ nas 63, sem aspas externas |
| Descrição `///` | ✔ nas 63 |
| Colunas conferidas no TMDL | ✔ todas as 63 contra os `.tmdl` das tabelas |
| Valor plausível conferido | ✔ 60 reproduzidas nos CSVs; 2 de `TOTALYTD` dependem de contexto; 1 (`Nota Media por Produto`) confere com `[Nota Media]` sem filtro |
| **Executada no Desktop** | **✘ pendente — ver abaixo** |

## O que falta validar

Reproduzi o resultado de cada medida nos CSVs de origem, mas **não executei o DAX no motor**.
O Desktop está fechado e não há `pbi-cli` nem o MCP de modelagem nesta máquina, então o
último passo do checklist da skill — `EVALUATE { [Medida] }` e `INFO.MEASURES()` sem
`ErrorMessage` — fica para a sua abertura.

O que a conferência contra os CSVs **não** cobre: erro de sintaxe, nome de coluna divergente
e comportamento de `BLANK()`. Esses aparecem na primeira abertura. Ao abrir, vale rodar:

```dax
EVALUATE
SELECTCOLUMNS (
    FILTER ( INFO.MEASURES (), NOT ISBLANK ( [ErrorMessage] ) ),
    "Medida", [Name],
    "Erro", [ErrorMessage]
)
```

Resultado vazio = as 63 compilaram.
