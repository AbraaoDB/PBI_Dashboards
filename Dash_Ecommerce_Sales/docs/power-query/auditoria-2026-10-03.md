# Auditoria Power Query — Dash_Sales (Dash_Ecommerce_Sales)

Data: 2026-10-03 · Fonte: `Dash_Sales.SemanticModel/definition/tables/*.tmdl` (partições M)

## Resumo executivo

- Queries auditadas: **5** (todas `mode: import`, todas `Csv.Document` + `File.Contents`)
- Parâmetros: **0** · Funções `fx`: **0** · Grupos/pastas: **0** · Steps renomeados: **0**
- Query Folding: **não aplicável** (CSV local não folda) — performance depende de redução antecipada
- Total de violações: **14** (🔴 5 / 🟡 6 / 🔵 3)

> **Status de aplicação (2026-10-03):** os **14 itens foram aplicados**. Fora do escopo final, com
> justificativa: grupos de query (item 10, parcial) e `returnErrorValuesAsNull`. A validação do
> grão revelou um defeito novo e mais grave nos dados — o cabeçalho do pedido agrega no máximo
> 5 itens. Dois registros de aplicação no fim do documento.

### Top 3 prioridades
1. 🔴 **`QuoteStyle.None` corrompe ~11.954 linhas (8,7%)** da fato principal — campos financeiros deslocados.
2. 🔴 **Todas as colunas monetárias tipadas `Int64.Type` sobre dados decimais** — receita, lucro, desconto e margem errados ou nulos em todo o modelo.
3. 🔴 **Caminhos absolutos `C:\Users\braob\Downloads\...`** apontando para fora do repositório (os CSVs estão em `./Data`) — refresh quebra em qualquer outra máquina.

---

## 🔴 Críticos

### 1. `QuoteStyle.None` quebra o parsing de campos com vírgula — query `ecommerce_sales_customer_analytics_150k`

**Problema:** a fonte usa `QuoteStyle=QuoteStyle.None` com `Columns=46`, mas o CSV usa aspas para proteger vírgulas internas. Verificado no arquivo:

- 11.954 de 138.116 linhas (**8,7%**) contêm `"`
- essas linhas, divididas por vírgula crua, produzem **47 campos** em vez de 46

Exemplo real (`customer_review`): `"Product is okay, nothing special but works."` — vira dois campos.

**Impacto:** nessas linhas tudo a partir de `customer_review` desloca uma posição: `marketing_channel` recebe texto de review, e `gross_sales`, `discount_amount`, `tax_amount`, `net_sales`, `product_cost`, `profit`, `customer_lifetime_value` ficam com valor da coluna vizinha ou erro/nulo. Receita e lucro do dashboard estão errados e **não há aviso** — `dataAccessOptions / returnErrorValuesAsNull` no `model.tmdl` converte os erros em nulos silenciosamente.

**Correção** — use `QuoteStyle.Csv` (e remova `Columns`, que é redundante e frágil):

```m
// M original
Fonte = Csv.Document(File.Contents("C:\Users\braob\Downloads\ecommerce_sales_customer_analytics_150k.csv"),
                     [Delimiter=",", Columns=46, Encoding=65001, QuoteStyle=QuoteStyle.None]),

// M corrigido
Fonte = Csv.Document(
    File.Contents(caminhoDados & "ecommerce-sales-customer-analytics-150k.csv"),
    [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]
),
```

O mesmo vale para `dataset_statistics` (`"$177,134,263.74"` é um único campo entre aspas, hoje quebrado em três).

---

### 2. Colunas decimais tipadas como `Int64.Type` — todas as 5 queries

**Problema:** a fonte é decimal, o tipo declarado é inteiro. Amostra dos CSVs:

