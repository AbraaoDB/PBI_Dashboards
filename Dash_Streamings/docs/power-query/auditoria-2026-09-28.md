# Auditoria Power Query — Dash_Streamings

**Data:** 2026-09-28
**Escopo:** ETL do projeto PBIP (5 tabelas em modo Import, todas oriundas de CSV local)

## Resumo executivo

- **Queries auditadas:** 5 (`streaming_catalog`, `country_summary`, `genre_summary`, `platform_summary`, `yearly_release_trends`)
- **Modo:** todas Import, fonte `Csv.Document` + `File.Contents`
- **Parâmetros:** nenhum (`expressions.tmdl` inexistente)
- **Funções customizadas:** nenhuma
- **Grupos/pastas:** nenhum
- **Violações:** 🔴 2 críticas · 🟡 7 avisos · 🔵 4 infos
- **Query Folding:** N/A (CSV não folda; nenhuma ganho perdido, mas também nenhuma proteção contra dataset grande)

**Top 3 prioridades:**
1. Eliminar caminho absoluto hard-coded (`C:\Users\braob\...`) — quebra em qualquer outra máquina e no Serviço.
2. Repensar as 4 tabelas `*_summary` — são agregações da própria fato feitas fora do modelo, gerando redundância e relacionamentos circulares AutoDetected. Substituir por medidas DAX.
3. Desativar Auto Date/Time e criar `dCalendario` explícita.

---

## 🔴 Críticos

### 1. Caminho absoluto hard-coded na fonte — todas as 5 queries
**Problema:** todas as partições referenciam literalmente `C:\Users\braob\OneDrive\Área de Trabalho\PBI_Dashboards\Dash_Streamings\Data\<arquivo>.csv`.

**Impacto:** refresh quebra em qualquer outra máquina; publicar no Serviço exige gateway apontando exatamente para esse caminho no Windows do usuário; mover a pasta do OneDrive quebra tudo silenciosamente. Também vaza o usuário Windows (`braob`) em qualquer artefato exportado (BIM, PBIP no Git).

**Correção:** criar dois parâmetros e reescrever a fonte.

Em `Dash_Streamings.SemanticModel/definition/expressions.tmdl` (criar):
```tmdl
expression caminhoDados = "C:\Users\braob\OneDrive\Área de Trabalho\PBI_Dashboards\Dash_Streamings\Data\" meta [IsParameterQuery=true, Type="Text", IsParameterQueryRequired=true]
```

Em cada partição, trocar o `File.Contents(...)`:
```m
// antes
Fonte = Csv.Document(File.Contents("C:\...\Data\streaming_catalog.csv"), [...])

// depois
Fonte = Csv.Document(File.Contents(caminhoDados & "streaming_catalog.csv"), [Delimiter=",", Columns=25, Encoding=65001, QuoteStyle=QuoteStyle.None])
```

Melhor ainda: colocar os CSVs em uma pasta compartilhada (OneDrive Business/SharePoint) e ler via `SharePoint.Files` ou `Folder.Files`, permitindo publicação sem gateway on-premises.

---

### 2. Encoding inconsistente entre arquivos da mesma fonte
**Problema:** `streaming_catalog` usa `Encoding=65001` (UTF-8) e as outras 4 usam `Encoding=1252` (Windows-1252).

**Impacto:** se algum CSV vier com acentos ou emojis (nomes de países, gêneros, títulos), você vai ter *mojibake* silencioso em 4 tabelas ou na fato — sem erro, só dado corrompido. Já vi títulos como "Amélie" virarem "AmÃ©lie".

**Correção:** padronize para UTF-8 (65001) em todos, ou confirme por que uma tabela é diferente. Se o gerador de CSV é o mesmo, o encoding deve ser o mesmo.

---

## 🟡 Avisos

### 3. Tabelas `*_summary` são agregações redundantes da fato
**Problema:** `country_summary`, `genre_summary`, `platform_summary`, `yearly_release_trends` contêm métricas (`total_titles`, `avg_imdb`, `total_hours_watched_million`, etc.) que são simples agregações de `streaming_catalog`. Os relacionamentos gerados são todos **AutoDetected** (`country → country_summary.country`, etc.), criando um modelo em floco de neve invertido onde a fato "aponta" para tabelas que a resumem.

