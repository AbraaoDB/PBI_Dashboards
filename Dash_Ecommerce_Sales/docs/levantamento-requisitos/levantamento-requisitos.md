# Levantamento de Requisitos — Dash_Ecommerce_Sales

Data: 2026-10-03 · Projeto: `Dash_Sales.pbip` · Modelo: `Dash_Sales.SemanticModel`

## Origem do material

Não houve reunião, cliente ou stakeholder. O material de entrada é a página pública do
dataset no Kaggle, mais a inspeção direta dos CSVs e do modelo semântico já construído.

| Item | Valor |
|---|---|
| Dataset | E-Commerce Sales Analytics Dataset |
| Autor | Shair Khan (`datascikhan`) |
| Licença | **CC0: Public Domain** |
| Tamanho | 88,6 MB · 15.729 downloads · 48.924 visualizações · 318 votos |
| Usability | 1.0 |
| Cobertura declarada | "150,000+ transactions spanning 2021–2025" |

**Isso muda a natureza do levantamento.** Sem stakeholder, não existe dor relatada: existe
tema analítico declarado pelo autor do dataset e defeito de dado observado. Toda dor abaixo
está marcada `[HIPÓTESE]` quando inferida, e `[CONFIRMADO]` só quando há citação literal da
página ou evidência medida nos arquivos. As perguntas que um cliente real precisaria
responder estão isoladas em `pendencias-cliente.md`.

---

## 1. Audiência e decisão

### Audiência

`[CONFIRMADO]` — a própria página declara o uso pretendido:

> "Exploratory Data Analysis and dashboarding (Power BI, Tableau)" · "Portfolio development
> and academic research"

- **Primária:** avaliador técnico de portfólio (recrutador, entrevistador, professor). Olha
  uma vez, por pouco tempo, e julga clareza de modelagem e rigor analítico.
- **Secundária:** o próprio autor do projeto, como base reutilizável para demonstrar DAX,
  modelagem e visual.
- **Simulada:** executivo de e-commerce. É a persona que dá sentido às perguntas de negócio,
  e o relatório deve se comportar como se ela existisse. `[HIPÓTESE]`

### Trabalho a ser feito

Entender a história **e** acompanhar performance. Não é monitoramento operacional: os dados
são históricos fechados (2021–2025) e não há carga incremental.

### Tom

Executivo na primeira página, analítico nas demais.

### Critério de sucesso

> O relatório funcionou se um avaliador que nunca viu o dataset conseguir, em menos de dois
> minutos e sem explicação verbal, dizer qual é a receita realizada do período, quanto dela
> se perde em devolução e cancelamento, quais categorias e canais sustentam a margem, e onde
> a operação de entrega falha — e se, ao conferir qualquer número contra o CSV, ele chegar
> ao mesmo valor.

A segunda metade do critério é a parte incomum e é deliberada: este dataset tem defeitos que
levam um dashboard descuidado a exibir números errados com aparência de certo (§6).

---

## 2. Dores

Derivadas dos temas que a página declara cobrir e dos defeitos medidos nos arquivos.

### D1 — Não se sabe quanto da receita é real

`[CONFIRMADO]` por medição. `order_items` registra valor para **todo** pedido, inclusive
`Cancelled` e `Returned`. Somar tudo infla a receita em **18,50%** (R$ 35.602.662,66).

- **Área:** financeiro, comercial
- **Impacto:** toda decisão de preço, meta e mix baseada em receita bruta superestima
- **Frequência:** permanente, em todo visual que não filtre status
- **Se não resolver:** o dashboard mente por 18,5% e ninguém percebe, porque o número é
  "a soma da coluna"

### D2 — Há duas fontes de receita, e a mais "oficial" está errada

`[CONFIRMADO]` por medição (detalhe em `docs/power-query/auditoria-2026-10-03.md`).
O cabeçalho do pedido agrega no máximo 5 itens; pedidos com 6+ itens têm o excedente
ignorado. Subconta **7,9%**, e o `dataset_statistics.csv` herda o erro.

- **Área:** toda a análise financeira
- **Impacto:** divergência de R$ 15.264.613,55 entre as duas leituras possíveis
- **Se não resolver:** dois analistas chegam a números diferentes e ninguém sabe quem errou

