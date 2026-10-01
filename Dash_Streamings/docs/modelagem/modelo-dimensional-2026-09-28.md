# Modelo Dimensional — Dash_Streamings

**Data:** 2026-09-28
**Arquiteto:** revisão via skill `dimensional-modeling`

## Diagrama conceitual

```
              ┌──────────────┐
              │ dCalendario  │  (marcada como tabela de datas)
              └──────┬───────┘
                     │ Data
                     ▼
    ┌────────┐   ┌───────────────────┐   ┌──────────┐
    │dPlatform│──▶│                   │◀──│ dGenre   │
    └────────┘   │                   │   └──────────┘
                 │ fStreamingCatalog │
    ┌────────┐   │  (grão: 1 título) │   ┌──────────┐
    │dCountry│──▶│                   │◀──│ dType    │
    └────────┘   │                   │   └──────────┘
                 └─────────┬─────────┘
                           ▲
                    ┌──────┴──────┐
                    │  dRating    │
                    └─────────────┘
```

Estrela pura, 6 dimensões, 1 fato. Sem snowflake, sem bidirecional, sem M:N.

## Fato

### `fStreamingCatalog`
- **Grão:** 1 linha por título (`show_id` é a chave natural única)
- **Origem:** CSV via M com `caminhoDados` parametrizado, Import
- **Modo:** Import
- **Chaves estrangeiras (ocultas):** `type`, `platform`, `primary_genre`, `country`, `rating`, `date_added`
- **Métricas numéricas (visíveis):**
  - `duration_minutes` (Int64, sum)
  - `num_seasons` (Int64, sum)
  - `num_episodes` (Int64, sum)
  - `imdb_rating` (double, **none** — score não soma)
  - `rotten_tomatoes_score` (double, **none**)
  - `imdb_votes` (Int64, sum)
  - `budget_million_usd` (Int64, sum)
  - `awards_won` (Int64, sum)
  - `hours_watched_million` (double, sum)
- **Atributos descritivos (visíveis):** `title`, `genres`, `director`, `cast`, `language`, `release_year`, `duration`, `production_company`, `description`

## Dimensões

| Tabela | Grão | Chave/Coluna | Origem | Modo |
|---|---|---|---|---|
| `dCalendario` | 1 linha por dia | `Data` (isKey) | `CALENDAR(2008-01-01, 2026-12-31)` | Calculated |
| `dPlatform` | 1 linha por plataforma | `Platform` | `DISTINCT(fStreamingCatalog[platform])` | Calculated |
| `dGenre` | 1 linha por gênero primário | `Genero` | `DISTINCT(fStreamingCatalog[primary_genre])` | Calculated |
| `dCountry` | 1 linha por país | `Pais` | `DISTINCT(fStreamingCatalog[country])` | Calculated |
| `dType` | 1 linha por tipo | `Tipo` (Movie/TV Show) | `DISTINCT(fStreamingCatalog[type])` | Calculated |
| `dRating` | 1 linha por classificação | `Classificacao` | `DISTINCT(fStreamingCatalog[rating])` | Calculated |

**`dCalendario`** traz Ano, NumMes, Mes (sortByColumn NumMes), NumTrimestre, Trimestre, AnoMes, Dia, NumDiaSemana, DiaSemana. `dataCategory: Time` + `isKey` na `Data` — pronta para time intelligence sem depender do auto date/time.

## Relacionamentos (todos 1:N, unidirecionais, ativos)

| De (dimensão) | Para (fato) | Cardinalidade | Direção |
|---|---|---|---|
| `dCalendario.Data` | `fStreamingCatalog.date_added` | 1:N | Single |
| `dPlatform.Platform` | `fStreamingCatalog.platform` | 1:N | Single |
| `dGenre.Genero` | `fStreamingCatalog.primary_genre` | 1:N | Single |
| `dCountry.Pais` | `fStreamingCatalog.country` | 1:N | Single |
| `dType.Tipo` | `fStreamingCatalog.type` | 1:N | Single |
| `dRating.Classificacao` | `fStreamingCatalog.rating` | 1:N | Single |