| Query | Coluna | Valor na fonte | Tipo declarado |
|---|---|---|---|
| `order_items` | `unit_price` | `244.4` | `Int64.Type` |
| `order_items` | `discount_percentage` | `0.33960729091410474` | `Int64.Type` |
| `order_items` | `net_sales` / `profit` | `351.88` / `172.12` | `Int64.Type` |
| `ecommerce_..._150k` | `gross_sales` / `profit` | `1350.19` / `345.04` | `Int64.Type` |
| `ecommerce_..._150k` | `profit_margin_percentage` | `36.5` | `Int64.Type` |
| `ecommerce_..._150k` | `customer_rating` | `3.5` | `Int64.Type` |
| `product_catalog` | `unit_price` / `product_rating` | `686.9` / `4.3` | `Int64.Type` |
| `customer_master` | `customer_acquisition_cost` | `13.58` | `Int64.Type` |
| `dataset_statistics` | `Average Rating` | `3.68` | `Int64.Type` |

**Impacto:** agravado pelo `sourceQueryCulture: pt-BR` do modelo — em pt-BR o ponto é separador de milhar, então `244.4` não converte para inteiro de forma previsível: cada valor vira erro (→ nulo, pela opção `returnErrorValuesAsNull`) ou um número de ordem de grandeza errada. `discount_percentage = 0.3396` truncado para inteiro é **0** — o desconto desaparece. Nenhum KPI financeiro do modelo é confiável hoje.

**Correção:** `Currency.Type` (decimal fixo, melhor compressão e sem erro de ponto flutuante) para dinheiro, `Percentage.Type` para taxas e `type number` para notas/ratings. Para `order_items`:

```m
#"Tipos Ajustados" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos", {
    {"order_id", type text},
    {"product_id", type text},
    {"quantity", Int64.Type},
    {"unit_price", Currency.Type},
    {"discount_percentage", Percentage.Type},
    {"discount_amount", Currency.Type},
    {"gross_sales", Currency.Type},
    {"tax_amount", Currency.Type},
    {"shipping_cost", Currency.Type},
    {"net_sales", Currency.Type},
    {"product_cost", Currency.Type},
    {"profit", Currency.Type}
}, "en-US")   // 3º argumento: a cultura DA FONTE, não a do modelo
```

O terceiro argumento de `Table.TransformColumnTypes` (cultura) é obrigatório aqui: os CSVs usam ponto decimal (`en-US`) e o modelo é `pt-BR`. Sem ele a conversão continua dependendo da máquina que faz o refresh — o mesmo arquivo dá resultado diferente em cada ambiente. Replique em todas as 5 queries.

---

### 3. Caminhos absolutos hard-coded e fora do repositório — todas as 5 queries

**Problema:** as 5 partições apontam para `C:\Users\braob\Downloads\<arquivo>.csv`. Os CSVs versionados estão em `Dash_Ecommerce_Sales/Data/`.

**Impacto:** 🔴 duplo — (a) refresh falha em qualquer máquina que não seja esta; (b) o repositório carrega dados que o modelo não lê, então **não há garantia de que o dashboard reflita os CSVs commitados**. Além disso o caminho expõe o nome de usuário.

**Correção:** crie um parâmetro e derive o caminho. Em `definition/expressions.tmdl` (arquivo ainda não existe no projeto):

```tmdl
expression caminhoDados = "C:\Users\braob\OneDrive\Área de Trabalho\PBI_Dashboards\Dash_Ecommerce_Sales\Data\" meta [IsParameterQuery=true, Type="Text", IsParameterQueryRequired=true]
```

E nas partições: `File.Contents(caminhoDados & "order-items.csv")`. Em cada ambiente novo só o parâmetro muda — no Serviço, via "Parâmetros" do dataset.

---

### 4. `dataset_statistics` guarda KPIs pré-calculados como texto — query `dataset_statistics`

**Problema:** tabela de 1 linha com `Total Revenue`, `Total Profit`, `Average Order Value` tipados `type text` (valores `"$177,134,263.74"`).