### D3 — Valores de moedas diferentes são somados como se fossem a mesma

`[CONFIRMADO]` por medição. São 7 moedas, cada uma ligada a um país. O preço unitário médio
é praticamente idêntico nas sete (244,85 a 247,43; mínimo 6,31 e máximo 1.482,95 em todas),
o que mostra que **os valores não foram convertidos para uma moeda comum** — são a mesma
faixa numérica com rótulo diferente.

- **Área:** financeiro, análise regional
- **Impacto:** `SUM(NetSales)` global não é dinheiro; é a soma de sete unidades distintas
- **Se não resolver:** qualquer comparação entre países é inválida, e o total não significa
  nada

### D4 — Devolução e cancelamento não têm leitura própria

`[CONFIRMADO]` pela estrutura: 6,85% de devolução e 6,08% de cancelamento, com 8 motivos
registrados, mas nenhuma medida que isole o prejuízo.

- **Área:** operação, qualidade, atendimento
- **Impacto:** 12,93% dos pedidos terminam mal e não há visibilidade do custo

### D5 — A operação de entrega não é medida contra a própria promessa

`[CONFIRMADO]` por medição: 14,87% das entregas concluídas passam do prazo estimado
(16.886 de 113.559), e a média real (4,56 dias) é maior que a estimada (4,30).

- **Área:** logística
- **Impacto:** a promessa ao cliente é sistematicamente otimista e isso não aparece

### D6 — O investimento em marketing não é confrontado com o retorno

`[CONFIRMADO]` que os dados existem: 10 canais de marketing e `customer_acquisition_cost`
por cliente (R$ 1.053.993,76 no total). `[HIPÓTESE]` de que a decisão sobre mix de canal é
tomada sem essa visão.

- **Área:** marketing
- **Impacto:** Organic Search gera R$ 38,7 M e YouTube R$ 5,7 M, sem leitura de eficiência

### D7 — Não se sabe de onde vem a margem

`[CONFIRMADO]` que o dado existe por produto, categoria, marca e fornecedor.
`[HIPÓTESE]` da dor. Agravante medido: 87% dos itens têm desconto, média de 14,71% e
máximo de 60%.

- **Área:** comercial, compras
- **Impacto:** desconto quase universal sem leitura do efeito na margem

### D8 — A base de clientes não é segmentada para ação

`[CONFIRMADO]` que há segmento, região, idade, gênero, CLV e pontos de fidelidade.
`[HIPÓTESE]` da dor. **Ressalva grave:** 99,76% dos pedidos são de cliente recorrente (ver §6).

- **Área:** CRM
- **Impacto:** 25.000 clientes tratados como bloco único

### D9 — Satisfação é coletada e não é usada

`[CONFIRMADO]`: nota média 3,68 e sentimento em 4 níveis, sem cruzamento com entrega,
devolução ou produto.

- **Área:** qualidade, produto
- **Impacto:** 792 avaliações negativas sem rastreio de causa

### D10 — Fidelidade não tem controle de passivo

`[CONFIRMADO]` por medição: 14.362.828 pontos ganhos contra 7.172.934 resgatados — **49,9%
de resgate**, logo ~7,19 milhões de pontos em aberto.

- **Área:** financeiro, CRM
- **Impacto:** passivo relevante sem acompanhamento

---

## 3. Perguntas de negócio

Cada pergunta traz a **decisão esperada** — sem isso ela não entra no relatório.

### Financeiro e comercial

| # | Pergunta | Decisão esperada |
|---|---|---|
| P1 | Qual a receita realizada do período, e como evolui no tempo? | Definir a base de qualquer meta futura |
| P2 | Quanto da receita bruta se perde em cancelamento e devolução? | Priorizar ou não um projeto de redução de perdas |
| P3 | Qual a margem, e quanto dela é imposto? | Avaliar se o negócio é rentável sem o efeito tributário |
| P4 | Qual o ticket médio, e ele varia por canal, região e tipo de cliente? | Decidir onde empurrar venda cruzada |
| P5 | Quais categorias, marcas e fornecedores concentram receita e margem? | Rever mix e negociação de compra |
| P6 | O desconto concedido se converte em volume? | Manter, cortar ou redirecionar a política de desconto |
| P7 | Qual canal de vendas (`SalesChannel`) tem melhor margem? | Priorizar investimento por canal |

