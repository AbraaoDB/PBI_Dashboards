# Matriz de Rastreabilidade — Dash_Ecommerce_Sales

Data: 2026-10-03 · Fonte: `levantamento-requisitos.md`

**Atualizado em 2026-10-03:** a coluna **Medida DAX** foi preenchida — as 63 medidas existem
em `tables/Medidas.tmdl` e cada valor esperado está conferido em `docs/medidas-dax.md`. As
colunas **Visual** e **Página** seguem com `-` até o relatório PBIR ser construído.

**Legenda de Status:** `[VIÁVEL]` o modelo já responde · `[REQUER MODELO]` falta medida,
coluna ou relacionamento · `[REQUER FONTE]` o dado não existe em nenhuma tabela.

**Fonte** usa os nomes do modelo: `fOrderItems`, `fOrders`, `dCliente`, `dProduto`,
`dCalendario`.

---

## Bloco A — Financeiro e comercial

| Dor | Pergunta de negócio | KPI | Fórmula de negócio | Fonte | Medida DAX | Visual | Página | Prioridade | Status |
|---|---|---|---|---|---|---|---|---|---|
| D1 | P1 Qual a receita realizada do período? | Receita Realizada | soma de `NetSales` com `OrderStatus = "Completed"` | fOrderItems, fOrders | [Receita Realizada] | - | - | Alta | [VIÁVEL] |
| D1 | P1 Como a receita evolui no tempo? | Receita Bruta | soma de `NetSales` | fOrderItems | [Receita Bruta] | - | - | Alta | [VIÁVEL] |
| D1 | P2 Quanto se perde em cancelamento e devolução? | Receita Perdida | soma de `NetSales` com status `Cancelled` ou `Returned` | fOrderItems, fOrders | [Receita Perdida] | - | - | Alta | [VIÁVEL] |
| D1 | P2 Quanto está indefinido? | Receita em Aberto | soma de `NetSales` com status `Pending` | fOrderItems, fOrders | [Receita em Aberto] | - | - | Média | [VIÁVEL] |
| D1 | P2 Qual o percentual de perda? | % Receita Perdida | `Receita Perdida ÷ Receita Bruta` | fOrderItems, fOrders | [% Receita Perdida] | - | - | Alta | [VIÁVEL] |
| D7 | P3 Qual a margem? | Lucro | soma de `Profit` | fOrderItems | [Lucro] | - | - | Alta | [VIÁVEL] |
| D7 | P3 Qual a margem? | Margem % | `Lucro ÷ Receita Bruta` | fOrderItems | [Margem %] | - | - | Alta | [VIÁVEL] |
| D7 | P3 Quanto da margem é imposto? | Margem sem Imposto % | `(Lucro − TaxAmount) ÷ (Receita − TaxAmount)` | fOrderItems | [Margem sem Imposto %] | - | - | Alta | [VIÁVEL] |
| D7 | P3 Quanto de imposto é arrecadado? | Imposto | soma de `TaxAmount` | fOrderItems | [Imposto] | - | - | Média | [VIÁVEL] |
| D1 | P4 Qual o ticket médio? | Ticket Médio (AOV) | `Receita Bruta ÷ Pedidos` | fOrderItems, fOrders | [Ticket Medio] | - | - | Alta | [VIÁVEL] |
| D1 | P4 Quantos pedidos? | Pedidos | contagem distinta de `fOrders[OrderId]` | fOrders | [Pedidos] | - | - | Alta | [VIÁVEL] |
| D1 | P4 Quantos itens por pedido? | Itens por Pedido | `Itens Vendidos ÷ Pedidos` | fOrderItems, fOrders | [Itens por Pedido] | - | - | Média | [VIÁVEL] |
| D1 | P4 Qual o volume vendido? | Unidades | soma de `Quantity` | fOrderItems | [Unidades] | - | - | Média | [VIÁVEL] |
| D1 | P4 Quantas linhas de venda? | Itens Vendidos | contagem de linhas de `fOrderItems` | fOrderItems | [Itens Vendidos] | - | - | Baixa | [VIÁVEL] |
| D7 | P5 Quais categorias concentram receita e margem? | Receita Bruta × `dProduto[ProductCategory]` | medida existente cruzada com dimensão | fOrderItems, dProduto | [Receita Bruta] + [Margem %] | - | - | Alta | [VIÁVEL] |
| D7 | P5 Quais marcas e fornecedores? | Receita Bruta × `Brand` / `Supplier` | idem | fOrderItems, dProduto | [Receita Bruta] + [Margem %] | - | - | Média | [VIÁVEL] |
| D7 | P6 O desconto se converte em volume? | Desconto Concedido | soma de `DiscountAmount` | fOrderItems | [Desconto Concedido] | - | - | Alta | [VIÁVEL] |
| D7 | P6 Qual o desconto típico? | Desconto Médio % | média de `DiscountPercentage` | fOrderItems | [Desconto Medio %] | - | - | Alta | [VIÁVEL] |
| D7 | P6 O preço praticado difere do de tabela? | Variação de Preço % | `AVERAGE(fOrderItems[UnitPrice]) ÷ AVERAGE(dProduto[UnitPrice]) − 1` | fOrderItems, dProduto | [Variacao de Preco %] | - | - | Média | [VIÁVEL] |
| D7 | P7 Qual canal de vendas tem melhor margem? | Margem % × `fOrders[SalesChannel]` | idem | fOrderItems, fOrders | [Margem %] | - | - | Alta | [VIÁVEL] |
| D1 | P1 Qual o custo de frete embutido? | Frete | soma de `ShippingCost` | fOrderItems | [Frete] | - | - | Baixa | [VIÁVEL] |

