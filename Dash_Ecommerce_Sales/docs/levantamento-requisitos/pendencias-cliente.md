# Pendências — decisões que o levantamento não pode tomar sozinho

Data: 2026-10-03 · Projeto: Dash_Ecommerce_Sales

Este projeto não tem cliente: a fonte é um dataset público CC0. As quatro pendências abaixo
são decisões de negócio, não dúvidas sobre o dado — o dado já foi medido e está descrito em
`levantamento-requisitos.md` §6. **Enquanto não houver resposta, vale a opção marcada como
padrão**, para o projeto não travar.

Nada aqui é pergunta cuja resposta esteja no material.

---

## Q1 🔴 — Como tratar as sete moedas?

### Pergunta exata

> "Os valores dos sete países estão todos na mesma faixa numérica — preço unitário entre
> 6,31 e 1.482,95 tanto em USD quanto em INR. Isso indica que a fonte não converteu nada
> para uma moeda comum. Você quer que o dashboard (a) declare 'unidade monetária única' e
> trate a moeda como atributo descritivo, (b) exiba os valores separados por moeda e nunca
> some o total global, ou (c) forneça uma tabela de câmbio para converter?"

### Por que não posso decidir

Afeta o número mais visível do relatório. A opção (a) é uma simplificação explícita; a (b)
elimina o KPI de receita total; a (c) exige um dado que não existe na fonte.

### Impacto de cada resposta

| Opção | Consequência |
|---|---|
| **(a) unidade única** ⟵ **padrão** | Mantém todos os KPIs. Exige faixa de ressalva visível na página 1. Honesto e simples |
| (b) separar por moeda | Receita total deixa de existir; a página 1 vira comparação entre 7 blocos não somáveis |
| (c) tabela de câmbio | `[REQUER FONTE]`. Converter dado sintético por câmbio real produz número de aparência precisa e significado nenhum — **não recomendo** |

### Evidência

`levantamento-requisitos.md` §6.1 — tabela de preço médio, mínimo e máximo por moeda.

---

## Q2 🟡 — O pedido `Pending` entra na receita?

### Pergunta exata

> "4,85% dos pedidos (6.697, somando R$ 9.145.112,99) estão com status `Pending`, e o
> pagamento deles está dividido entre `Paid` (1.997), `Failed` (2.031) e `Pending` (2.669).
> Esse valor deve entrar na receita realizada, ficar de fora, ou aparecer como linha
> separada de 'receita em aberto'?"

### Por que não posso decidir

É regra de reconhecimento de receita, que varia por política contábil. R12 está marcada
`[HIPÓTESE]` por isso.

### Impacto de cada resposta

| Opção | Receita do período |
|---|---|
| **Linha separada** ⟵ **padrão** | Realizada `156.796.214,63` + Em aberto `9.145.112,99` exibidos lado a lado |
| Incluir na realizada | `165.941.327,62` — mistura venda confirmada com venda incerta |
| Excluir e não mostrar | `156.796.214,63` — esconde 4,75% da operação |

### Evidência

`levantamento-requisitos.md` §5 regra R4 (tabulação `order_status` × `payment_status`,
sem exceção) e bloco A do §4.

---

## Q3 🟡 — Qual é a definição oficial de margem?

### Pergunta exata

> "A coluna `profit` da fonte equivale exatamente a `net_sales − product_cost −
> shipping_cost`, confirmado em 397.569 de 397.569 linhas. Como `net_sales` inclui o
> imposto arrecadado, esse `profit` conta imposto como lucro, e a margem sai em 42,98%.
> Excluindo o imposto, cai para 36,97%. Qual das duas é a margem oficial do dashboard?"

### Por que não posso decidir

As duas são defensáveis: a primeira é fiel à fonte, a segunda é economicamente mais
correta. A diferença é de 6 pontos percentuais no indicador mais citado depois da receita.

### Impacto de cada resposta

| Opção | Margem | Observação |
|---|---|---|
| **Exibir as duas** ⟵ **padrão** | 42,98% e 36,97% | "Margem" e "Margem sem Imposto", lado a lado |
| Só a da fonte | 42,98% | Reproduz a fonte, mas infla o resultado |
| Só sem imposto | 36,97% | Mais correta, mas não bate com nenhuma coluna da fonte |

### Evidência

`levantamento-requisitos.md` §5 regras R1, R2 e R3 — ambas as identidades verificadas em
100% das linhas.

---

## Q4 🔵 — Vale restaurar campanha, cupom e texto de avaliação?

### Pergunta exata

> "Na auditoria de ETL eu removi `campaign_name`, `coupon_code` e `customer_review` do
> modelo, porque nenhuma delas tinha consumo e `customer_review` era a coluna de maior
> cardinalidade do projeto. Restaurar as duas primeiras habilita análise de campanha e de
> cupom. Vale o custo de memória, ou essas análises ficam fora do escopo?"

### Por que não posso decidir

É troca entre escopo analítico e tamanho do modelo, e depende de haver interesse real nessas
duas perguntas.

### Impacto de cada resposta

| Opção | Consequência |
|---|---|
| **Manter removidas** ⟵ **padrão** | Modelo enxuto; análise de campanha e cupom fora do escopo |
| Restaurar `campaign_name` e `coupon_code` | Habilita 2 itens do bloco D. Custo de memória baixo (cardinalidade moderada) |
| Restaurar também `customer_review` | Só se houver intenção real de análise de texto. É a coluna mais cara do dataset e o Power BI não é a ferramenta para isso |

### Evidência

`docs/power-query/auditoria-2026-10-03.md`, registro de aplicação, item 6.

---

## Pendência informativa — não requer decisão

### A descrição do dataset não corresponde aos arquivos

A página do Kaggle lista `Order_Priority`, `Loyalty_Tier`, `Unit_Cost`,
`Acquisition_Channel`, `Total_Sales`, `Discount_Pct` e `Profit_Margin`. Os CSVs publicados
não têm `Order_Priority` nem `Loyalty_Tier`, e os demais aparecem com outros nomes.

Não há o que decidir: **os requisitos seguem os arquivos**. Fica registrado para que ninguém
procure, mais adiante, um campo que a descrição promete e o dataset não entrega.

---

## Resumo para quem for responder

| # | Assunto | Severidade | Padrão se não houver resposta |
|---|---|---|---|
| Q1 | Tratamento das 7 moedas | 🔴 | Unidade monetária única, com ressalva na página 1 |
| Q2 | Status `Pending` na receita | 🟡 | Linha separada de "receita em aberto" |
| Q3 | Definição de margem | 🟡 | Exibir as duas, com e sem imposto |
| Q4 | Restaurar campanha e cupom | 🔵 | Manter removidas |

Nenhuma delas bloqueia o início da construção das medidas: as quatro têm padrão definido, e
os valores do bloco A do §4 já refletem esses padrões.