### Operação e logística

| # | Pergunta | Decisão esperada |
|---|---|---|
| P8 | Qual o percentual de entregas dentro do prazo prometido? | Definir SLA realista |
| P9 | Quais centros de distribuição atrasam mais? | Intervir no CD específico |
| P10 | O atraso na entrega aumenta a devolução e derruba a nota? | Justificar investimento em logística com impacto em receita |
| P11 | Quais são os motivos de devolução e quanto cada um custa? | Atacar o motivo de maior custo |
| P12 | Qual método de envio entrega melhor pelo custo que tem? | Renegociar ou trocar transportadora |

### Cliente e marketing

| # | Pergunta | Decisão esperada |
|---|---|---|
| P13 | Quais segmentos e regiões geram mais valor? | Direcionar esforço comercial |
| P14 | Qual canal de marketing traz a receita mais rentável? | Realocar orçamento de mídia |
| P15 | O custo de aquisição se paga frente ao valor do cliente? | Definir teto de CAC |
| P16 | Qual o passivo de pontos de fidelidade em aberto? | Provisionar o passivo |
| P17 | A satisfação varia por categoria de produto? | Revisar sortimento ou fornecedor |
| P18 | Qual método de pagamento concentra falha? | Ajustar meios de pagamento |

---

## 4. KPIs

Formato comum a todos: **grão de cálculo** = item de pedido (`fOrderItems`) salvo indicação
contrária; **atualização** = manual, carga full (CSV local, sem incremental);
**dimensões de análise** = `dCalendario`, `dCliente` (segmento, região, país, cidade,
idade, gênero), `dProduto` (categoria, subcategoria, marca, fornecedor) e os atributos de
`fOrders` (status, canal, pagamento, envio, CD, marketing, sentimento).

Os critérios de aceite trazem **o valor esperado para o período completo sem filtro**, medido
direto nos CSVs. Toda medida nova deve reproduzir esse número.

### Bloco A — Financeiro (primeira entrega)

| KPI | Fórmula de negócio | Critério de aceite (total do período) |
|---|---|---|
| **Receita Bruta** | soma de `NetSales` de todos os pedidos | `192.398.877,29` |
| **Receita Realizada** | `Receita Bruta` apenas de pedidos `Completed` | `156.796.214,63` |
| **Receita Perdida** | `Receita Bruta` de `Cancelled` + `Returned` | `26.457.549,67` (13,75%) |
| **Receita em Aberto** | `Receita Bruta` de `Pending` | `9.145.112,99` (4,75%) |
| **Lucro** | soma de `Profit` | `82.698.038,55` |
| **Margem %** | `Lucro ÷ Receita Bruta` | `42,98%` |
| **Margem sem Imposto %** | `(Lucro − TaxAmount) ÷ (Receita − TaxAmount)` | `36,97%` |
| **Desconto Concedido** | soma de `DiscountAmount` | `35.649.166,70` |
| **Desconto Médio %** | média de `DiscountPercentage` | `14,71%` |
| **Imposto** | soma de `TaxAmount` | `18.361.184,20` |
| **Frete** | soma de `ShippingCost` | `3.343.161,70` |
| **Pedidos** | contagem distinta de `fOrders[OrderId]` | `138.116` |
| **Itens Vendidos** | contagem de linhas de `fOrderItems` | `397.569` |
| **Unidades** | soma de `Quantity` | `842.366` |
| **Ticket Médio (AOV)** | `Receita Bruta ÷ Pedidos` | `1.393,02` |
| **Itens por Pedido** | `Itens ÷ Pedidos` | `2,88` |

**Nota sobre o AOV:** `1.393,02`, e não os `1.282,50` do `dataset_statistics.csv`. A
diferença é o defeito D2 — o CSV de estatísticas usa a receita subcontada.

### Bloco B — Operação (primeira entrega)

