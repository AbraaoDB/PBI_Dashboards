# Matriz de Rastreabilidade — Dash_Streamings

**Convenção de status:**
- `[VIÁVEL]` — modelo atual já responde
- `[REQUER MODELO]` — falta medida, coluna calculada ou relacionamento
- `[REQUER FONTE]` — dado não existe em nenhuma tabela do projeto (exige uma segunda fonte por serviço; a fonte atual — Streaming Content Catalog, Kaggle — não traz assinantes, ARPU, churn, demografia nem devices)

## Matriz

| Dor | Pergunta de negócio | KPI | Fórmula de negócio | Fonte | Medida DAX | Visual | Página | Prioridade | Status |
|---|---|---|---|---|---|---|---|---|---|
| D1 Comparação manual entre plataformas | Q1 Quantos títulos no catálogo global? | KPI-1 Total de Títulos | Contagem de show_id | fStreamingCatalog | - | Card | Executive Overview | Alta | [REQUER MODELO] |
| D2 Sem visão temporal do catálogo | Q2 Como cresceu nos últimos 5 anos? | KPI-9 Títulos Adicionados no Período | Contagem filtrada por dCalendario | fStreamingCatalog, dCalendario | - | Gráfico de linha (12 meses) | Executive Overview | Alta | [REQUER MODELO] |
| D2 Sem visão temporal do catálogo | Q12 Var % YoY | KPI-10 Var % Títulos YoY | KPI-9 vs SAMEPERIODLASTYEAR | fStreamingCatalog, dCalendario | - | KPI visual (com trend) | Executive Overview | Alta | [REQUER MODELO] |
| D1 Comparação manual | Q3 Qual plataforma domina cada gênero? | KPI-1 (segmentado) | Contagem por Plataforma × Gênero | fStreamingCatalog, dPlatform, dGenre | - | Matriz color-scaled | Platform Battle | Alta | [REQUER MODELO] |
| D3 Qualidade não é medida | Q4 IMDb / RT médio por plataforma | KPI-2 IMDb Médio, KPI-3 RT Médio | AVERAGE(imdb_rating), AVERAGE(rotten_tomatoes_score) | fStreamingCatalog | - | Scatter IMDb × RT (bolha por plataforma) | Platform Battle | Alta | [REQUER MODELO] |
| D4 Orçamento × retorno opaco | Q5 Top 10 títulos por horas assistidas | KPI-4 Horas Assistidas | SUM(hours_watched_million) | fStreamingCatalog | - | Barras horizontais Top N | Efficiency Lab | Alta | [REQUER MODELO] |
| D4 Orçamento × retorno opaco | Q6 Eficiência orçamento → audiência | KPI-6 Horas/US$ | DIVIDE(sum horas, sum orçamento) | fStreamingCatalog | - | Scatter Budget × Hours (cor = eficiência) | Efficiency Lab | Alta | [REQUER MODELO] |
| D3 Qualidade não é medida | Q14 IMDb alto × baixa audiência (joias escondidas) | Combinação KPI-2 + KPI-4 | Filtro IMDb≥8 AND horas no bottom 20% | fStreamingCatalog | - | Tabela + destaque | Efficiency Lab | Média | [REQUER MODELO] |
| D5 Distribuição geográfica opaca | Q7 País de origem por plataforma | KPI-1 por dCountry | Contagem por país | fStreamingCatalog, dCountry | - | Mapa preenchido | Global Map | Alta | [VIÁVEL] |
| D5 Distribuição geográfica opaca | (drill) Títulos por país específico | detalhe | Lista de títulos | fStreamingCatalog, dCountry | - | Tabela drill-through | Global Map (drill) | Média | [VIÁVEL] |
| D1 Comparação manual | Q8 Movie vs TV Show por plataforma | KPI-1 por dType × dPlatform | Contagem cruzada | fStreamingCatalog, dType, dPlatform | - | Barras 100% empilhadas | Platform Battle | Média | [REQUER MODELO] |
| D6 Prêmios ignorados na conversa financeira | Q9 Prêmios × Orçamento | KPI-7 Prêmios, KPI-8 Prêmios/US$ Mi | SUM(awards_won), DIVIDE | fStreamingCatalog | - | Scatter Awards × Budget | Awards & Prestige | Alta | [REQUER MODELO] |
| D3 Qualidade não é medida | Q10 Distribuição de rating (classificação) | KPI-1 por dRating | Contagem por classificação | fStreamingCatalog, dRating | - | Barras horizontais | Platform Battle | Baixa | [VIÁVEL] |
| D2 Sem visão temporal | Q11 Gênero que cresce mais rápido | KPI-9 por dGenre × ano | Contagem por gênero × ano | fStreamingCatalog, dGenre, dCalendario | - | Small multiples (linha por gênero) | Executive Overview | Média | [REQUER MODELO] |
| D3 Qualidade não é medida | Q13 Idade média do catálogo por plataforma | KPI-11 Idade Média | AVERAGEX(YEAR(TODAY) - release_year) | fStreamingCatalog | - | Barras horizontais | Platform Battle | Média | [REQUER MODELO] |
| D3 Qualidade não é medida | Q15 Duração média (filme min / série temporadas) | KPI-12, KPI-13 | AVERAGE filtrado por type | fStreamingCatalog, dType | - | 2 cards | Executive Overview | Baixa | [REQUER MODELO] |
| D5 Distribuição geográfica opaca | Diversidade geográfica | KPI-14 # Países | DISTINCTCOUNT(country) | fStreamingCatalog | - | Card | Executive Overview | Baixa | [REQUER MODELO] |
| D2 Sem visão temporal | Novidade do catálogo | KPI-15 Índice de Novidade | Títulos ≤12m / total | fStreamingCatalog, dCalendario | - | Gauge / KPI | Executive Overview | Média | [REQUER MODELO] |
| — (V2) | Quantos assinantes tem cada plataforma? | KPI-D1 Assinantes | subscribers_millions (segunda fonte, não disponível) | fStreamingServices (a criar) | - | Barras | Subscribers & Growth | Média | [REQUER FONTE] |
| — (V2) | ARPU por plataforma | KPI-D2 ARPU | arpu_usd | fStreamingServices | - | Card + ranking | Subscribers & Growth | Média | [REQUER FONTE] |
| — (V2) | Churn médio mensal | KPI-D3 Churn % | AVERAGE(churn_rate_pct) | fStreamingServices | - | KPI | Subscribers & Growth | Média | [REQUER FONTE] |
| — (V2) | Perfil etário do público por plataforma | KPI-D4 Distrib. Etária | 6 colunas age_group_*_pct | fStreamingServices | - | 100% stacked bars | Audience Demographics | Baixa | [REQUER FONTE] |
| — (V2) | Uso por device | KPI-D5 Distrib. Devices | 6 colunas device_*_pct | fStreamingServices | - | Radar / stacked bar | Audience Demographics | Baixa | [REQUER FONTE] |
| — (V2) | Crescimento de assinantes 2020-2024 | KPI-D6 CAGR Assinantes | (subs_2024 / subs_2020)^(1/4) - 1 | fStreamingServices | - | Linha por plataforma | Subscribers & Growth | Média | [REQUER FONTE] |

## Legenda de leitura
- **Origem em D#** → dor rastreável (seção 1 do levantamento).
- **Origem em Q#** → pergunta rastreável (seção 2).
- **KPI-#** → definição em seção 3 do levantamento.
- Colunas **Medida DAX**, **Visual detalhado** ficam com `-` até serem construídos.