**Impacto:** valores que não agregam, não respeitam filtro e **divergem** do que as medidas calculam sobre a fato — duas fontes de verdade para receita. É um snapshot congelado: na primeira atualização dos CSVs ele fica desatualizado sem avisar.

**Correção:** remova a tabela do modelo e calcule tudo em DAX sobre a fato. Se o uso é só conferência de carga, mantenha a query com **Enable load desmarcado** (sem `ref table` no `model.tmdl`) ou mova para `docs/`. Se ela precisar existir, limpe os símbolos antes de tipar:

```m
#"Moeda Limpa" = Table.TransformColumns(#"Cabeçalhos Promovidos", {
    {"Total Revenue", each Number.FromText(Text.Remove(_, {"$", ","}), "en-US"), Currency.Type},
    {"Total Profit",  each Number.FromText(Text.Remove(_, {"$", ","}), "en-US"), Currency.Type},
    {"Average Order Value", each Number.FromText(Text.Remove(_, {"$", ","}), "en-US"), Currency.Type}
})
```

---

### 5. Fato duplicada: `ecommerce_..._150k` x `order_items` + dimensões

**Problema:** `ecommerce_sales_customer_analytics_150k` é uma tabela **totalmente desnormalizada** (46 colunas, 138.116 linhas, grão = pedido) que repete:
- os atributos de cliente já presentes em `customer_master` (`customer_name`, `customer_age`, `gender`, `customer_segment`, `customer_city`, `customer_state`, `customer_country`, `region`, `customer_postal_code` — 9 colunas duplicadas);
- as métricas já presentes em `order_items` (`quantity`, `gross_sales`, `discount_amount`, `tax_amount`, `shipping_cost`, `net_sales`, `product_cost`, `profit`), mas **agregadas por pedido** contra o grão de item (397.569 linhas).

E `relationships.tmdl` liga `order_items[order_id] → ecommerce[order_id]`, tratando a fato de pedido como dimensão de `order_items`.

**Impacto:** dois caminhos para calcular receita, com números diferentes (`SUM(order_items[net_sales])` vs `SUM(ecommerce[net_sales])`). Quem monta o visual escolhe sem saber. Atributos de cliente duplicados fazem o mesmo slicer aparecer duas vezes com comportamento de filtro diferente. É o maior risco de manutenção do modelo.

**Correção (decisão de modelagem, não de M):** escolha um grão e siga o esquema estrela:
- `fOrderItems` (grão item) ou `fOrders` (grão pedido) como **única** fato de vendas;
- remover da fato as 9 colunas de cliente (ficam em `dCliente`, via `customer_id`);
- `ecommerce_..._150k` passa a ser **staging** (Enable load desmarcado) e dela se derivam `fOrders` (cabeçalho: datas, status, canal, pagamento, entrega, chaves) e `dCliente`;
- relacionamentos saem de `fOrderItems[order_id] → fOrders[order_id]` e `fOrders[customer_id] → dCliente[customer_id]`.

Se o escopo não permite reestruturar agora, no mínimo documente qual tabela é a oficial de receita e esconda as métricas da outra (`isHidden`). Detalhamento na skill `dimensional-modeling`.

---

## 🟡 Avisos

### 6. Nenhuma coluna removida — 46 colunas carregadas na fato

Nenhuma query tem `Table.SelectColumns` / `Table.RemoveColumns`. `customer_review` (texto livre de alta cardinalidade), `coupon_code`, `campaign_name` e as 9 colunas de cliente duplicadas entram inteiras no modelo. Em Import o custo é direto no tamanho do arquivo e na RAM. **Remova cedo**, no segundo step, antes de tipar:

```m
#"Colunas Selecionadas" = Table.SelectColumns(#"Cabeçalhos Promovidos", {
    "order_id", "order_date", "order_time", "order_status", "sales_channel",
    "customer_id", "payment_method", "payment_status", "currency",
    "shipping_method", "warehouse", "delivery_days", "estimated_delivery_days",
    "delivery_status", "return_status", "return_reason", "customer_rating",
    "review_sentiment", "marketing_channel"
}),
```