| KPI | Fórmula de negócio | Critério de aceite |
|---|---|---|
| **Taxa de Devolução** | pedidos `Returned` ÷ pedidos | `6,85%` |
| **Taxa de Cancelamento** | pedidos `Cancelled` ÷ pedidos | `6,08%` |
| **Taxa de Conclusão** | pedidos `Completed` ÷ pedidos | `82,22%` |
| **Prazo Médio de Entrega** | média de `DeliveryDays` (só concluídos) | `4,56` dias |
| **Prazo Médio Estimado** | média de `EstimatedDeliveryDays` | `4,30` dias |
| **Aderência ao Prazo %** | entregas com `DeliveryDays ≤ Estimated` ÷ concluídas | `85,13%` |
| **Entregas Atrasadas** | contagem de `DeliveryDays > Estimated` | `16.886` |
| **Nota Média** | média de `CustomerRating` (só concluídos) | `3,68` |
| **Avaliações Negativas** | pedidos com `ReviewSentiment = "Negative"` | `792` |

### Bloco C — Cliente e marketing (primeira entrega parcial)

| KPI | Fórmula de negócio | Critério de aceite | Escopo |
|---|---|---|---|
| **Clientes Ativos** | contagem distinta de clientes com pedido | `24.911` de `25.000` | 1ª entrega |
| **Receita por Cliente** | `Receita Bruta ÷ Clientes Ativos` | `7.723,45` | 1ª entrega |
| **Custo de Aquisição** | soma de `CustomerAcquisitionCost` em `dCliente` | `1.053.993,76` | 1ª entrega |
| **CAC Médio** | `Custo de Aquisição ÷ Clientes` | `42,16` | 1ª entrega |
| **Razão CLV / CAC** | média de `CustomerLifetimeValue` ÷ `CAC Médio` | — | 1ª entrega |
| **Pontos em Aberto** | `LoyaltyPointsEarned − LoyaltyPointsRedeemed` | `7.189.894` | 1ª entrega |
| **Taxa de Resgate** | `Redeemed ÷ Earned` | `49,94%` | 1ª entrega |
| **Receita por Canal de Marketing** | `Receita Bruta` por `MarketingChannel` | Organic Search `38.745.393,68` (maior) | 1ª entrega |

### Bloco D — Diferido, com motivo

| KPI | Por que fica para depois |
|---|---|
| **% Atingimento de Meta** | não existe tabela de metas — `[REQUER FONTE]` |
| **Crescimento YoY** | a receita é plana nos 5 anos (±0,8%); a medida existiria mostrando ~0% e passaria por erro. Ver §6.2 |
| **Novos vs Recorrentes** | apenas 334 pedidos de cliente novo em 138.116 (0,24%) — a análise é degenerada. Ver §6.3 |
| **Análise de Coorte / Retenção** | falta data de primeira compra do cliente e a base é 99,76% recorrente |
| **Receita por Campanha / Cupom** | `campaign_name` e `coupon_code` foram removidos do modelo na auditoria de ETL por não ter consumo. Restaurar é `[REQUER MODELO]` |
| **Análise de Sortimento Morto** | todos os 1.175 produtos do catálogo têm venda; não há estoque morto |
| **Receita em Moeda Única** | exige tabela de câmbio — `[REQUER FONTE]`. Ver §6.1 |
| **Análise de texto de avaliação** | `customer_review` foi removido na auditoria (alta cardinalidade, sem consumo); só `ReviewSentiment` ficou |

---

## 5. Regras de negócio

### Confirmadas por medição nos arquivos