## Bloco B — Operação, logística e qualidade

| Dor | Pergunta de negócio | KPI | Fórmula de negócio | Fonte | Medida DAX | Visual | Página | Prioridade | Status |
|---|---|---|---|---|---|---|---|---|---|
| D4 | P2 Qual a taxa de devolução? | Taxa de Devolução | pedidos `Returned` ÷ pedidos | fOrders | [Taxa de Devolucao %] | - | - | Alta | [VIÁVEL] |
| D4 | P2 Qual a taxa de cancelamento? | Taxa de Cancelamento | pedidos `Cancelled` ÷ pedidos | fOrders | [Taxa de Cancelamento %] | - | - | Alta | [VIÁVEL] |
| D4 | P2 Qual a taxa de conclusão? | Taxa de Conclusão | pedidos `Completed` ÷ pedidos | fOrders | [Taxa de Conclusao %] | - | - | Alta | [VIÁVEL] |
| D5 | P8 Cumprimos o prazo prometido? | Aderência ao Prazo % | pedidos com `DeliveryDays ≤ EstimatedDeliveryDays` ÷ concluídos | fOrders | [Aderencia ao Prazo %] | - | - | Alta | [VIÁVEL] |
| D5 | P8 Quanto tempo leva a entrega? | Prazo Médio de Entrega | média de `DeliveryDays` | fOrders | [Prazo Medio de Entrega] | - | - | Alta | [VIÁVEL] |
| D5 | P8 Quanto prometemos? | Prazo Médio Estimado | média de `EstimatedDeliveryDays` | fOrders | [Prazo Medio Estimado] | - | - | Média | [VIÁVEL] |
| D5 | P8 Qual o tamanho do desvio? | Desvio de Prazo | `Prazo Médio − Prazo Estimado` | fOrders | [Desvio de Prazo] | - | - | Média | [VIÁVEL] |
| D5 | P9 Quais CDs atrasam mais? | Entregas Atrasadas × `Warehouse` | contagem de `DeliveryDays > Estimated` por CD | fOrders | [Entregas Atrasadas] | - | - | Alta | [VIÁVEL] |
| D5 | P12 Qual método de envio entrega melhor? | Aderência ao Prazo % × `ShippingMethod` | idem | fOrders | [Aderencia ao Prazo %] | - | - | Média | [VIÁVEL] |
| D5 | P12 Quanto custa cada método? | Frete Médio × `ShippingMethod` | `Frete ÷ Pedidos` por método | fOrderItems, fOrders | [Frete Medio por Pedido] | - | - | Média | [VIÁVEL] |
| D5+D4 | P10 O atraso aumenta a devolução? | Taxa de Devolução × faixa de atraso | taxa cruzada com `DeliveryDays − Estimated` | fOrders | [Taxa de Devolucao %] | - | - | Alta | [VIÁVEL] |
| D5+D9 | P10 O atraso derruba a nota? | Nota Média × faixa de atraso | idem | fOrders | [Nota Media] | - | - | Alta | [VIÁVEL] |
| D4 | P11 Quais os motivos de devolução? | Pedidos Devolvidos × `ReturnReason` | contagem por motivo | fOrders | [Pedidos Devolvidos] | - | - | Alta | [VIÁVEL] |
| D4 | P11 Quanto custa cada motivo? | Receita Perdida × `ReturnReason` | soma de `NetSales` por motivo | fOrderItems, fOrders | [Receita Perdida] | - | - | Alta | [VIÁVEL] |
| D9 | P17 Qual a satisfação? | Nota Média | média de `CustomerRating` | fOrders | [Nota Media] | - | - | Alta | [VIÁVEL] |
| D9 | P17 Quantas avaliações negativas? | Avaliações Negativas | contagem de `ReviewSentiment = "Negative"` | fOrders | [Avaliacoes Negativas] + [% Avaliacoes Negativas] | - | - | Média | [VIÁVEL] |
| D9 | P17 A satisfação varia por categoria? | Nota Média × `ProductCategory` | idem, via fOrderItems | fOrders, dProduto | **[Nota Media por Produto]** | - | - | Média | [VIÁVEL] |

