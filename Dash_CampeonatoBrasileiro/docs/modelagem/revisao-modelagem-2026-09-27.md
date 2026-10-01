# Revisão de modelagem dimensional — Dash_CampeonatoBrasileiro

Data: 27/09/2026 · Base: TMDL salvo pelo Desktop após as correções de ETL · Modo: Import (CSV local, correto para a fonte)

## Veredito

**O modelo ainda não é um modelo estrela.** As dimensões básicas estão corretas: o calendário está marcado, a data/hora automática está desligada, as chaves são únicas, a integridade é de 100% e todos os filtros vão num sentido só. O problema é que `fPartidas` funciona ao mesmo tempo como fato e como dimensão das outras três tabelas, e guarda os dois clubes em colunas separadas (Mandante/Visitante). Isso gera relacionamentos fato→fato e deixa `dClube` sem conseguir filtrar as partidas.

| Severidade | Qtde |
|---|---|
| 🔴 Crítico | 3 |
| 🟡 Aviso | 4 |
| 🔵 Info | 3 |

---

## 🔴 Críticos

### 1. Contagens de `fEstatisticas` salvas como texto (erro da correção anterior)
`Chutes`, `ChutesNoAlvo`, `Passes`, `Faltas`, `CartaoAmarelo`, `CartaoVermelho`, `Impedimentos` e `Escanteios` estão com `dataType: string`. A etapa que anula os zeros (`Table.ReplaceValue` com substituidor customizado) descarta o tipo das colunas, e o Desktop as carregou como texto. Com isso, nenhuma dessas colunas pode ser somada.
**Correção:** retipar as colunas como `Int64` depois dessa etapa.

### 2. Relacionamentos fato→fato
`fGols`, `fCartoes` e `fEstatisticas` se relacionam com `fPartidas`, e só `fPartidas` se relaciona com `dCalendario`. Os fatos não se ligam ao calendário: o filtro de data passa por outro fato. `fPartidas` mistura atributos de dimensão (arena, técnico, formação, hora) com métricas (placar, arrecadação).
**Correção:** separar os atributos da partida em `dPartida` e dar a cada fato a sua própria `Data`, ligada direto a `dCalendario`.

### 3. Clube em papel duplo (Mandante/Visitante) em `fPartidas`
Com dois clubes por linha, `dClube` não consegue filtrar partidas: as duas relações estão inativas, e nenhuma medida usa `USERELATIONSHIP`, então são **relacionamentos inativos órfãos**. Perguntas básicas de futebol, como pontos, vitórias, gols pró/contra e aproveitamento por clube, exigiriam uma medida com `OR` entre as duas colunas em cada cálculo.
**Correção:** mudar o grão para **clube × partida** (2 linhas por jogo). Esse é exatamente o grão de `fEstatisticas`, com 2 linhas por partida em 100% dos 9.165 jogos e os clubes sempre batendo com mandante/visitante. As duas tabelas se fundem em um único fato, `fDesempenho`.

---

## 🟡 Avisos

### 4. Chave de clube em texto
`Clube` (string) é a chave de 3 relacionamentos. A regra do modelo é chave `Int64`. **Correção:** `dClube[ClubeId]` inteiro e `ClubeId` nos fatos.

### 5. Chaves estrangeiras e colunas técnicas visíveis nos fatos
`PartidaId`, `Clube`, `Data` e `Rodada` aparecem em todos os fatos. O usuário deveria filtrar pelas dimensões. **Correção:** ocultar as chaves nos fatos, levar `Rodada` para `dPartida` e usar `isAvailableInMdx: false` nas colunas ocultas.

### 6. `dCalendario` incompleta
Faltam Trimestre, Semana, Dia e Mês/Ano ordenável. **Correção:** acrescentar `Dia`, `Trimestre`, `SemanaISO`, `AnoMesNumero` e `MesAno` (com `sortByColumn`).

### 7. `Temporada` só no calendário, sem `Rodada` junto
Análises por rodada ("classificação na rodada 19") precisam de temporada + rodada na mesma dimensão. **Correção:** `dPartida` com `Temporada` e `Rodada`, mantendo `Temporada` também no calendário.

---

## 🔵 Infos