| # | Regra | Evidência |
|---|---|---|
| R1 `[CONFIRMADO]` | `NetSales = GrossSales − DiscountAmount + TaxAmount + ShippingCost` | confere em 50.000 de 50.000 itens da amostra |
| R2 `[CONFIRMADO]` | `Profit = NetSales − ProductCost − ShippingCost` | confere em **397.569 de 397.569** itens (100%), soma das diferenças = 0 |
| R3 `[CONFIRMADO]` | Logo, `Profit` **inclui o imposto arrecadado**. A margem de 42,98% é inflada; sem imposto é 36,97% | derivado de R1 e R2 |
| R4 `[CONFIRMADO]` | `payment_status` é determinado pelo `order_status`: `Cancelled`→`Failed`, `Returned`→`Refunded`, `Completed`→`Paid` (102.253) ou `Pending` (11.306) | tabulação cruzada completa, sem exceção |
| R5 `[CONFIRMADO]` | Receita só é realizada em pedido `Completed`. `Cancelled` e `Returned` são perda; `Pending` é indefinido | R4 |
| R6 `[CONFIRMADO]` | `DeliveryDays`, `EstimatedDeliveryDays` e `CustomerRating` só existem em pedido `Completed` — nulo nos outros três status, sem exceção | 0 nulos em 113.559 `Completed`; 100% nulos nos 24.557 restantes |
| R7 `[CONFIRMADO]` | `ReturnReason` só é preenchido em pedido `Returned`; os 8 motivos somam exatamente 9.462 | contagem |
| R8 `[CONFIRMADO]` | `delivery_status = "Cancelled"` (24.557) **não** é o mesmo que `order_status = "Cancelled"` (8.398): marca "não houve entrega", cobrindo devolvido, cancelado e pendente | contagem |
| R9 `[CONFIRMADO]` | Moeda é determinada pelo país, 1:1 — AED/UAE, AUD/Australia, CAD/Canada, EUR/Germany, GBP/UK, INR/India, USD/USA | tabulação cruzada |
| R10 `[CONFIRMADO]` | `CustomerLifetimeValue` e `CustomerOrderCount` são atributos do cliente repetidos em cada pedido — nunca somar | modelagem, §4.2 de `modelagem-dimensional.md` |
| R11 `[CONFIRMADO]` | `dProduto[UnitPrice]` é preço de tabela; o praticado é `fOrderItems[UnitPrice]` | colunas distintas nas duas fontes |

### Hipóteses a validar

| # | Regra | Status |
|---|---|---|
| R12 | `Pending` deve ser excluído da receita realizada e exibido em separado, não somado nem descartado | `[HIPÓTESE]` |
| R13 | A métrica de aderência ao prazo deve usar `DeliveryDays ≤ EstimatedDeliveryDays`, e não `DeliveryStatus`, porque `DeliveryStatus` mistura `Early` e `On Time` e trata não-entrega como `Cancelled` | `[HIPÓTESE]` |
| R14 | Devolução conta no mês do pedido, não no mês da devolução — não existe data de devolução na fonte | `[HIPÓTESE]`, forçada pelo dado |
| R15 | O `customer_acquisition_cost` é único por cliente (não por pedido), logo não se rateia por pedido | `[HIPÓTESE]` |

### Pendentes de decisão humana

Detalhadas em `pendencias-cliente.md`: tratamento de moeda (Q1), status `Pending` (Q2),
definição oficial de margem (Q3), e se vale restaurar campanha e cupom (Q4).

---

## 6. Ressalvas de dados — leia antes de desenhar qualquer visual

Este dataset é sintético. As seis ressalvas abaixo foram medidas, não supostas, e cada uma
tem potencial de produzir um dashboard tecnicamente correto e analiticamente falso.

### 6.1 Moedas não normalizadas `🔴`

Sete moedas, com distribuição de preço unitário idêntica:

| Moeda | Receita | Pedidos | Preço médio | Mín | Máx |
|---|---:|---:|---:|---:|---:|
| USD | 111.361.507,18 | 82.600 | 245,06 | 6,31 | 1.482,95 |
| GBP | 30.158.057,24 | 20.235 | 244,85 | 6,31 | 1.482,95 |
| EUR | 17.313.749,83 | 11.593 | 245,06 | 6,31 | 1.482,95 |
| CAD | 10.038.295,60 | 7.015 | 245,95 | 6,31 | 1.482,95 |
| AUD | 9.518.800,27 | 6.794 | 245,18 | 6,31 | 1.482,95 |
| INR | 8.368.138,18 | 5.707 | 245,96 | 6,31 | 1.482,95 |
| AED | 5.640.328,99 | 4.172 | 247,43 | 6,31 | 1.482,95 |

Mínimo e máximo **idênticos nas sete** é a prova: os valores foram gerados na mesma faixa e
só rotulados com moeda diferente. Em dado real, 1 INR não teria o mesmo alcance que 1 USD.

**Tratamento recomendado:** declarar na primeira página que os valores estão em **unidade
monetária única (u.m.)** e tratar `Currency` como atributo descritivo, nunca como unidade.
Não construir conversão fictícia. Decisão em Q1.

### 6.2 Receita plana nos cinco anos `🟡`