### 7. Encoding inconsistente entre as queries

`65001` (UTF-8) em `ecommerce_..._150k`, `customer_master`, `product_catalog`; `1252` em `order_items` e `dataset_statistics`. Os arquivos vêm do mesmo pipeline. Padronize em `65001` — `1252` corrompe acentos silenciosamente em nomes de cidade/cliente (`customer_city`, `supplier`).

### 8. Auto Date/Time ativo

`annotation __PBI_TimeIntelligenceEnabled = 1` em `model.tmdl` + `LocalDateTable_9cefa3d2...` para `order_date`. O Power BI cria uma tabela de datas oculta por coluna de data, inflando o modelo e sem permitir ano fiscal, feriado ou flag de semana. Desative em Opções → Carregamento de Dados e crie uma `dCalendario` explícita (função `fxGeraCalendario`), marcada como tabela de datas. O range está em `dataset_statistics`: 2021-01-01 a 2025-12-31.

### 9. Sem camada Fonte → Staging → Modelo

As 5 queries vão de `File.Contents` ao modelo em 3 steps, sem staging, e nenhuma tem Enable load desmarcado. Com a reestruturação do item 5 isso passa a ser necessário: `srcEcommerceSales` (staging, não carrega) → `fOrders` + `dCliente` (carregam).

### 10. Nomenclatura fora da convenção

Nenhuma query tem prefixo de papel, nenhuma coluna está em PascalCase, nenhum grupo existe. Mapa sugerido:

| Hoje | Sugerido | Grupo |
|---|---|---|
| `ecommerce_sales_customer_analytics_150k` | `srcEcommerceSales` → `fOrders` | Staging / Fatos |
| `order_items` | `fOrderItems` | Fatos |
| `customer_master` | `dCliente` | Dimensões |
| `product_catalog` | `dProduto` | Dimensões |
| `dataset_statistics` | remover (ver item 4) | — |
| — | `dCalendario` | Dimensões |
| — | `caminhoDados` | Parâmetros |

Renomear colunas para PascalCase (`NetSales`, `OrderDate`) é opcional: afeta medidas e visuais. Decida antes de o relatório crescer — hoje há só 1 página, é o momento mais barato.

### 11. Sem tratamento de erro nas colunas críticas

Depois de corrigir os tipos (item 2), proteja as colunas financeiras. Com `returnErrorValuesAsNull` ligado no modelo, um valor inválido vira nulo sem rastro:

```m
#"Validação Carga" = Table.AddColumn(#"Tipos Ajustados", "TemErroValor",
    each try [net_sales] <> null and [profit] <> null otherwise false, type logical),
```

Carregue com Enable load desmarcado uma query de contagem de erros, ou use a validação em QA e remova antes do deploy.

---

## 🔵 Infos

### 12. Steps com nomes default em português

Todas as queries usam `Fonte` / `#"Cabeçalhos Promovidos"` / `#"Tipo Alterado"`. `Fonte` é aceitável; renomeie o resto para o que a etapa faz: `#"Tipo Alterado"` → `#"Tipos Ajustados"`, e nomeie o novo `#"Colunas Selecionadas"`.

### 13. Zero comentários no M

Nenhuma das 5 queries tem `//`. Depois das correções, as decisões que precisam de comentário: a cultura `"en-US"` no `TransformColumnTypes` (não é óbvio num modelo pt-BR), o `QuoteStyle.Csv`, e por que `srcEcommerceSales` não carrega.

### 14. Nomes de arquivo em snake_case

A convenção para arquivos externos é kebab-case: `ecommerce-sales-customer-analytics-150k.csv`, `order-items.csv`, `customer-master.csv`, `product-catalog.csv`. Mudança cosmética — só vale junto com a troca de caminho do item 3, e exige renomear em `Data/` no mesmo commit.