### 8. Atleta sem dimensão própria
A fonte não tem ID de atleta, só o nome (1.636 nos gols, 2.291 nos cartões). Criar `dAtleta` por nome arrisca juntar homônimos. **Decisão:** manter `Atleta` como atributo degenerado em `fGols`/`fCartoes`.

### 9. Sem tabela `Medidas`
Ainda não há nenhuma medida. Esse é o próximo passo depois do modelo (skill `dax`).

### 10. Arena, técnico e formação
Não viram dimensões próprias: arena vai como atributo de `dPartida`, técnico e formação como atributos do clube na partida (em `fDesempenho`).

---

## Modelo proposto

```
   dCalendario ──┐          ┌── dPartida
                 ├─► fDesempenho ◄─┤
   dClube ───────┤          │
                 ├─► fGols ◄───────┤
                 └─► fCartoes ◄────┘
   (dCalendario, dClube e dPartida filtram os três fatos)
```

Todas as relações são muitos→um, com filtro da dimensão para o fato e ativas. **Três dimensões conformadas** (`dCalendario`, `dClube`, `dPartida`) servem aos **três fatos**.

### Grão declarado

| Tabela | Tipo | Uma linha representa | Linhas |
|---|---|---|---|
| `fDesempenho` | Fato | um clube em uma partida | 18.330 |
| `fGols` | Fato | um gol | 10.820 |
| `fCartoes` | Fato | um cartão | 20.953 |
| `dPartida` | Dimensão | uma partida | 9.165 |
| `dClube` | Dimensão | um clube | 46 |
| `dCalendario` | Dimensão | um dia (jan/2003–dez/2025) | 8.401 |

### `fDesempenho` (substitui `fPartidas` + `fEstatisticas`)
PartidaId · Data · ClubeId · AdversarioId (sem relacionamento, só atributo) · Mando (Casa/Fora) · GolsPro · GolsContra · Resultado (Vitória/Empate/Derrota) · Pontos (3/1/0) · Formacao · Tecnico · Arrecadacao (só na linha Casa) · Chutes … Escanteios · PossePct · PrecisaoPassesPct · TemEstatistica

Assim, a classificação sai direto com `SUM(fDesempenho[Pontos])` por `dClube`, sem `USERELATIONSHIP`.

### Relacionamentos

| De (muitos) | Para (um) | Ativo |
|---|---|---|
| fDesempenho[PartidaId] | dPartida[PartidaId] | sim |
| fDesempenho[Data] | dCalendario[Data] | sim |
| fDesempenho[ClubeId] | dClube[ClubeId] | sim |
| fGols[PartidaId] | dPartida[PartidaId] | sim |
| fGols[Data] | dCalendario[Data] | sim |
| fGols[ClubeId] | dClube[ClubeId] | sim |
| fCartoes[PartidaId] | dPartida[PartidaId] | sim |
| fCartoes[Data] | dCalendario[Data] | sim |
| fCartoes[ClubeId] | dClube[ClubeId] | sim |

0 bidirecionais, 0 muitos-para-muitos, 0 inativos.

### Decisões registradas
- **Import:** a fonte é CSV local, sem OneLake. Direct Lake e DirectQuery não se aplicam.
- **Gol contra:** em `fGols[Clube]` fica o clube **beneficiado**. Os gols por clube batem com o placar de cada lado em 100% das 6.326 combinações clube×partida.
- **`AdversarioId` sem relacionamento:** um segundo caminho para `dClube` seria ambíguo. Serve como coluna de rótulo ou filtro via `TREATAS` quando for preciso.
- **Atleta degenerado:** a fonte não tem ID de atleta (item 8).

---

## Checklist atendido

- ✅ Calendário próprio, contínuo, marcado (`dataCategory: Time` + `isKey`)
- ✅ Data/hora automática desligada (sem `LocalDateTable`)
- ✅ Chaves únicas no lado um (PartidaId 9.165/9.165; Clube 46/46)
- ✅ Integridade referencial de 100% (todo PartidaId e Clube dos fatos existe nas dimensões)
- ✅ Direção de filtro única, sem muitos-para-muitos nem bidirecional
- ✅ `summarizeBy: none` em IDs, rodada e camisa; texto temporal com `sortByColumn`
- ✅ Moeda em `decimal`; Data e Hora já separadas
- ✅ Nomenclatura `f`/`d` e PascalCase