| Ano | Receita |
|---|---:|
| 2021 | 38.785.263,03 |
| 2022 | 38.506.756,18 |
| 2023 | 38.418.258,96 |
| 2024 | 38.537.986,65 |
| 2025 | 38.150.612,47 |

Variação total de 1,6% entre o maior e o menor ano. **Não há tendência nem sazonalidade
real.** Qualquer página construída em cima de crescimento, YoY ou projeção vai exibir linha
reta e ~0%. Usar o tempo para contexto e comparação de mix, não para narrativa de
crescimento.

### 6.3 A base é praticamente toda recorrente `🟡`

| Campo | Distribuição |
|---|---|
| `is_repeat_customer` | 137.782 verdadeiro · **334 falso** |
| `customer_type` | Loyal 129.447 · Returning 8.335 · **New 334** |

0,24% de pedidos de cliente novo. Aquisição, coorte, retenção e funil de primeira compra
são inviáveis. `CustomerOrderCount` médio alto confirma.

### 6.4 Motivos de devolução sem sinal `🟡`

Os 8 motivos estão quase uniformemente distribuídos (1.137 a 1.237 cada, amplitude de 8%).
Uma análise de causa-raiz vai mostrar oito barras iguais. Vale exibir como composição, não
como diagnóstico.

### 6.5 A razão CLV / CAC é implausível `🟡`

Medido: CLV médio de `8.971,64` contra CAC médio de `42,16` — razão de **212,8×**. Em
e-commerce real, 3× a 5× já é considerado saudável, e acima de 10× indica subinvestimento em
aquisição. Aqui os dois campos foram gerados de forma independente, sem relação entre si.

A medida `[Razao CLV sobre CAC]` existe e está correta em relação à fonte, mas **o número não
sustenta conclusão de negócio**. Exibir com ressalva, ou tratar CLV e CAC separadamente.

### 6.6 O cabeçalho do pedido subconta 7,9% `🔴`

Já corrigido no modelo e documentado em `docs/power-query/auditoria-2026-10-03.md`: as
métricas aditivas vivem só em `fOrderItems`. A consequência para os requisitos é que
**nenhum número do dashboard vai bater com o `dataset_statistics.csv` na parte monetária**.
Os indicadores de contagem batem (devolução 6,85%, cancelamento 6,08%, nota 3,68, clientes
24.911); os de dinheiro, não — e o certo é o do modelo.

---

## 7. Inventário do modelo existente

Fonte: leitura dos `.tmdl` em `Dash_Sales.SemanticModel/definition/`.

```
Fatos:
  fOrderItems (grão: item de pedido, 397.569) — FKs: PedidoSK, ProdutoSK
  fOrders     (grão: pedido, 138.116) — chave PedidoSK; FKs: ClienteSK, OrderDate
              + 13 atributos degenerados (status, canal, pagamento, envio, CD, marketing…)

Dimensões:
  dCalendario (dia, 1.827) — dataCategory: Time, isKey em Data, 2021-2025
  dCliente    (cliente, 25.000) — isKey em ClienteSK
  dProduto    (produto, 1.175) — isKey em ProdutoSK

Não carregam: caminhoDados, fxGeraCalendario, srcEcommerceSales, srcOrderItems,
              srcCustomerMaster, srcProductCatalog, statsCargaCsv, qaValidacaoCarga

Medidas existentes: 63, em `tables/Medidas.tmdl` (12 pastas)

Trabalho de modelo faltando:
  - campaign_name e coupon_code, se Q4 aprovar restaurá-los
  - tabela de metas, se houver decisão de criar baseline sintético
  - (tabela `Medidas` e as 63 medidas: FEITO em 2026-10-03)

Riscos:
  - nenhum órfão, nenhuma chave duplicada (validado contra os CSVs)
  - dCliente[CustomerName] com 25.000 valores → exige busca, não lista
  - fOrders[OrderId] com 138.116 valores → nunca em slicer, só em drill-through
  - warehouse com 19 valores, marketing_channel com 10 → cabem em slicer
  - SK posicional: estável só enquanto a carga for full refresh
```

**Status predominante na matriz:** `[VIÁVEL]` — 52 dos 63 itens rastreados. Os 11 restantes
são os diferidos e os que exigem fonte nova.

---

## 8. Entrega e restrições