---

## Checklist de boas práticas atendidas

- ✅ Todas as tabelas em `mode: import` e coerentes entre si (sem modelo composto acidental)
- ✅ `Table.PromoteHeaders` com `PromoteAllScalars=true`
- ✅ Tipagem explícita presente em todas as queries (o conjunto de tipos está errado, mas nada ficou como `Any`)
- ✅ Nenhum `Table.Buffer` desnecessário
- ✅ Nenhuma credencial ou token no código M
- ✅ `formatString` definido nas colunas numéricas do TMDL
- ✅ Cultura do modelo declarada (`pt-BR`) e cultura `pt-BR.tmdl` presente
- ✅ Nenhum merge/expand custoso em M (joins resolvidos por relacionamento)
- ⬜ Incremental Refresh: não aplicável (CSV local não folda; `RangeStart`/`RangeEnd` exigiriam fonte dobrável)

## Ordem sugerida de execução

1. Itens 3 e 7 — caminho parametrizado + encoding (destrava refresh reproduzível)
2. Item 1 — `QuoteStyle.Csv` (para de corromper 8,7% das linhas)
3. Item 2 — tipos decimais com cultura `en-US` (corrige todos os KPIs financeiros)
4. Item 6 — remoção antecipada de colunas
5. Itens 4, 5 e 8 — reestruturação do modelo estrela + `dCalendario`
6. Itens 9 a 14 — camadas, nomenclatura, comentários

Os passos 1 a 4 são correções de M isoladas e podem ir em um único commit. O passo 5 toca relacionamentos, medidas e visuais — trate como trabalho separado.


---

# Registro de aplicação — 2026-10-03

## Arquivos alterados

| Arquivo | Mudança |
|---|---|
| `definition/expressions.tmdl` | **novo** — parâmetro `caminhoDados`, `queryGroup: Parâmetros` |
| `definition/model.tmdl` | `ref expression caminhoDados` |
| `tables/order_items.tmdl` | partição + 9 colunas retipadas |
| `tables/product_catalog.tmdl` | partição + 3 colunas retipadas |
| `tables/customer_master.tmdl` | partição + 2 colunas retipadas |
| `tables/dataset_statistics.tmdl` | partição (+ step `Moeda Limpa`) + 4 colunas retipadas |
| `tables/ecommerce_..._150k.tmdl` | partição (+ step `Colunas Removidas`) + 10 colunas retipadas + **12 blocos `column` removidos** |

Em todas as 5 partições: caminho via `caminhoDados`, `QuoteStyle.Csv`, `Columns=N` removido,
`Encoding=65001` uniforme, step final renomeado para `#"Tipos Ajustados"`, cultura `"en-US"`
no `TransformColumnTypes` e comentários `//` nas decisões não óbvias.

## Desvio em relação ao item 6

A lista publicada no item 6 removia também as métricas da fato (`gross_sales`, `net_sales`,
`profit`...). **Isso não foi aplicado** — decidir o grão é o item 5, e tirar as métricas antes
dessa decisão deixaria o modelo sem fonte de receita no nível de pedido. Foram removidas **12
colunas**, só as que não dependem da decisão de grão:

- 9 duplicatas de `customer_master`: `customer_name`, `customer_age`, `gender`, `customer_segment`, `customer_city`, `customer_state`, `customer_country`, `region`, `customer_postal_code`
- 3 textos livres de alta cardinalidade sem consumo: `customer_review`, `campaign_name`, `coupon_code`

A fato passou de **46 para 34 colunas**. `customer_review` era a coluna de maior cardinalidade do modelo.

## Validação executada (sem abrir o Power BI)

Parsing e conversão simulados sobre os CSVs de `Data/`, com leitor que respeita aspas
(equivalente a `QuoteStyle.Csv`):