Nenhum bidirecional, nenhum inativo, nenhum M:N. `release_year` fica como atributo do fato (número), não vira dimensão — se surgir necessidade, criar relacionamento inativo com `dCalendario.Ano` via `USERELATIONSHIP`.

## Decisões de modelagem

1. **Descontinuadas as 4 tabelas summary** (`country_summary`, `genre_summary`, `platform_summary`, `yearly_release_trends`). Continham métricas agregadas que devem ser medidas DAX sobre a fato — manter era duplicação + risco de duas verdades. CSVs permanecem no disco como referência histórica, sem entrar no modelo.

2. **Dimensões via calculated tables (DAX `DISTINCT`)** em vez de CSVs ou queries M próprias:
   - Sempre em sincronia com a fato (novo país no CSV aparece automaticamente).
   - Zero manutenção de arquivos separados.
   - Custo: dependem de `fStreamingCatalog` estar carregada primeiro (fato processada antes das dimensões — Power BI resolve isso sozinho).
   - Trade-off aceito porque as dimensões aqui são pobres em atributos (só o nome). Quando dimensão ganhar atributos descritivos (ex.: `dPlatform` com `Fundacao`, `SedeHQ`, `Assinantes`), migrar para tabela própria em Power Query.

3. **Sem `dReleaseYear`**: `release_year` fica como atributo numérico no fato. Análises temporais fazem sentido pela `date_added` (quando entrou no catálogo). Se surgir a necessidade de comparar "ano de lançamento" com "ano de adição", cria-se relacionamento inativo `dCalendario.Ano ← fStreamingCatalog.release_year` (role-playing) com `USERELATIONSHIP`.

4. **FKs ocultas no fato:** `type`, `platform`, `primary_genre`, `country`, `rating`, `date_added` marcadas `isHidden`. Usuário filtra pelas dimensões correspondentes, garantindo consistência do modelo estrela.

5. **`imdb_rating` e `rotten_tomatoes_score` com `summarizeBy: none`** — scores não somam. Total de score numa tabela não faz sentido; agregações vão via medidas explícitas (`AvgImdb = AVERAGE(...)`).

6. **`dCalendario` marcada como tabela de datas** (`dataCategory: Time` + `isKey` na `Data`). Auto Date/Time já foi desligado no passo anterior. Time intelligence (`TOTALYTD`, `SAMEPERIODLASTYEAR`, `DATEADD`) agora funciona de forma correta e única.

7. **Modo:** tudo Import. Não é DirectQuery nem Direct Lake — volume pequeno, refresh manual, CSV local. Se migrar para Lakehouse do Fabric no futuro, converter `fStreamingCatalog` para `mode: directLake` (as dimensões calculadas continuam Import).

## Próximos passos sugeridos

- **Medidas DAX** — criar tabela `Medidas` com os KPIs que antes vinham hard-coded nas summary:
  - `TotalTitulos = COUNTROWS(fStreamingCatalog)`
  - `AvgImdb = AVERAGE(fStreamingCatalog[imdb_rating])`
  - `AvgRottenTomatoes = AVERAGE(fStreamingCatalog[rotten_tomatoes_score])`
  - `TotalHorasAssistidas = SUM(fStreamingCatalog[hours_watched_million])`
  - `TotalPremios = SUM(fStreamingCatalog[awards_won])`
  - `OrcamentoMedio = AVERAGE(fStreamingCatalog[budget_million_usd])`
  - `TitulosNovosMes = TOTALMTD([TotalTitulos], dCalendario[Data])`
  - `TitulosNovosYTD = TOTALYTD([TotalTitulos], dCalendario[Data])`
  - `VarYoY% = DIVIDE([TotalTitulos] - CALCULATE([TotalTitulos], SAMEPERIODLASTYEAR(dCalendario[Data])), CALCULATE([TotalTitulos], SAMEPERIODLASTYEAR(dCalendario[Data])))`
- **Refinar `dPlatform`** se ganhar atributos (logo, cor da marca, país sede) — migrar de calculated para query M própria.
- **Colunas `cast` e `genres` são strings multivalor** ("Actor A, Actor B, Actor C") — se filtrar por ator/gênero secundário virar requisito, criar tabelas ponte `fCastBridge` e `fGenreBridge` com M:N.
