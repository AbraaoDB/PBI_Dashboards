# Auditoria Power Query — Dash_CampeonatoBrasileiro

Data: 27/09/2026 · Escopo: auditoria completa (nomenclatura, organização, performance, qualidade dos dados) · Modo de armazenamento: **Import** em todas as tabelas

## Resumo executivo

- Queries auditadas: **4** (`campeonato-brasileiro-full`, `-gols`, `-cartoes`, `-estatisticas-full`), todas CSV local com 3 etapas geradas automaticamente
- Total de violações: **18** (🔴 3 / 🟡 11 / 🔵 4)
- Query Folding: **não se aplica** (CSV não tem motor de consulta; todas as etapas rodam localmente, e com ~4 MB de dados o refresh é rápido)
- Top 3 prioridades:
  1. **Estatísticas zeradas contadas como zero de verdade**: 59% das linhas (10.750 de 18.330) são jogos sem coleta, e qualquer média de chutes, passes ou faltas sai muito abaixo do real
  2. **Temporada ≠ ano civil**: a temporada 2020 (pandemia) terminou em fev/2021, então 112 jogos caem no ano errado (2020 mostra 268 jogos, 2021 mostra 492)
  3. **Caminho absoluto do OneDrive fixo nas 4 queries**: quebra quando o projeto muda de pasta ou de máquina, ou quando é publicado

> O relatório ainda não tem visuais (a única página está vazia). **Agora é o momento mais barato para renomear tabelas e colunas**, porque nada no relatório depende desses nomes ainda.

### Inventário

| Query | Linhas | Colunas | Encoding | Etapas | Grupo | Papel real |
|---|---|---|---|---|---|---|
| `campeonato-brasileiro-full` | 9.165 | 17 | 65001 (UTF-8) | Fonte → Cabeçalhos Promovidos → Tipo Alterado | — | Fato de partidas |
| `campeonato-brasileiro-gols` | 10.820 | 6 | 65001 | idem | — | Fato de gols |
| `campeonato-brasileiro-cartoes` | 20.953 | 8 | 65001 | idem | — | Fato de cartões |
| `campeonato-brasileiro-estatisticas-full` | 18.330 | 13 | **1252** | idem | — | Fato de estatísticas por clube e partida |

Parâmetros: nenhum. Funções: nenhuma. Grupos: nenhum. Fontes: 4 × `File.Contents` na mesma pasta local.

---

## 🔴 Críticos

### 1. Zero significando "não coletado": query `campeonato-brasileiro-estatisticas-full`

**Problema:** nas temporadas 2003–2014 e **2024 inteira**, todas as linhas trazem `chutes`, `passes`, `faltas` etc. = 0. Não é zero real, é ausência de coleta. Distribuição de linhas zeradas por ano:

| 2003–2013 | 2014 | 2015–2018 | 2019 | 2020–2023 | 2024 | 2025 |
|---|---|---|---|---|---|---|
| 100% | 96% | 0% | 6% | 0% | **100%** | 0% |

**Impacto:** `AVERAGE(chutes)` sai cerca de 60% abaixo do valor real, rankings de "time que mais finaliza" favorecem quem jogou mais nas temporadas com coleta, e a série histórica mostra 2024 como um ano sem nenhuma finalização.