| Arquivo | Linhas | Colunas | Desalinhamento | Conversão de tipos |
|---|---:|---:|---|---|
| `ecommerce_..._150k.csv` | 138.116 | 46 | **nenhum** | OK |
| `order_items.csv` | 397.569 | 12 | nenhum | OK |
| `product_catalog.csv` | 1.175 | 9 | nenhum | OK |
| `customer_master.csv` | 25.000 | 11 | nenhum | OK |

As 11.954 linhas que antes produziam 47 campos agora resolvem em 46 — **o item 1 está
confirmado como corrigido**. Todo valor declarado `Int64.Type` é íntegro na fonte e todo
`Currency.Type` / `type number` converte.

## Achado novo: nulos estruturais (não é defeito)

`delivery_days`, `estimated_delivery_days` e `customer_rating` estão vazios em 24.557 linhas.
A distribuição mostra que é semântica, não sujeira:

| `order_status` | vazios | total |
|---|---:|---:|
| Completed | 0 | 113.559 |
| Returned | 9.462 | 9.462 |
| Cancelled | 8.398 | 8.398 |
| Pending | 6.697 | 6.697 |

Só pedido concluído tem prazo de entrega e nota. O nulo é a representação correta — **não
substitua por 0**, que distorceria qualquer média. Medidas de prazo e satisfação devem usar
`AVERAGE` (que ignora nulo), nunca `DIVIDE(SUM(...), COUNTROWS(...))` sobre a fato inteira.

## A conferir na primeira abertura do Desktop

1. **`ref expression caminhoDados` em `model.tmdl`** — é a forma correta pelo spec do TMDL para
   objeto em arquivo separado. Se o Desktop reclamar da linha, remova-a: `expressions.tmdl` é
   carregado pela convenção de pasta.
2. **Ajuste o valor de `caminhoDados`** se o repositório estiver em outro caminho.
3. **Refresh completo**, depois confira `SUM(order_items[net_sales])` contra o `Total Revenue`
   de `dataset_statistics` (`$177,134,263.74`) — a divergência que sobrar é o item 5.


---

# Registro de aplicação (2) — 2026-10-03, itens restantes

Aplicados agora: **5, 8, 9, 10, 11 e 14**. Com isso os 14 itens da auditoria estão
fechados, com duas exceções declaradas no fim desta seção.

## Achado crítico novo: o cabeçalho do pedido agrega no máximo 5 itens

Ao validar a escolha do grão, as duas fontes de receita divergiram em **R$ 15.264.613,55**:

| Fonte | Receita (`net_sales`) |
|---|---:|
| Soma dos itens (`order-items.csv`, 397.569 linhas) | **192.398.877,29** |
| `net_sales` do cabeçalho (`ecommerce-...-150k.csv`) | 177.134.263,74 |
| `Total Revenue` de `dataset-statistics.csv` | 177.134.263,74 |

O diagnóstico é conclusivo, não estatístico:

- **126.890 pedidos (91,9%) batem exatamente** entre cabeçalho e soma dos itens.
- Nos **11.226 divergentes, 11.226 (100%)** têm o cabeçalho igual à soma de um
  **prefixo** dos itens — nunca a um subconjunto arbitrário.
- O prefixo nunca passa de **5 itens**. A divergência é 0% em pedidos de 1 item,
  1,1% com 2 itens, 15,2% com 5 itens e **100% em todo pedido com 6 itens ou mais**.
- `quantity` diverge exatamente nos mesmos 11.226 pedidos.
- `order_status` não explica nada: a taxa é ~8% uniforme nos quatro status.

Ou seja: o cabeçalho foi calculado com um **teto de 5 itens por pedido** e ignora o que
passa disso. `order_items` é a lista completa; o cabeçalho está **subcontado em 7,9%**, e
o `Total Revenue` do `dataset-statistics.csv` herda o mesmo erro — ele bate centavo a
centavo com o cabeçalho, o que confirma que foi gerado a partir dele.

