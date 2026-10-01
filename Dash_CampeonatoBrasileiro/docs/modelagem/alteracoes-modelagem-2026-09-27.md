# Alterações de modelagem — 27/09/2026

Origem: `revisao-modelagem-2026-09-27.md`. Backup do estado anterior em `docs/modelagem/backup-2026-09-27/`.

## Aplicado
- **Correção:** contagens de estatísticas voltam a ser `Int64` (o substituidor customizado descartava o tipo)
- **Modelo estrela:** 3 fatos × 3 dimensões conformadas, 9 relacionamentos ativos e unidirecionais, sem relacionamento fato→fato
- **`fDesempenho`** (grão: clube × partida, 18.330 linhas) substitui `fPartidas` + `fEstatisticas`: GolsPro, GolsContra, Resultado, Pontos, Mando, Adversario, Formacao, Tecnico, Arrecadacao (só na linha Casa) + estatísticas
- **`dPartida`** (grão: partida): Temporada, Rodada, Hora, Arena, Mandante, Visitante, Confronto, Placar, Vencedor
- **`fGols` e `fCartoes`** ganham `Data` (via `stgPartidas`) e se ligam direto a `dCalendario`; `Rodada` sai dos fatos e fica em `dPartida`
- **Staging** (sem carga no modelo): `stgPartidas`, `stgEstatisticas`
- **Função `fxTemporada`** usada por `dCalendario` e `dPartida` (regra da temporada 2020 em um só lugar)
- Chaves ocultas nos fatos (`PartidaId`, `Data`, `Clube`)
- Relacionamentos inativos Mandante/Visitante removidos

## Não aplicado (decisão)
- Chave inteira `ClubeId`: ganho imperceptível com 46 clubes
- Trimestre/semana no calendário: a análise é por temporada e rodada
- `isAvailableInMdx: false`: micro-otimização irrelevante neste volume

## Validação
- Simulação em Python: 18.330 linhas, 100% das estatísticas casadas, classificações 2020 e 2023 batem com as oficiais
- Estrutura TMDL carregada sem erro pelo motor tabular (6 tabelas, 9 relacionamentos, 5 expressões)

## Custo conhecido
`stgPartidas` alimenta 6 consultas; sem folding em CSV, o arquivo de partidas (1,2 MB) é lido 6 vezes no refresh. Irrelevante neste volume.
