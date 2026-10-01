# Alterações aplicadas — 27/09/2026

Origem: `auditoria-2026-09-27.md`. Backup dos arquivos originais em `docs/power-query/backup-2026-09-27/`.

## Power Query
- Novo parâmetro `caminhoPasta` (grupo Parâmetros) e função `fxLerCsv` (grupo Funções): UTF-8 e `QuoteStyle.Csv` nas 4 fontes
- Queries renomeadas: `campeonato-brasileiro-full` → `fPartidas`, `-gols` → `fGols`, `-cartoes` → `fCartoes`, `-estatisticas-full` → `fEstatisticas`
- Colunas em PascalCase (`rodata` → `Rodada`, `mandante_Placar` → `PlacarMandante`…)
- Conversões com cultura `pt-BR` explícita
- `fPartidas`: arena sem U+00A0 e sem ", Cidade"; `"-"` → `"Empate"`; `Arrecadacao` em moeda; vazios → null
- `fGols` / `fCartoes`: `MinutoTexto` + `Minuto` + `Acrescimo`; `TipoGol` vazio → "Normal"; "Zagueira" → "Zagueiro"; vazios → null
- `fEstatisticas`: `PossePct` e `PrecisaoPassesPct` em percentual; coluna `TemEstatistica`; contagens anuladas nas linhas sem coleta
- Etapas renomeadas e comentadas

## Modelo
- Novas tabelas `dClube` e `dCalendario` (marcada como tabela de datas, com `Temporada`, que corrige 2020/2021)
- Data/hora automática desligada; `LocalDateTable` e `DateTableTemplate` removidas
- Relacionamentos: gols/cartões/estatísticas → `fPartidas` (PartidaId); `fPartidas` → `dCalendario` (Data); gols/cartões/estatísticas → `dClube` (Clube); `fPartidas[Mandante]` e `fPartidas[Visitante]` → `dClube` **inativos** (use `USERELATIONSHIP`)
- IDs, rodada e número da camisa com `summarizeBy: none`

## Observação sobre nomes de arquivo
As tabelas novas foram gravadas nos arquivos antigos (ex.: `fPartidas` dentro de `campeonato-brasileiro-full.tmdl`, `dCalendario` dentro de `LocalDateTable_….tmdl`). O Power BI Desktop renomeia os arquivos no primeiro salvamento.