**Consequência prática:** `fOrderItems` é a única fonte de receita e lucro do modelo, e os
números do dashboard passam a ser ~8% mais altos que o `Total Revenue` do CSV de
estatísticas. **A divergência é o CSV estar errado, não o modelo.** Por isso as métricas
aditivas foram removidas de `fOrders` em vez de se escolher o cabeçalho como oficial.

Isso também inverte a conferência sugerida no registro anterior: `SUM(fOrderItems[NetSales])`
**não deve** bater com `Total Revenue`. Se bater, é sinal de que algo voltou a ler o
cabeçalho.

## Item 5 — modelo estrela

Tabelas do modelo (5), todas `mode: import`:

| Tabela | Grão | Origem |
|---|---|---|
| `fOrderItems` | item de pedido (397.569) | `srcOrderItems` |
| `fOrders` | pedido (138.116) | `srcEcommerceSales`, sem métricas aditivas |
| `dCliente` | cliente (25.000) | `srcCustomerMaster` |
| `dProduto` | produto (1.175) | `srcProductCatalog` |
| `dCalendario` | dia (2021-2025) | `fxGeraCalendario` |

Removidas de `fOrders`: `quantity`, `gross_sales`, `discount_amount`, `tax_amount`,
`shipping_cost`, `net_sales`, `product_cost`, `profit`, `profit_margin_percentage` —
as nove colunas que duplicavam `fOrderItems` em outro grão (e, como se viu acima, com
valores truncados). `fOrders` mantém só o que é de pedido e não é aditivo: status, canal,
pagamento, entrega, devolução, nota, pontos de fidelidade e CLV.

Relacionamentos (todos muitos-para-um, direção única):

```
dCalendario[Data]  1 --> N  fOrders[OrderDate]
dCliente[CustomerId] 1 --> N  fOrders[CustomerId]
fOrders[OrderId]   1 --> N  fOrderItems[OrderId]
dProduto[ProductId] 1 --> N  fOrderItems[ProductId]
```

**Trade-off assumido:** `order-items.csv` não traz data nem cliente, então o filtro de
data e de cliente chega a `fOrderItems` **através de `fOrders`** (cadeia de dois saltos,
não um floco de neve acidental). A alternativa seria um merge de 397 mil linhas contra 138
mil para denormalizar `OrderDate` e `CustomerId` na fato de itens. Não foi feito: custa um
merge em cada refresh para economizar um salto de propagação que o engine resolve bem.
Se no futuro a fato de itens crescer muito, reavalie.

**Integridade referencial validada (zero órfãos, chaves únicas):**

| Verificação | Resultado |
|---|---|
| `fOrders[OrderId]` único | 138.116 de 138.116 |
| `dProduto[ProductId]` único | 1.175 de 1.175 |
| `dCliente[CustomerId]` único | 25.000 de 25.000 |
| `fOrderItems[OrderId]` órfãos | 0 |
| `fOrderItems[ProductId]` órfãos | 0 |
| `fOrders[CustomerId]` órfãos | 0 |

Nenhuma relação vai gerar linha em branco.

## Item 8 — Auto Date/Time e dCalendario

- `__PBI_TimeIntelligenceEnabled` passou de `1` para `0`.
- Removidas as tabelas ocultas `LocalDateTable_9cefa3d2…` e `DateTableTemplate_d980badf…`,
  e a `variation` que `order_date` tinha para a hierarquia automática.
- `dCalendario` criada com `dataCategory: Time` e `isKey` na coluna `Data` — é assim que
  o TMDL marca uma tabela de datas.
- Colunas: `Data`, `Ano`, `Trimestre`, `MesNome`, `MesAbrev`, `AnoMes`, `DiaMes`,
  `SemanaAno`, `DiaSemanaNome`, `FimDeSemana`, mais as auxiliares `*Num` ocultas servindo
  de `sortByColumn` (sem elas "Abril" vem antes de "Janeiro").