| Item | Definição |
|---|---|
| **Alvo** | PBIP local, versionado em Git. Sem workspace Fabric definido |
| **Permissão de editar o modelo** | Sim, total — modelo e relatório são do próprio projeto |
| **Acessibilidade** | WCAG AA por padrão: contraste mínimo 4.5:1 em texto, nunca cor como único portador de informação, ordem de tabulação coerente, texto alternativo nos visuais |
| **Ferramental** | Power BI Desktop (2.158.1177.0, set/2026), edição direta de TMDL/PBIR, Git |
| **Atualização** | Manual. CSV local, full refresh, sem gateway e sem incremental |
| **Volume** | 563 mil linhas no total — confortável para Import |
| **Ressalvas de dados** | §6 por completo. A de moeda e a do cabeçalho são bloqueantes para leitura financeira |

---

## 9. Escopo da primeira entrega × diferido

### Primeira entrega — 4 páginas

| Página | Audiência | Conteúdo | Perguntas |
|---|---|---|---|
| **1. Visão Executiva** | executivo | Receita Realizada, Margem, AOV, Pedidos, Taxa de Conclusão; evolução mensal; composição da receita por status; top categorias e canais | P1, P2, P3, P4 |
| **2. Comercial e Produto** | analista | Receita e margem por categoria, subcategoria, marca, fornecedor; efeito do desconto; preço praticado vs tabela | P5, P6, P7 |
| **3. Operação e Qualidade** | operador | Aderência ao prazo, atraso por CD e método de envio, devolução por motivo, nota por categoria, cruzamento atraso × devolução × nota | P8, P9, P10, P11, P12, P17 |
| **4. Cliente e Marketing** | analista | Segmento, região, CLV vs CAC, canal de marketing, pontos de fidelidade, método de pagamento | P13, P14, P15, P16, P18 |

Mais uma faixa fixa de ressalva na página 1, declarando unidade monetária única e a
divergência proposital com o `dataset_statistics.csv`. Não é enfeite: sem ela o avaliador
que conferir o CSV conclui que o dashboard está errado.

### Diferido

Bloco D do §4, com o motivo de cada item. Em resumo: metas e câmbio faltam na fonte;
crescimento, coorte e aquisição são inviabilizados pelos defeitos 6.2 e 6.3; campanha,
cupom e texto de avaliação exigem restaurar colunas removidas.

---

## 10. Interações esperadas

### Slicers globais (sincronizados nas 4 páginas)

- **Período** — `dCalendario[Data]`, slicer de intervalo relativo ou entre datas
- **Ano** — `dCalendario[Ano]`, botão de seleção única
- **Status do pedido** — `fOrders[OrderStatus]`, com `Completed` pré-selecionado na página 1

### Slicers por página

- Página 2: categoria, marca, fornecedor (`dProduto`)
- Página 3: CD (`Warehouse`, 19 valores), método de envio, motivo de devolução
- Página 4: segmento, região, país (`dCliente`), canal de marketing

### Busca em alta cardinalidade

`dCliente[CustomerName]` (25.000) e `dProduto[ProductName]` (1.175) entram como campo de
busca, nunca como lista. `fOrders[OrderId]` (138.116) não entra em slicer — só aparece em
drill-through e tabela de detalhe.

### Drill-through

- Categoria ou produto → detalhe de itens de `fOrderItems`
- Cliente → pedidos daquele cliente, com status, data e valor
- CD ou motivo de devolução → pedidos afetados

### Navegação

Barra lateral com as 4 páginas via bookmarks. Botão "limpar filtros" por página.
Tooltip de página para os KPIs do bloco A, exibindo a decomposição por status.

---

## 11. Critérios de qualidade atendidos

- Nenhum KPI sem pergunta de negócio associada — rastreado em `matriz-rastreabilidade.md`
- Nenhuma pergunta sem decisão esperada declarada — §3
- Nenhuma regra crítica implícita — 11 regras confirmadas por medição, 4 hipóteses marcadas
- Necessidade distinguida de solução: as ressalvas do §6 **negam** parte do que a página do
  dataset promete (crescimento, aquisição, coorte) em vez de prometer o que o dado não dá
- Nenhuma pergunta ao cliente cuja resposta já esteja no material: as 4 pendências são
  decisões de negócio, não dúvidas de dado