**Impacto:**
- Duplicação de dados (mesma verdade em dois lugares → duas verdades ao longo do tempo).
- Relacionamentos AutoDetected são fonte clássica de bugs (cardinalidade errada, direção de filtro estranha).
- Manutenção: mudar a regra de negócio exige atualizar CSV *e* medida DAX.

**Correção:** manter só `streaming_catalog` como fato. Substituir cada summary por medidas DAX:
```dax
TotalTitles      = COUNTROWS ( streaming_catalog )
AvgImdb          = AVERAGE ( streaming_catalog[imdb_rating] )
TotalHoursWatched = SUM ( streaming_catalog[hours_watched_million] )
```
Filtro por `country`, `primary_genre`, `platform`, `release_year` já vem naturalmente do próprio contexto da fato. Se precisar de dimensões reais, extraia `dCountry`, `dGenre`, `dPlatform` só com a coluna-chave e atributos descritivos (sem métricas).

Elimine também os 4 relationships AutoDetected em `relationships.tmdl` — devem ser criados explicitamente com direção e cardinalidade certas.

---

### 4. Colunas em snake_case (padrão do curso é PascalCase)
**Problema:** `show_id`, `release_year`, `imdb_rating`, `hours_watched_million`, `production_company`... todas snake_case.

**Correção:** renomear no último step de cada query com `Table.RenameColumns`:
```m
#"Colunas Renomeadas" = Table.RenameColumns(#"Tipo Alterado", {
    {"show_id",              "ShowId"},
    {"title",                "Title"},
    {"release_year",         "ReleaseYear"},
    {"imdb_rating",          "ImdbRating"},
    {"hours_watched_million","HoursWatchedMillion"},
    // ...
})
```
Faça isso **antes** de qualquer relacionamento — depois vai obrigar reajuste em todos os visuais.

---

### 5. Tabelas em snake_case sem prefixo semântico
**Problema:** `streaming_catalog` é fato, mas o nome não deixa claro. Nenhum prefixo `f`/`d`.

**Correção (convenção do curso):**
- `streaming_catalog` → `fStreamingCatalog` (fato)
- Se mantiver dimensões: `dCountry`, `dGenre`, `dPlatform`, `dCalendario`

---

### 6. Métricas de score/média tipadas como Int64
**Problema:** `imdb_rating`, `rotten_tomatoes_score`, `avg_imdb`, `avg_rt`, `avg_budget`, `avg_votes` estão como `Int64.Type`. IMDb rating vai de 0 a 10 com uma casa decimal (7.4, 8.2). Rotten Tomatoes idem para % (74.5). Médias por definição têm decimais.

**Impacto:** truncamento silencioso — um IMDb 7.8 vira 7 (ou 8 se arredondar), matando qualquer ranking fino e distorcendo médias no visual.

**Correção:** trocar para `type number` (Double) no `Table.TransformColumnTypes`:
```m
{"imdb_rating", type number},
{"rotten_tomatoes_score", type number},
{"avg_imdb", type number},
// ...
```
E ajuste `summarizeBy` das colunas no TMDL (`sum` → `none` para colunas de score, que não devem ser somadas).

---

### 7. Etapas com nomes default (`Cabeçalhos Promovidos`, `Tipo Alterado`, `Fonte`)
**Problema:** todos os steps vêm da UI sem renomeação. Não é bloqueante, mas dificulta review e debug.

**Correção:** renomear para verbos descritivos:
```m
let
    fonteCsv          = Csv.Document(File.Contents(caminhoDados & "streaming_catalog.csv"), [...]),
    cabecalhosPromovidos = Table.PromoteHeaders(fonteCsv, [PromoteAllScalars=true]),
    tiposAplicados    = Table.TransformColumnTypes(cabecalhosPromovidos, {...}),
    colunasRenomeadas = Table.RenameColumns(tiposAplicados, {...})
in
    colunasRenomeadas
```