- O range **não é fixo**: sai de `List.Min`/`List.Max` de `order_date` no staging,
  arredondado para anos inteiros. Hoje resolve 2021-01-01 a 2025-12-31; se os CSVs
  mudarem, acompanha sem edição.
- Objetos `calendar` do TMDL não foram usados: exigem `compatibilityLevel: 1702` e o
  modelo está em `1606`.

## Itens 9 e 10 — camadas e nomenclatura

`expressions.tmdl` passou a concentrar tudo que não carrega no modelo:

| Expressão | Papel |
|---|---|
| `caminhoDados` | parâmetro de caminho |
| `fxGeraCalendario` | função (a primeira do projeto) |
| `srcEcommerceSales`, `srcOrderItems`, `srcCustomerMaster`, `srcProductCatalog` | staging: fonte + tipos |
| `statsCargaCsv` | ex-`dataset_statistics`, fora do modelo |
| `qaValidacaoCarga` | conferência de carga |

As 5 tabelas do modelo ficaram com partição de uma linha (`Fonte = srcX`), ou duas no caso
de `fOrders`. Toda leitura de arquivo e tipagem vive na camada de staging — mexer em
encoding ou tipo agora é um lugar só.

Colunas renomeadas para PascalCase via `sourceColumn:`, que mapeia o nome do modelo para o
nome do CSV. O M continua produzindo `order_id` e o modelo expõe `OrderId`, sem renomeação
em M. Chaves estrangeiras (`fOrderItems[OrderId]`, `fOrderItems[ProductId]`,
`fOrders[CustomerId]`) ficaram `isHidden` — não servem para arrastar em visual.

Descrições `///` foram adicionadas nas tabelas, nas chaves e nas colunas que têm pegadinha:
`DeliveryDays` e `CustomerRating` (nulo estrutural), `CustomerLifetimeValue` e
`CustomerOrderCount` (repetem por pedido, não somar).

## Item 11 — conferência de carga

`qaValidacaoCarga` (não carregada) devolve `Tabela`, `Coluna`, `Linhas`, `Nulos` para as
colunas críticas das quatro stagings. Para usar: Transformar dados → selecionar a query.
Os nulos esperados estão comentados no próprio M, para não assustar quem abrir depois.

## Item 14 — nomes de arquivo

CSVs renomeados em `Data/` para kebab-case, com `git mv` para preservar o histórico:
`ecommerce-sales-customer-analytics-150k.csv`, `order-items.csv`, `customer-master.csv`,
`product-catalog.csv`, `dataset-statistics.csv`. As referências no M acompanharam.

## Duas coisas que não foram feitas

1. **Grupos de query** (pastas Parâmetros / Staging / Fatos no editor). Foi a causa do erro
   `Cannot resolve all the paths while de-serializing Database` na primeira tentativa: um
   `queryGroup:` só pode apontar para um objeto `queryGroup` declarado no `model.tmdl`, e
   esse modelo não tem nenhum (`Query Groups: 0`). A sintaxe de declaração não está
   documentada na skill, então a linha foi removida em vez de ser adivinhada. O jeito
   seguro é criar os grupos uma vez pela interface do Power Query e deixar o Desktop
   serializar — depois as expressões podem apontar para eles.
2. **`dataAccessOptions / returnErrorValuesAsNull`** continua ligada no `model.tmdl`. É ela
   que transformava erro de conversão em nulo silencioso. Desligar deixaria erros futuros
   visíveis, mas muda o comportamento de runtime do modelo inteiro, e não estava no escopo
   da auditoria. `qaValidacaoCarga` cobre a necessidade sem esse risco. Vale decidir à
   parte.

## Estado final

- Queries: **13** (5 tabelas + 8 expressões), contra 5 tabelas e 0 expressões no início.
- Parâmetros: 1 · Funções `fx`: 1 · Staging: 4 · Não carregadas: 8.
- Colunas na fato de pedidos: **25**, contra 46 originais.
- Fonte única de receita: `fOrderItems`.
