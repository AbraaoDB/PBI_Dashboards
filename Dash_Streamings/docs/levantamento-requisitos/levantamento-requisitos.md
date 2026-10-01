# Levantamento de Requisitos — Dash_Streamings

**Data:** 2026-09-28
**Atualização 2026-09-30:** fonte dos dados confirmada — ver abaixo. A divergência de fonte registrada na versão anterior está resolvida.
**Fonte dos dados:** Kaggle — [Streaming Content Catalog: Netflix, Prime, Disney+](https://www.kaggle.com/datasets/meruvakodandasuraj/streaming-content-catalog-netflix-prime-disney?select=yearly_release_trends.csv) (Meruva Kodanda Suraj). Dataset **sintético** para fins educacionais: nomes de plataformas, atores e produtoras são reais apenas para dar realismo; notas, orçamentos, prêmios e audiência são simulados.
**Arquivos no projeto:** 5 CSVs em `Data/`, idênticos aos publicados no dataset — 1 catálogo de títulos (`streaming_catalog.csv`) + 4 summaries agregados (país, gênero, plataforma, ano). O modelo semântico foi reestruturado em modelo estrela sobre `fStreamingCatalog` + 6 dimensões (ver [modelo-dimensional-2026-09-28.md](../modelagem/modelo-dimensional-2026-09-28.md)).
**Ressalva:** não houve reunião com cliente real (projeto de portfólio/estudo). O briefing abaixo é uma persona sintética construída a partir do contexto do dataset Kaggle e do conteúdo dos CSVs, seguindo o rigor da skill `requirements-discovery`.

---

## 0. Audiência e decisão

### Audiência primária: **Executivo de conteúdo/estratégia** de um streamer ou fundo de investimento em conteúdo
- Cargo típico: *Head of Content*, *VP Strategy*, *Head of Programming*.
- Rotina: revisão semanal (segunda-feira de manhã) para orientar decisões de aquisição, produção original e priorização de catálogo por plataforma/região.

### Audiência secundária: **Analista de mercado / Growth**
- Explora o catálogo por país, gênero, plataforma. Compara performance e qualidade. Faz drill em títulos específicos.

### Audiência terciária: **Marketing / Aquisição de assinantes**
- Precisa entender qual plataforma domina qual gênero em qual país (guia campanhas competitivas).

### Trabalho a ser feito
- **Executivo:** entender a história (visão geral do mercado) + acompanhar performance (comparar plataformas ao longo do tempo).
- **Analista:** achar outliers (título com muito prêmio e pouco orçamento; plataforma sub-representada num gênero) + explorar registros individuais.
- **Growth:** comparar entidades (plataforma × gênero × país).

### Tom
- Página 1 executiva (cards de KPI, tendência, ranking).
- Página 2 analítica (matriz, tabelas com busca, drill-through).
- Página 3 narrativa (comparação plataforma × plataforma).

### Critério de sucesso
> "O relatório funcionou se, na segunda-feira de manhã, o VP de conteúdo identificar em menos de 3 minutos:
> (1) qual plataforma cresceu mais em número de títulos no último ano,
> (2) em qual gênero há espaço competitivo (poucos títulos, alta nota IMDb),
> (3) quais três títulos do último trimestre entregam maior retorno de audiência por dólar de orçamento —
> sem precisar pedir ajuda a um analista."

---

## 1. Dores extraídas

### D1 — Comparação entre plataformas é manual
- **Declarada:** "Não temos uma foto única para comparar Netflix, Prime, Disney+ lado a lado — cada análise vem de uma planilha diferente."
- **Área:** Estratégia de conteúdo.
- **Impacto:** decisões de investimento se baseiam em amostras enviesadas (a plataforma que "grita mais alto" no fim do mês).
- **Frequência:** semanal.
- **Consequência:** perda de janela competitiva em gêneros onde um concorrente ganha share.

### D2 — Falta visão temporal do catálogo
- **Declarada:** "A gente sabe quantos títulos temos hoje, mas não sabe direito quando entraram, e se estamos crescendo ou perdendo peso."
- **Área:** Programação / Aquisição.
- **Impacto:** ritmo de aquisição sem baseline; pipeline invisível.
- **Frequência:** mensal (fechamento de mês).
- **Consequência:** contratos de licenciamento negociados no escuro.

### D3 — Qualidade do catálogo não é medida
- **Declarada:** "Ninguém sabe se o que a gente coloca no ar é bom. Quando o IMDb cai, a gente descobre pela imprensa."
- **Área:** Curadoria.
- **Impacto:** cauda longa de títulos ruins ocupa espaço de merchandising.
- **Frequência:** contínua.
- **Consequência:** churn (não medido aqui, mas apontado como sintoma).

### D4 — Orçamento × retorno é opaco
- **Declarada:** "A gente aprova US$ 100M pra um filme e não tem métrica de eficiência: quantas horas de audiência esse dinheiro comprou?"
- **Área:** Finanças / Greenlight.
- **Impacto:** viés por título "estrelado" em detrimento de apostas eficientes.
- **Frequência:** por ciclo de aprovação (trimestral).
- **Consequência:** ROI de conteúdo sub-otimizado.

### D5 — Distribuição geográfica é caixa-preta
- **Declarada:** "Qual país tem mais nossos títulos? Que gênero funciona na Coreia mas não no Brasil? Ninguém responde na hora."
- **Área:** Expansão internacional.
- **Impacto:** priorização de mercados por intuição.
- **Frequência:** trimestral.
- **Consequência:** investimento em conteúdo local sem base.

### D6 — Prêmios e reconhecimento não entram na conversa financeira
- **Declarada:** "Ganhar Emmy é bom, mas ninguém junta prêmios com horas assistidas e com orçamento numa tela só."
- **Área:** Estratégia de originais.
- **Impacto:** decisões de "prestige content" vs "commercial content" sem trade-off explícito.
- **Frequência:** anual (temporada de premiações).
- **Consequência:** portfólio desbalanceado.

---

## 2. Perguntas de negócio

| # | Pergunta | Origem | Decisão esperada |
|---|---|---|---|
| Q1 | Quantos títulos temos hoje no catálogo global? | D1, D2 | Baseline mensal |
| Q2 | Como o catálogo cresceu nos últimos 5 anos (por ano de adição)? | D2 | Ritmo de aquisição |
| Q3 | Qual plataforma tem mais títulos por gênero? | D1 | Aposta competitiva |
| Q4 | Qual a nota média IMDb / Rotten Tomatoes por plataforma? | D3 | Curadoria por streamer |
| Q5 | Quais são os top 10 títulos por horas assistidas no período? | D4 | Foco de marketing |
| Q6 | Qual o orçamento médio por plataforma e como ele se traduz em horas assistidas? | D4 | Eficiência (ROI de audiência) |
| Q7 | Quais países dominam a origem dos títulos por plataforma? | D5 | Expansão / dublagem / legenda |
| Q8 | Qual a proporção Movie vs TV Show no catálogo, por plataforma? | D1, D3 | Balanceamento de formato |
| Q9 | Quais títulos ganharam mais prêmios com menor orçamento (eficiência de reconhecimento)? | D6 | Aposta em prestige com custo baixo |
| Q10 | Como evolui a distribuição de classificação etária no catálogo (rating)? | D3 | Adequação de audiência |
| Q11 | Qual gênero cresce mais rápido em número de títulos nos últimos 3 anos? | D2, D5 | Tendência de programação |
| Q12 | Qual a variação % de títulos adicionados este ano vs. ano anterior (YoY)? | D2 | Alerta de aceleração/desaceleração |
| Q13 | Qual plataforma tem catálogo mais recente vs. mais legado (idade média de lançamento)? | D3 | Estratégia de novidade |
| Q14 | Há títulos com IMDb alto (≥8) e baixa audiência (bottom 20% em horas)? | D3, D4 | Merchandising: joias escondidas |
| Q15 | Qual a duração média por tipo (filme em minutos, série em episódios/temporadas)? | D1 | Padrão de formato |

---

## 3. KPIs candidatos

### KPI-1 · Total de títulos
- **Objetivo:** dimensionar o catálogo global e por segmento.
- **Fórmula:** `COUNTROWS(fStreamingCatalog)`.
- **Granularidade:** título.
- **Filtros:** período (data_added), plataforma, gênero, país, tipo, rating.
- **Dimensões:** todas as 6.
- **Fonte:** `fStreamingCatalog`.
- **Freq. de atualização:** mensal (na atualização do CSV).
- **Aceite:** total sem filtro bate com `COUNT(show_id)` no CSV.
- **Escopo:** 1ª entrega.
- **Interações:** slicer global de período; visíveis em card e cartão de destaque.

### KPI-2 · IMDb Médio
- **Objetivo:** medir qualidade percebida do catálogo.
- **Fórmula:** `AVERAGE(fStreamingCatalog[imdb_rating])`.
- **Granularidade:** média sobre títulos filtrados.
- **Filtros:** plataforma, gênero, país, período.
- **Fonte:** `fStreamingCatalog[imdb_rating]`.
- **Aceite:** validado contra planilha de conferência (10 títulos amostrados por plataforma).
- **Escopo:** 1ª entrega.
- **Nota:** score não soma — `summarizeBy: none` obrigatório (já aplicado).

### KPI-3 · Rotten Tomatoes Médio
- **Objetivo:** cruzar percepção crítica com percepção de público (IMDb).
- **Fórmula:** `AVERAGE(fStreamingCatalog[rotten_tomatoes_score])`.
- **Aceite:** dois eixos num scatter (KPI-2 × KPI-3) devem separar plataformas visualmente.
- **Escopo:** 1ª entrega.

### KPI-4 · Horas Assistidas (Milhões)
- **Objetivo:** medir engajamento absoluto.
- **Fórmula:** `SUM(fStreamingCatalog[hours_watched_million])`.
- **Filtros:** plataforma, período, gênero, tipo.
- **Aceite:** total no card = soma da coluna.
- **Escopo:** 1ª entrega.

### KPI-5 · Orçamento Total (US$ Mi)
- **Objetivo:** custo de produção agregado.
- **Fórmula:** `SUM(fStreamingCatalog[budget_million_usd])`.
- **Escopo:** 1ª entrega.

### KPI-6 · Horas por Dólar de Orçamento (Eficiência)
- **Objetivo:** ROI de audiência por dólar investido.
- **Fórmula:** `DIVIDE( SUM(hours_watched_million), SUM(budget_million_usd) )` — em horas (milhões) por US$ milhão.
- **Aceite:** título com orçamento zero é excluído (denominador). Documentar.
- **Escopo:** 1ª entrega. **KPI protagonista para D4.**

### KPI-7 · Prêmios Ganhos (Total)
- **Objetivo:** volume de reconhecimento.
- **Fórmula:** `SUM(fStreamingCatalog[awards_won])`.
- **Escopo:** 1ª entrega.

### KPI-8 · Prêmios por Milhão de Orçamento
- **Objetivo:** eficiência de "prestige content".
- **Fórmula:** `DIVIDE( SUM(awards_won), SUM(budget_million_usd) )`.
- **Escopo:** 1ª entrega.

### KPI-9 · Títulos Adicionados no Período
- **Objetivo:** ritmo de aquisição/produção.
- **Fórmula:** `CALCULATE( COUNTROWS(fStreamingCatalog), USERELATIONSHIP entre date_added e dCalendario[Data] )` — o relacionamento já é o ativo, então: `COUNTROWS(fStreamingCatalog)` com filtro de `dCalendario`.
- **Escopo:** 1ª entrega.

### KPI-10 · Var. % Títulos YoY
- **Objetivo:** alerta de aceleração/desaceleração.
- **Fórmula:** `DIVIDE( [KPI-9] - CALCULATE([KPI-9], SAMEPERIODLASTYEAR(dCalendario[Data])), CALCULATE([KPI-9], SAMEPERIODLASTYEAR(dCalendario[Data])) )`.
- **Aceite:** valor formatado como % com uma casa; sinal claro (verde ↑ / vermelho ↓).
- **Escopo:** 1ª entrega.

### KPI-11 · Idade Média do Catálogo
- **Objetivo:** medir se o portfólio é recente ou legado.
- **Fórmula:** `AVERAGEX( fStreamingCatalog, YEAR(TODAY()) - fStreamingCatalog[release_year] )`.
- **Escopo:** 1ª entrega.

### KPI-12 · Duração Média (Filme, min)
- **Objetivo:** padrão de formato.
- **Fórmula:** `CALCULATE( AVERAGE(fStreamingCatalog[duration_minutes]), fStreamingCatalog[type] = "Movie" )`.
- **Escopo:** 1ª entrega.

### KPI-13 · Temporadas Médias (Série)
- **Fórmula:** `CALCULATE( AVERAGE(fStreamingCatalog[num_seasons]), fStreamingCatalog[type] = "TV Show" )`.
- **Escopo:** 1ª entrega.

### KPI-14 · Diversidade Geográfica (# Países Únicos)
- **Objetivo:** medir espalhamento internacional do catálogo.
- **Fórmula:** `DISTINCTCOUNT(fStreamingCatalog[country])`.
- **Escopo:** 1ª entrega.

### KPI-15 · Índice de Novidade (Títulos ≤ 12 meses / Total)
- **Objetivo:** % do catálogo que é "quente".
- **Fórmula:** `DIVIDE( CALCULATE([KPI-1], fStreamingCatalog[date_added] >= EDATE(TODAY(), -12)), [KPI-1] )`.
- **Escopo:** 1ª entrega.

### KPIs diferidos (exigem fonte adicional — fora do dataset atual)
- **KPI-D1 · Assinantes por Plataforma** — exige base de assinantes por serviço.
- **KPI-D2 · ARPU (Ad Revenue por Usuário)** — exige base de assinantes por serviço.
- **KPI-D3 · Churn Mensal Médio** — exige base de assinantes por serviço.
- **KPI-D4 · Distribuição Etária (6 faixas)** — exige base de assinantes por serviço.
- **KPI-D5 · Distribuição de Devices (6 categorias)** — exige base de assinantes por serviço.
- **KPI-D6 · Crescimento de Assinantes 2020–2024** — exige base de assinantes por serviço.

---

## 4. Regras de negócio

| # | Regra | Status |
|---|---|---|
| R1 | Um título aparece uma única vez no catálogo global (chave `show_id`). | `[CONFIRMADO]` — coluna presente e única no CSV. |
| R2 | `type` só assume dois valores: "Movie" e "TV Show". | `[HIPÓTESE]` — validar via `DISTINCT(type)`. |
| R3 | `imdb_rating` é escala 0–10 com uma casa decimal; `rotten_tomatoes_score` é % 0–100. | `[HIPÓTESE]` — CSV atual estava tipado como Int64 (corrigido para double). Validar amostra. |
| R4 | `hours_watched_million` é em milhões (unidade explícita no nome). | `[CONFIRMADO]` |
| R5 | `budget_million_usd` é em US$ milhões (unidade no nome). | `[CONFIRMADO]` |
| R6 | Um título com `budget_million_usd = 0` NÃO entra no cálculo de KPI-6 (eficiência) — evita divisão por zero e distorção. | `[CONFIRMADO]` (regra técnica) |
| R7 | `date_added` é a data em que o título entrou no catálogo da plataforma; `release_year` é o ano de produção original — são diferentes. | `[CONFIRMADO]` |
| R8 | Análise temporal padrão usa `date_added` (relacionamento ativo com `dCalendario`). | `[CONFIRMADO]` |
| R9 | `country` é o país principal de produção; `genres` (multivalor) contém gêneros secundários; `primary_genre` é o principal. Análise padrão usa `primary_genre`. | `[CONFIRMADO]` |
| R10 | `rating` (classificação etária) segue padrão MPAA/TV Parental (G, PG, PG-13, R, TV-14, TV-MA, etc.). | `[HIPÓTESE]` — validar valores presentes. |
| R11 | Fim de trimestre = dia 31/03, 30/06, 30/09, 31/12 (padrão calendário civil). | `[CONFIRMADO]` |
| R12 | Semana começa na segunda-feira (`WEEKDAY(..., 2)` já aplicado em `dCalendario`). | `[CONFIRMADO]` |
| R13 | Título com IMDb null ou zero é excluído do cálculo de KPI-2. | `[PENDENTE CLIENTE]` — hoje o modelo não filtra. |
| R14 | "Última janela" padrão para KPI-15 (novidade) é 12 meses; ajustável. | `[PENDENTE CLIENTE]` |

---

## 5. Inventário do modelo existente

Fatos: **`fStreamingCatalog`** (grão: título; chaves-FK ocultas para `dType`, `dPlatform`, `dGenre`, `dCountry`, `dRating`, `dCalendario`).

Dimensões: **`dCalendario`** (marcada como Time, 2008–2026), **`dPlatform`**, **`dGenre`**, **`dCountry`**, **`dType`**, **`dRating`** (todas calculadas via `DISTINCT` sobre o fato).

Parâmetros: **`caminhoDados`** (path da pasta `Data/`).

Medidas existentes: **nenhuma** — tabela `Medidas` a criar (skill `dax`).

Relacionamentos: 6, todos 1:N unidirecionais, ativos.

**Trabalho de modelo provavelmente faltando (identificado durante a discovery):**
- Tabela `Medidas` com os 15 KPIs acima.
- Se uma fonte de dados por serviço for incorporada no futuro (ex.: o dataset Kaggle "Global Streaming Services"): `fStreamingServices` (grão: serviço) + atributos ricos em `dPlatform` (parent_company, launch_year, subscribers_millions, arpu_usd, churn_rate_pct).
- Coluna de busca em `fStreamingCatalog[title]` — 25 col, ~milhares de linhas: usuário vai querer buscar.
- Hierarquia em `dCalendario`: Ano → Trimestre → Mês → Data (para drill).
- Consolidar `genres` (multivalor) numa bridge `bGenres` se surgir requisito de filtrar por gênero secundário — hoje `primary_genre` resolve.

**Riscos:**
- CSV local (não escala; sem query folding).
- Sem `refreshPolicy` — full refresh a cada carga.
- `date_added` como dateTime mas semanticamente é date — sem prejuízo hoje.
- `duration` mistura semântica (filme: "142 min"; série: "3 Seasons") — coluna texto; usar `duration_minutes` para filmes e `num_seasons`/`num_episodes` para séries.
- Título com múltiplos países em `country` (multivalor "US, UK") pode inflar `dCountry` — validar `DISTINCT(country)` e decidir se normaliza.

---

## 6. Entrega e restrições

- **Alvo:** PBIP local (Git-friendly), publicação futura em workspace Fabric quando o cliente decidir.
- **Permissão de editar o modelo:** sim (full — inclui adicionar fatos, dimensões, medidas, roles).
- **Acessibilidade:** WCAG AA (contraste, alt-text, ordem de leitura por página).
- **Idioma:** PT-BR nas labels e medidas; identificadores M/DAX em inglês.
- **Ressalvas de dados:**
  - CSVs são o snapshot publicado no Kaggle ([Streaming Content Catalog: Netflix, Prime, Disney+](https://www.kaggle.com/datasets/meruvakodandasuraj/streaming-content-catalog-netflix-prime-disney?select=yearly_release_trends.csv)); dados sintéticos — não citar como números reais de mercado.
  - Os 4 CSVs de summary foram descontinuados no modelo — as métricas viram medidas DAX sobre a fato.
  - **Fonte confirmada (resolvido em 2026-09-30):** os CSVs vêm do dataset de catálogo de títulos citado acima. Assinantes, ARPU, churn, demografia e devices **não existem** nesta fonte.
- **Ferramental disponível:** Power BI Desktop, PBIP/TMDL, MCP `powerbi-modeling-mcp`, `pbi-cli`.

---

## 7. Escopo — primeira entrega × diferido

### Primeira entrega (V1)
- **Modelo:** star schema atual (já pronto) + tabela `Medidas` com KPI-1 a KPI-15.
- **Páginas:**
  1. **Executive Overview** — 6 cards de KPI (KPI-1, 4, 5, 7, 9, 10) + tendência de KPI-9 (12 meses) + Top 5 plataformas.
  2. **Platform Battle** — matriz plataforma × gênero (color-scaled por # títulos) + scatter IMDb × RT.
  3. **Efficiency Lab** — scatter Orçamento × Horas Assistidas (KPI-6 como cor) + Top 10 "joias escondidas" (Q14).
  4. **Global Map** — mapa por país (# títulos), + tabela drill-through de títulos por país.
  5. **Awards & Prestige** — barras horizontais por título com mais prêmios; scatter Awards × Budget (KPI-8).

### Diferido (V2)
- Assinantes, ARPU, churn, demografia e devices: **fora do escopo** com a fonte atual; exigem uma segunda fonte por serviço — ver `pendencias-cliente.md` P1.
- Página **Subscribers & Growth** (só com a segunda fonte).
- Página **Audience Demographics** (idade e devices).
- Time intelligence avançado (rolling 12m, moving average).
- RLS por região (se surgir requisito multi-tenant).
- Refresh incremental — só faz sentido com fonte maior que CSV local.

---

## 8. Interações esperadas

- **Slicers globais** (aplicáveis a todas as páginas via sync slicers):
  - Período (slicer relativo em `dCalendario[Data]`, default: últimos 24 meses).
  - Plataforma (`dPlatform[Platform]`).
  - Tipo (`dType[Tipo]`).
- **Slicers por página:**
  - Executive Overview: nenhum extra (só globais).
  - Platform Battle: Gênero.
  - Efficiency Lab: faixa de orçamento (range).
  - Global Map: continente (calculated column em `dCountry` — diferido).
  - Awards & Prestige: Ano de lançamento.
- **Busca:** obrigatória em `fStreamingCatalog[title]` (alta cardinalidade) — visual "Slicer com pesquisa".
- **Drill-through:**
  - `dPlatform[Platform]` → página oculta "Plataforma detalhe" (todos os títulos, tabela).
  - `dCountry[Pais]` → página oculta "País detalhe".
- **Bookmarks/Navegação:** botão de página em cada tela (padrão).
- **Tooltip customizada** por título (mini card com title, poster placeholder, IMDb, RT, awards).
- **Tema:** dark mode alinhado a identidade de streamer (a definir na skill `dataviz-html-dax` / mockup Figma).

---

## Anexos

- Matriz canônica: [matriz-rastreabilidade.md](matriz-rastreabilidade.md)
- Pendências: [pendencias-cliente.md](pendencias-cliente.md)
- Modelo dimensional: [../modelagem/modelo-dimensional-2026-09-28.md](../modelagem/modelo-dimensional-2026-09-28.md)
- Auditoria Power Query: [../power-query/auditoria-2026-09-28.md](../power-query/auditoria-2026-09-28.md)