---

### 8. Auto Date/Time ativado + sem tabela calendário
**Problema:** `annotation __PBI_TimeIntelligenceEnabled = 1` e existência de `LocalDateTable_36fb94e2-...` e `DateTableTemplate_3c5ad65e-...`. Power BI cria tabelas ocultas de data para cada coluna do tipo date.

**Impacto:**
- Infla o modelo em memória (cada `date` vira uma dim escondida com 1461+ linhas).
- Impede time intelligence consistente entre visuais.
- Convive mal com `dCalendario` real depois.

**Correção:**
1. Desativar em `model.tmdl`: remover `annotation __PBI_TimeIntelligenceEnabled = 1` (ou setar `0`). No Desktop: Arquivo → Opções → Carregamento de Dados → desmarcar Auto Date/Time.
2. Apagar `DateTableTemplate_*.tmdl` e `LocalDateTable_*.tmdl` de `tables/`.
3. Criar `dCalendario` explícita (ver skill `dax` / `dimensional-modeling`).

---

### 9. Sem separação Fonte → Staging → Modelo
**Problema:** cada query lê o CSV e já entrega ao modelo (`Enable Load` implicitamente ligado em todas). Padrão recomendado:
- **Fontes** (privadas): retornam o CSV cru, `Enable Load = false`
- **Staging** (opcional): limpezas comuns, também `Enable Load = false`
- **Modelo**: query final tipada e renomeada, `Enable Load = true`

Não é obrigatório para 5 tabelas pequenas, mas cresce mal. Se o projeto vai receber mais fontes, monte essa espinha agora.

---

## 🔵 Infos

### 10. Sem grupos/pastas para organizar queries
Criar no Power Query Editor: `Parâmetros`, `Fontes`, `Modelo`. Ajuda quando o número de queries passa de ~5.

### 11. Sem comentários no código M
Nenhuma explicação de regra. Não crítico enquanto o M é linear e curto, mas quando aparecer uma coluna calculada com lógica de negócio, comente o **porquê** (não o quê).

### 12. `production_company`, `cast`, `genres` provavelmente são strings multivalor
`cast` e `genres` em datasets estilo Netflix/IMDb costumam ser listas separadas por vírgula. Se for o caso, seria melhor normalizar em tabelas ponte durante o ETL. Não é problema até você tentar filtrar "filmes com o ator X".

### 13. CSV local como fonte definitiva
Para dashboard produtivo, mesmo com parâmetro de caminho, considere:
- Consolidar num Dataflow Gen2 (staging no Fabric)
- Ou promover a um SQL/Lakehouse quando o volume crescer

CSV local + refresh manual funciona no protótipo; morre em produção.

---

## Checklist do que está OK

- ✅ `mode: import` explícito nas partições
- ✅ Tipos definidos logo após promoção de cabeçalhos (não tardio)
- ✅ Sem `Table.Buffer` desnecessário
- ✅ Sem credenciais/tokens no código M
- ✅ Estrutura `let ... in ...` limpa e legível
- ✅ Sem chamadas a funções pesadas (`List.Accumulate`, `List.Generate` M-only) que atrapalhariam performance
- ✅ Modelo pequeno o suficiente para refresh rápido enquanto está em CSV

---

## Ordem sugerida de correção

1. Criar `expressions.tmdl` com `caminhoDados` e parametrizar as 5 fontes (item 1) — 10 min, destrava portabilidade.
2. Padronizar encoding para 65001 (item 2) — 2 min.
3. Trocar tipos de score/média para `type number` (item 6) — 5 min, evita bug silencioso.
4. Desativar Auto Date/Time e apagar as duas tabelas ocultas (item 8) — 5 min, aliviamento imediato.
5. Decisão de arquitetura: manter as 4 tabelas summary ou migrar para medidas DAX (item 3). Se migrar, também vale renomear colunas e tabelas (itens 4 e 5) junto, num único commit.
6. Renomear steps (item 7) e organizar em grupos (item 10) — polish final.