## Bloco C — Cliente, marketing e fidelidade

| Dor | Pergunta de negócio | KPI | Fórmula de negócio | Fonte | Medida DAX | Visual | Página | Prioridade | Status |
|---|---|---|---|---|---|---|---|---|---|
| D8 | P13 Quantos clientes compram? | Clientes Ativos | contagem distinta de clientes com pedido | fOrders, dCliente | [Clientes Ativos] | - | - | Alta | [VIÁVEL] |
| D8 | P13 Quanto cada cliente gera? | Receita por Cliente | `Receita Bruta ÷ Clientes Ativos` | fOrderItems, dCliente | [Receita por Cliente] | - | - | Alta | [VIÁVEL] |
| D8 | P13 Quais segmentos geram valor? | Receita Bruta × `CustomerSegment` | idem | fOrderItems, dCliente | [Receita Bruta] | - | - | Alta | [VIÁVEL] |
| D8 | P13 Quais regiões geram valor? | Receita Bruta × `Region` / `CustomerCountry` | idem | fOrderItems, dCliente | [Receita Bruta] | - | - | Alta | [VIÁVEL] |
| D8 | P13 Qual o perfil demográfico? | Receita Bruta × faixa de `CustomerAge` / `Gender` | idem | fOrderItems, dCliente | [Receita Bruta] | - | - | Baixa | [VIÁVEL] |
| D6 | P14 Qual canal de marketing traz receita? | Receita Bruta × `MarketingChannel` | idem | fOrderItems, fOrders | [Receita Bruta] | - | - | Alta | [VIÁVEL] |
| D6 | P14 Qual canal traz a receita mais rentável? | Margem % × `MarketingChannel` | idem | fOrderItems, fOrders | [Margem %] | - | - | Alta | [VIÁVEL] |
| D6 | P15 Quanto gastamos para adquirir? | Custo de Aquisição | soma de `CustomerAcquisitionCost` | dCliente | [Custo de Aquisicao] | - | - | Alta | [VIÁVEL] |
| D6 | P15 Qual o CAC por cliente? | CAC Médio | `Custo de Aquisição ÷ Clientes` | dCliente | [CAC Medio] | - | - | Alta | [VIÁVEL] |
| D6 | P15 O cliente se paga? | Razão CLV / CAC | `AVERAGE(CustomerLifetimeValue) ÷ CAC Médio` | fOrders, dCliente | [Razao CLV sobre CAC] | - | - | Alta | [VIÁVEL] |
| D10 | P16 Qual o passivo de pontos? | Pontos em Aberto | `soma(LoyaltyPointsEarned) − soma(LoyaltyPointsRedeemed)` | fOrders | [Pontos em Aberto] | - | - | Alta | [VIÁVEL] |
| D10 | P16 Qual a taxa de resgate? | Taxa de Resgate | `Redeemed ÷ Earned` | fOrders | [Taxa de Resgate %] | - | - | Média | [VIÁVEL] |
| D8 | P18 Qual meio de pagamento concentra falha? | Taxa de Falha × `PaymentMethod` | pedidos `PaymentStatus = "Failed"` ÷ pedidos por método | fOrders | [Taxa de Falha de Pagamento %] | - | - | Média | [VIÁVEL] |
| D8 | P18 Qual meio é mais usado? | Pedidos × `PaymentMethod` | contagem por método | fOrders | [Pedidos] | - | - | Baixa | [VIÁVEL] |