**Correção:** marcar a linha e anular as contagens quando não houve coleta (código completo na seção [M refatorado](#m-refatorado)):

```m
// Temporadas sem coleta vêm com tudo zerado: aqui zero significa "não medido", não "zero"
__colunasContagem = {"Chutes", "ChutesNoAlvo", "Passes", "Faltas", "CartaoAmarelo", "CartaoVermelho", "Impedimentos", "Escanteios"},
ColetaMarcada = Table.AddColumn(TiposDefinidos, "TemEstatistica",
    each List.Sum(Record.ToList(Record.SelectFields(_, __colunasContagem))) > 0, type logical),
ZerosSemColetaComoNulo = Table.ReplaceValue(ColetaMarcada, each [TemEstatistica], null,
    (valor, temEstatistica, novo) => if temEstatistica then valor else novo, __colunasContagem)
```

Com `null`, `AVERAGE`/`SUM` ignoram a linha, e `TemEstatistica` permite filtrar ou mostrar "sem dados" no visual.

### 2. Temporada tratada como ano civil: query `campeonato-brasileiro-full`

**Problema:** a única dimensão de tempo é a data, e a hierarquia automática agrupa por `YEAR([data])`. O Brasileirão 2020 foi de ago/2020 a fev/2021 por causa da pandemia:

| Ano civil | Jogos | Temporada real |
|---|---|---|
| 2020 | 268 | 380 (268 + 112 de jan–fev/2021) |
| 2021 | 492 | 380 (a partir de mai/2021) |

**Impacto:** qualquer análise "por ano" (campeão, média de gols, arrecadação) fica errada em 2020 e 2021, e sem nenhum erro visível.

**Correção:**

```m
// Temporada 2020 (pandemia) terminou em fev/2021: jogos de jan–abr/2021 pertencem a 2020
TemporadaAdicionada = Table.AddColumn(TiposDefinidos, "Temporada",
    each if Date.Year([Data]) = 2021 and Date.Month([Data]) < 5 then 2020 else Date.Year([Data]),
    Int64.Type)
```

Validado nos dados: com essa regra, 2020 e 2021 ficam com 380 jogos cada. Gols, cartões e estatísticas herdam a temporada pelo relacionamento via `PartidaId`.

### 3. Caminho absoluto fixo: as 4 queries, etapa `Fonte`

**Problema:**
```m
File.Contents("C:\Users\braob\OneDrive\Área de Trabalho\PBI_Dashboards\Dash_CampeonatoBrasileiro\Data\campeonato-brasileiro-cartoes.csv")
```
repetido 4 vezes, apontando para o seu perfil de usuário.

**Impacto:** mover a pasta, abrir em outra máquina ou publicar (exigiria gateway apontando para esse caminho exato) quebra o refresh das 4 tabelas, e o conserto tem que ser feito query por query.

**Correção:** um parâmetro e uma função de leitura (ver [M refatorado](#m-refatorado)):
```m
// Parâmetro (Gerenciar Parâmetros → Texto)
caminhoPasta = "C:\Users\braob\OneDrive\Área de Trabalho\PBI_Dashboards\Dash_CampeonatoBrasileiro\Data\"
```
Se o painel for publicado, o próximo passo natural é mover os CSVs para uma pasta do SharePoint/OneDrive for Business e ler com `SharePoint.Files`, o que dispensa gateway.

---

## 🟡 Avisos

### 4. Percentuais como texto: `estatisticas-full` · `posse_de_bola`, `precisao_passes`
Valores `"35%"`, `"None"` e `""` carregados como texto: não dá para somar, calcular média nem formatar. Correção: converter para `Percentage.Type` com `None`/vazio → `null` (função local `__paraPercentual` no M refatorado).

### 5. Arrecadação como texto: `full` · `arrecadacao`
Números puros (`"220000"`) carregados como texto, e 8.785 dos 9.165 jogos vêm vazios (só há dado a partir de 2025). Correção: `Currency.Type`, com vazio → `null`.

### 6. Nome de arena sujo e inconsistente: `full` · `arena`
- **5.077 linhas** começam com espaço não separável (U+00A0), que o `Text.Trim` padrão **não** remove
- A partir de 2025, o formato muda para `"Mineirão, Belo Horizonte"` (380 linhas), então o mesmo estádio aparece com duas grafias
- Ainda restam variantes históricas: `Castelão`, `Castelão (CE)`, `Arena Castelão`, `Plácido Castelo`

Correção no M: cortar em `,` e remover U+00A0 (feito no refatorado), o que reduz de 191 para 171 arenas distintas. Para as variantes históricas, a solução certa é uma pequena tabela de-para `dArena` (NomeOriginal → NomePadronizado), não mais regras no M.

### 7. Minuto com acréscimo como texto: `gols` e `cartoes` · `minuto`
Valores como `"45+2"` e `"90+14"` impedem faixa de minuto ("gols nos acréscimos", "0–15 min") e ordenação numérica. Correção: manter `MinutoTexto` para exibição e criar `Minuto` (Int) + `Acrescimo` (Int).

### 8. Empate codificado como `"-"`: `full` · `vencedor`
2.421 partidas com `vencedor = "-"`, que aparece como um "clube" na segmentação. Correção: substituir por `"Empate"`.

### 9. Conversão de data dependente da cultura + erros silenciados: `full` · etapa `Tipo Alterado`
`Table.TransformColumnTypes` sem o argumento de cultura depende da localidade do arquivo e da máquina. A data vem como `29/03/2003`, que funciona em pt-BR mas vira erro (ou mês/dia trocado) em uma máquina ou serviço en-US. Além disso, o modelo tem `returnErrorValuesAsNull` ligado, então **datas que falharem viram brancos sem nenhum aviso**. Correção: passar `"pt-BR"` explicitamente no `TransformColumnTypes`.

### 10. Leitura do CSV frágil e inconsistente: as 4 queries, etapa `Fonte`
- `estatisticas-full` usa `Encoding=1252`, e as outras usam 65001. Hoje não quebra porque esse arquivo é ASCII puro, mas no dia em que vier um `"São Paulo"` acentuado, o nome sai corrompido e deixa de casar com as outras tabelas.
- `QuoteStyle.None` faz uma quebra de linha dentro de um campo entre aspas partir a linha. Todos os campos desses arquivos vêm entre aspas, e a coluna `arena` já contém vírgulas. `QuoteStyle.Csv` é o correto para esse formato.
- `Columns=N` fixo: se a fonte ganhar uma coluna, ela é descartada em silêncio.

Correção: uma única `fxLerCsv` com UTF-8 + `QuoteStyle.Csv` para as 4 queries.

### 11. Nomenclatura de queries e colunas
- Queries com o nome do arquivo (`campeonato-brasileiro-full`) e sem prefixo `f`/`d`. Sugestão: `fPartidas`, `fGols`, `fCartoes`, `fEstatisticas`.
- Colunas em snake_case misturado (`mandante_Placar`, `visitante_Estado`) em vez de PascalCase.
- Erro de digitação herdado da fonte: **`rodata`** → `Rodada`.

### 12. Sem grupos, parâmetros ou função: lógica de leitura duplicada 4×
Todas as queries estão na raiz, e a leitura do CSV (Fonte + cabeçalho) está copiada 4 vezes. Grupos sugeridos: `Parâmetros` (caminhoPasta), `Funções` (fxLerCsv), `Fatos`, `Dimensões`.

### 13. Texto vazio `""` em vez de `null`
Colunas afetadas: `posicao` (1.198), `tipo_de_gol` (9.527), `num_camisa` (386), `atleta` (6), formação e técnico (~4.600–5.000 cada). Em CSV, vazio chega como `""`, não `null`, e isso faz `COUNT`/`DISTINCTCOUNT` contarem o vazio como valor e a segmentação mostrar um item em branco que não é o `(Em branco)` real. Correção: `Table.ReplaceValue("", null, ...)` antes da tipagem. Em `tipo_de_gol`, vazio significa gol normal, então vira `"Normal"`.

### 14. Categoria duplicada: `cartoes` · `posicao`
`"Zagueira"` (22 linhas) ao lado de `"Zagueiro"` (7.460). Correção: substituir por `"Zagueiro"`.

---

## 🔵 Infos

### 15. Etapas com nome default
`#"Cabeçalhos Promovidos"` e `#"Tipo Alterado"` em todas as queries. Com 3 etapas não chega a atrapalhar, mas com a refatoração as etapas passam a ter nomes descritivos (`ColunasRenomeadas`, `ArenaLimpa`, `TemporadaAdicionada`…).

### 16. Nenhum comentário
Regras não óbvias (temporada 2020, zero = sem coleta, `"-"` = empate) precisam de `//` explicando o porquê. O M refatorado já inclui esses comentários.

### 17. Detecção automática de tipo ligada
Em `editorSettings.json`, `typeDetectionEnabled: true` gera o `Tipo Alterado` a partir das primeiras 1.000 linhas. Foi assim que `arrecadacao` virou texto (as primeiras 1.000 linhas estão vazias). Vale desligar em *Opções → Arquivo atual → Carregamento de dados* e tipar manualmente.

### 18. Privacy Level
Há uma única fonte (arquivos locais), então o risco é baixo. Defina como **Organizacional** para evitar avisos se uma fonte web ou SharePoint entrar depois.

---

## Fora do escopo do M (encaminhar para modelagem)

Esses achados são de **modelo**, não de query. Registro aqui porque afetam diretamente o que o ETL deve entregar:

- **Nenhum relacionamento entre as tabelas.** O único relacionamento existente é o automático de data. Gols, cartões e estatísticas precisam de `PartidaId → fPartidas[PartidaId]`. A integridade é de 100%: todos os `partida_id` existem em `full`.
- **Falta `dClube`**: hoje, filtrar por clube em partidas (mandante/visitante) e em gols/cartões exige duas colunas diferentes. O M da dimensão está abaixo.
- **Tabela de data automática** (`LocalDateTable_…`) em vez de um `dCalendario` marcado. Desligue *Data/hora automática* e crie a calendário com a coluna `Temporada`.
- **IDs somáveis:** `partida_id`, `rodata` e `num_camisa` estão com `summarizeBy: sum`, então devem virar `none`.

Para esses pontos: skills `dimensional-modeling` e `power-bi-best-practices`.

---

## M refatorado

Ordem de criação: parâmetro → função → fatos → dimensão. Os nomes novos quebram só referências do relatório, e hoje não há nenhuma.

### Parâmetro `caminhoPasta` (grupo Parâmetros)

Em TMDL (`expressions.tmdl`):
```tmdl
expression caminhoPasta = "C:\Users\braob\OneDrive\Área de Trabalho\PBI_Dashboards\Dash_CampeonatoBrasileiro\Data\" meta [IsParameterQuery=true, Type="Text", IsParameterQueryRequired=true]
```

### `fxLerCsv` (grupo Funções, Enable Load desmarcado)
```m
// Lê um CSV da pasta do projeto: UTF-8, aspas respeitadas, cabeçalho promovido.
// Centraliza encoding e QuoteStyle para as 4 fontes ficarem iguais.
(nomeArquivo as text) as table =>
let
    Fonte = Csv.Document(
        File.Contents(caminhoPasta & nomeArquivo),
        [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
    ),
    CabecalhosPromovidos = Table.PromoteHeaders(Fonte, [PromoteAllScalars = true])
in
    CabecalhosPromovidos
```

### `fPartidas` (grupo Fatos)
```m
let
    Fonte = fxLerCsv("campeonato-brasileiro-full.csv"),
    ColunasRenomeadas = Table.RenameColumns(Fonte, {
        {"ID", "PartidaId"}, {"rodata", "Rodada"}, {"data", "Data"}, {"hora", "Hora"},
        {"mandante", "Mandante"}, {"visitante", "Visitante"},
        {"formacao_mandante", "FormacaoMandante"}, {"formacao_visitante", "FormacaoVisitante"},
        {"tecnico_mandante", "TecnicoMandante"}, {"tecnico_visitante", "TecnicoVisitante"},
        {"vencedor", "Vencedor"}, {"arena", "Arena"},
        {"mandante_Placar", "PlacarMandante"}, {"visitante_Placar", "PlacarVisitante"},
        {"mandante_Estado", "EstadoMandante"}, {"visitante_Estado", "EstadoVisitante"},
        {"arrecadacao", "Arrecadacao"}
    }),
    // CSV entrega vazio como "": vira null para não contar como valor
    VaziosComoNulo = Table.ReplaceValue(ColunasRenomeadas, "", null, Replacer.ReplaceValue,
        Table.ColumnNames(ColunasRenomeadas)),
    // Arena: descarta o ", Cidade" (formato de 2025) e o espaço não separável U+00A0 do início
    ArenaLimpa = Table.TransformColumns(VaziosComoNulo, {{"Arena",
        each if _ = null then null
             else Text.Trim(Text.Split(_, ","){0}, {" ", Character.FromNumber(160)}),
        type text}}),
    // Na fonte, "-" significa empate
    EmpatePadronizado = Table.ReplaceValue(ArenaLimpa, "-", "Empate", Replacer.ReplaceValue, {"Vencedor"}),
    // Cultura explícita: a data vem dd/MM/yyyy e não pode depender da máquina que faz o refresh
    TiposDefinidos = Table.TransformColumnTypes(EmpatePadronizado, {
        {"PartidaId", Int64.Type}, {"Rodada", Int64.Type}, {"Data", type date}, {"Hora", type time},
        {"Mandante", type text}, {"Visitante", type text},
        {"FormacaoMandante", type text}, {"FormacaoVisitante", type text},
        {"TecnicoMandante", type text}, {"TecnicoVisitante", type text},
        {"Vencedor", type text}, {"Arena", type text},
        {"PlacarMandante", Int64.Type}, {"PlacarVisitante", Int64.Type},
        {"EstadoMandante", type text}, {"EstadoVisitante", type text},
        {"Arrecadacao", Currency.Type}
    }, "pt-BR"),
    // Temporada 2020 (pandemia) terminou em fev/2021: jogos de jan–abr/2021 pertencem a 2020
    TemporadaAdicionada = Table.AddColumn(TiposDefinidos, "Temporada",
        each if Date.Year([Data]) = 2021 and Date.Month([Data]) < 5 then 2020 else Date.Year([Data]),
        Int64.Type)
in
    TemporadaAdicionada
```

### `fGols` (grupo Fatos)
```m
let
    Fonte = fxLerCsv("campeonato-brasileiro-gols.csv"),
    ColunasRenomeadas = Table.RenameColumns(Fonte, {
        {"partida_id", "PartidaId"}, {"rodata", "Rodada"}, {"clube", "Clube"},
        {"atleta", "Atleta"}, {"minuto", "MinutoTexto"}, {"tipo_de_gol", "TipoGol"}
    }),
    // Tipo vazio = gol comum (só Penalty e Gol Contra vêm preenchidos)
    TipoGolPadronizado = Table.ReplaceValue(ColunasRenomeadas, "", "Normal", Replacer.ReplaceValue, {"TipoGol"}),
    TiposDefinidos = Table.TransformColumnTypes(TipoGolPadronizado, {
        {"PartidaId", Int64.Type}, {"Rodada", Int64.Type}, {"Clube", type text},
        {"Atleta", type text}, {"MinutoTexto", type text}, {"TipoGol", type text}
    }, "pt-BR"),
    // "45+2" = minuto 45 com 2 de acréscimo; separado para permitir faixa de minuto e ordenação
    MinutoAdicionado = Table.AddColumn(TiposDefinidos, "Minuto",
        each Number.From(Text.Split([MinutoTexto], "+"){0}), Int64.Type),
    AcrescimoAdicionado = Table.AddColumn(MinutoAdicionado, "Acrescimo",
        each Number.From(Text.Split([MinutoTexto], "+"){1}? ?? "0"), Int64.Type)
in
    AcrescimoAdicionado
```

### `fCartoes` (grupo Fatos)
```m
let
    Fonte = fxLerCsv("campeonato-brasileiro-cartoes.csv"),
    ColunasRenomeadas = Table.RenameColumns(Fonte, {
        {"partida_id", "PartidaId"}, {"rodata", "Rodada"}, {"clube", "Clube"}, {"cartao", "Cartao"},
        {"atleta", "Atleta"}, {"num_camisa", "NumeroCamisa"}, {"posicao", "Posicao"}, {"minuto", "MinutoTexto"}
    }),
    VaziosComoNulo = Table.ReplaceValue(ColunasRenomeadas, "", null, Replacer.ReplaceValue,
        {"Atleta", "NumeroCamisa", "Posicao"}),
    // Fonte grafa "Zagueira" em 22 linhas: mesma posição
    PosicaoPadronizada = Table.ReplaceValue(VaziosComoNulo, "Zagueira", "Zagueiro", Replacer.ReplaceValue, {"Posicao"}),
    TiposDefinidos = Table.TransformColumnTypes(PosicaoPadronizada, {
        {"PartidaId", Int64.Type}, {"Rodada", Int64.Type}, {"Clube", type text}, {"Cartao", type text},
        {"Atleta", type text}, {"NumeroCamisa", Int64.Type}, {"Posicao", type text}, {"MinutoTexto", type text}
    }, "pt-BR"),
    // "90+3" = minuto 90 com 3 de acréscimo
    MinutoAdicionado = Table.AddColumn(TiposDefinidos, "Minuto",
        each Number.From(Text.Split([MinutoTexto], "+"){0}), Int64.Type),
    AcrescimoAdicionado = Table.AddColumn(MinutoAdicionado, "Acrescimo",
        each Number.From(Text.Split([MinutoTexto], "+"){1}? ?? "0"), Int64.Type)
in
    AcrescimoAdicionado
```

### `fEstatisticas` (grupo Fatos)
```m
let
    Fonte = fxLerCsv("campeonato-brasileiro-estatisticas-full.csv"),
    ColunasRenomeadas = Table.RenameColumns(Fonte, {
        {"partida_id", "PartidaId"}, {"rodata", "Rodada"}, {"clube", "Clube"},
        {"chutes", "Chutes"}, {"chutes_no_alvo", "ChutesNoAlvo"},
        {"posse_de_bola", "PossePct"}, {"passes", "Passes"}, {"precisao_passes", "PrecisaoPassesPct"},
        {"faltas", "Faltas"}, {"cartao_amarelo", "CartaoAmarelo"}, {"cartao_vermelho", "CartaoVermelho"},
        {"impedimentos", "Impedimentos"}, {"escanteios", "Escanteios"}
    }),
    // "35%" → 0,35 ; "None" e "" = não medido
    __paraPercentual = (t as nullable text) as nullable number =>
        if t = null or t = "" or t = "None" then null
        else Number.From(Text.Remove(t, "%")) / 100,
    PercentuaisConvertidos = Table.TransformColumns(ColunasRenomeadas, {
        {"PossePct", __paraPercentual, Percentage.Type},
        {"PrecisaoPassesPct", __paraPercentual, Percentage.Type}
    }),
    TiposDefinidos = Table.TransformColumnTypes(PercentuaisConvertidos, {
        {"PartidaId", Int64.Type}, {"Rodada", Int64.Type}, {"Clube", type text},
        {"Chutes", Int64.Type}, {"ChutesNoAlvo", Int64.Type}, {"Passes", Int64.Type},
        {"Faltas", Int64.Type}, {"CartaoAmarelo", Int64.Type}, {"CartaoVermelho", Int64.Type},
        {"Impedimentos", Int64.Type}, {"Escanteios", Int64.Type}
    }, "pt-BR"),
    // Temporadas sem coleta (2003–2014, 2024) vêm com tudo zerado: aqui zero = "não medido"
    __colunasContagem = {"Chutes", "ChutesNoAlvo", "Passes", "Faltas",
                         "CartaoAmarelo", "CartaoVermelho", "Impedimentos", "Escanteios"},
    ColetaMarcada = Table.AddColumn(TiposDefinidos, "TemEstatistica",
        each List.Sum(Record.ToList(Record.SelectFields(_, __colunasContagem))) > 0, type logical),
    ZerosSemColetaComoNulo = Table.ReplaceValue(ColetaMarcada, each [TemEstatistica], null,
        (valor, temEstatistica, novo) => if temEstatistica then valor else novo, __colunasContagem)
in
    ZerosSemColetaComoNulo
```

### `dClube` (grupo Dimensões)
```m
// Todo clube foi mandante ao menos uma vez: basta a lista de mandantes com o estado
let
    Fonte = fPartidas,
    ColunasSelecionadas = Table.SelectColumns(Fonte, {"Mandante", "EstadoMandante"}),
    ColunasRenomeadas = Table.RenameColumns(ColunasSelecionadas, {{"Mandante", "Clube"}, {"EstadoMandante", "Estado"}}),
    ClubesUnicos = Table.Distinct(ColunasRenomeadas, {"Clube"})
in
    ClubesUnicos
```
*Nota de custo:* sem folding, `dClube` relê o CSV de partidas no refresh. Com 1,2 MB o custo é irrelevante.

---

## Checklist de boas práticas atendidas

- ✅ Modo Import em todas as tabelas, então nenhuma restrição de DirectQuery/Dual se aplica
- ✅ Tipos definidos logo após promover cabeçalho (cedo, não no fim)
- ✅ Nenhum `Table.Buffer`, `List.Accumulate` ou transformação pesada
- ✅ Nenhuma credencial ou token no código
- ✅ **Integridade referencial de 100%**: todo `partida_id` de gols, cartões e estatísticas existe em partidas
- ✅ **Nomes de clube idênticos** nas 4 tabelas (46 clubes, sem variação de grafia)
- ✅ **Gols batem com o placar** em 100% das 4.172 partidas que têm gols registrados
- ✅ Sem linhas duplicadas; `rodada` consistente entre gols e partidas
- ✅ Nenhum CSV contém quebra de linha dentro de campo (hoje `QuoteStyle.None` não corrompe dados, só é frágil)