## Bloco D — Diferido

| Dor | Pergunta de negócio | KPI | Fórmula de negócio | Fonte | Medida DAX | Visual | Página | Prioridade | Status |
|---|---|---|---|---|---|---|---|---|---|
| — | Estamos acima ou abaixo da meta? | % Atingimento de Meta | `Receita Realizada ÷ Meta do período` | — | - | - | - | Diferido | **[REQUER FONTE]** |
| D1 | A receita cresce ano a ano? | Crescimento YoY % | `Receita ÷ Receita do ano anterior − 1` | fOrderItems, dCalendario | - | - | - | Diferido | [REQUER MODELO] — receita plana, ver §6.2 |
| D8 | Quantos clientes são novos? | Novos vs Recorrentes | pedidos por `CustomerType` | fOrders | - | - | - | Diferido | [REQUER MODELO] — 0,24% novos, ver §6.3 |
| D8 | Os clientes retornam ao longo do tempo? | Retenção por Coorte | matriz de coorte por mês de 1ª compra | — | - | - | - | Diferido | **[REQUER FONTE]** — falta data de aquisição |
| D6 | Qual campanha performou melhor? | Receita por Campanha | `Receita Bruta` por `campaign_name` | — | - | - | - | Diferido | [REQUER MODELO] — coluna removida, restaurar (Q4) |
| D6 | O cupom gera incremento? | Receita por Cupom | `Receita Bruta` por `coupon_code` | — | - | - | - | Diferido | [REQUER MODELO] — coluna removida, restaurar (Q4) |
| D7 | Quais produtos não giram? | Produtos sem Venda | produtos de `dProduto` sem linha em `fOrderItems` | fOrderItems, dProduto | - | - | - | Diferido | [VIÁVEL] — mas o resultado é 0, ver §6 |
| D3 | Qual a receita em moeda única? | Receita em Moeda Única | `Receita × taxa de câmbio da data` | — | - | - | - | Diferido | **[REQUER FONTE]** — falta tabela de câmbio (Q1) |
| D9 | O que os clientes escrevem? | Temas da Avaliação | análise de texto de `customer_review` | — | - | - | - | Diferido | [REQUER MODELO] — coluna removida na auditoria |
| — | Qual a prioridade do pedido? | Pedidos por Prioridade | contagem por `Order_Priority` | — | - | - | - | Fora de escopo | **[REQUER FONTE]** — citado na página do Kaggle, **não existe nos CSVs** |
| — | Qual a distribuição por tier de fidelidade? | Pedidos por Loyalty Tier | contagem por `Loyalty_Tier` | — | - | - | - | Fora de escopo | **[REQUER FONTE]** — citado na página, **não existe nos CSVs** |

---

## Resumo

| Status | Itens |
|---|---:|
| `[VIÁVEL]` — medida criada e valor conferido | **52** |
| `[REQUER MODELO]` — diferido | 5 |
| `[REQUER FONTE]` | 5 |
| `[VIÁVEL]` sem trabalho de modelo | 1 |
| **Total rastreado** | **63** |

Os 52 itens da primeira entrega estão resolvidos no modelo: 63 medidas em
`tables/Medidas.tmdl`, distribuídas em 12 pastas, com o valor esperado de cada uma medido
nos CSVs (`docs/medidas-dax.md`). Falta apenas a camada visual — nenhum visual foi
construído ainda.

### Discrepância entre a página do Kaggle e os arquivos

A descrição do dataset lista campos que **não existem** nos CSVs publicados:
`Order_Priority`, `Loyalty_Tier`, `Unit_Cost`, `Acquisition_Channel`, `Total_Sales`,
`Discount_Pct`, `Profit_Margin`. Os arquivos reais trazem equivalentes de nome diferente
(`marketing_channel`, `product_cost`, `net_sales`, `discount_percentage`,
`profit_margin_percentage`) e **não trazem** prioridade nem tier de fidelidade — no lugar
deste há `loyalty_points_earned` e `loyalty_points_redeemed`.

Conclusão: a descrição é genérica e não foi mantida em sincronia com os arquivos.
**Os requisitos seguem os arquivos, não a descrição.**
