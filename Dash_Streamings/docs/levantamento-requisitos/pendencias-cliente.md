# Pendências para o cliente — Dash_Streamings

Cada item precisa de resposta objetiva antes de fechar o escopo da V1. Ordem: primeiro os que travam decisão de arquitetura, depois os operacionais.

---

## P1 · Fonte dos dados — ✅ RESOLVIDA (2026-09-30)

**Decisão:** a fonte do projeto é o dataset Kaggle [Streaming Content Catalog: Netflix, Prime, Disney+](https://www.kaggle.com/datasets/meruvakodandasuraj/streaming-content-catalog-netflix-prime-disney?select=yearly_release_trends.csv) (Meruva Kodanda Suraj), que contém exatamente os 5 CSVs de `Data/` (catálogo de 15.000 títulos + 4 resumos). É uma base **sintética** para fins educacionais.

**Consequência:** os KPIs D1–D6 (assinantes, ARPU, churn, demografia, devices, crescimento de assinantes) **saem do escopo**, porque esses dados não existem nesta fonte. Só voltam se uma segunda fonte por serviço for aprovada (candidata: dataset Kaggle "Global Streaming Services", citado na versão anterior deste documento).

---

## P2 · Regra R13: título com IMDb nulo/zero entra no cálculo de média?

**Contexto:** hoje o modelo não trata. Um título com IMDb = 0 puxa a média para baixo. Título com null é ignorado pelo `AVERAGE`, mas se o CSV tiver zero como sentinela isso vira ruído.

**Pergunta:** IMDb = 0 significa "sem nota atribuída" (excluir do cálculo) ou "efetivamente zero" (incluir)?

- [ ] Excluir zero — tratar como sem nota.
- [ ] Incluir zero — refletir catálogo bruto.
- [ ] Depende do gênero/tipo — descrever.

---

## P3 · Regra R14: qual é a "janela de novidade" para o KPI-15?

**Contexto:** KPI-15 (Índice de Novidade) mede a fatia do catálogo adicionada nos últimos N meses. Default proposto: 12 meses.

**Pergunta:** 12 meses serve para todas as plataformas ou você prefere 6 / 24?

- [ ] 6 meses (agressivo — só o "quente").
- [ ] 12 meses (padrão).
- [ ] 24 meses (conservador).
- [ ] Configurável via parâmetro what-if.

---

## P4 · Coluna `country` do CSV parece multivalor ("US, UK") — como tratar?

**Contexto:** ainda não validamos. Se a coluna vier como "US, UK, France", cada linha aumenta a cardinalidade de `dCountry` e polui análises geográficas.

**Pergunta:** um título produzido em vários países deve:

- [ ] Aparecer no país **principal** (primeiro da lista) — mais simples, perde nuance.
- [ ] Aparecer em **todos os países** via bridge (`bTituloPais`) — mais rico, mais complexo.
- [ ] Manter como texto e não filtrar por país (tratar `country` como atributo descritivo).

*Cliente decide após ver amostra dos valores presentes — vamos preparar diagnóstico.*

---

## P5 · Coluna `genres` (multivalor) — mesma decisão de P4

**Contexto:** `genres` provavelmente lista todos os gêneros ("Drama, Thriller, Sci-Fi"). Hoje o modelo usa `primary_genre` (single) — funciona, mas descarta gêneros secundários.

**Pergunta:** você quer conseguir filtrar por gênero secundário (ex.: "todos os títulos que tocam Thriller mesmo que primário seja Drama")?

- [ ] Sim → criar bridge `bTituloGenero` (M:N com filtro bidirecional controlado).
- [ ] Não → seguir com `primary_genre` apenas.

---

## P6 · Slicer de período — default de "últimos 24 meses" está ok?

**Contexto:** proposta na seção 8 do levantamento.

**Pergunta:** ao abrir o relatório, o período visível padrão deve ser:

- [ ] Últimos 24 meses (proposto).
- [ ] Últimos 12 meses.
- [ ] Ano corrente (YTD).
- [ ] Todo o histórico.

---

## P7 · Frequência de atualização real dos dados

**Contexto:** os CSVs no projeto são o snapshot publicado no Kaggle (base sintética); não há atualização automática.

**Pergunta:** o cliente vai:

- [ ] Fazer download manual mensal do Kaggle e substituir o CSV.
- [ ] Automatizar via API/scraping (fora do escopo do dashboard).
- [ ] Considerar o snapshot atual como "fotografia fechada" (dashboard estático).

Isso decide se vale investir em `refreshPolicy` (Incremental Refresh) ou não.

---

## P8 · Publicação e permissão

**Pergunta:**
- [ ] Fica em PBIP local (portfólio / apresentação).
- [ ] Publica em workspace Fabric — qual?
- [ ] Compartilha com equipe — quem? (RLS por região necessária?)

---

## P9 · Identidade visual

**Pergunta:** existe brand book / paleta / tipografia que o dashboard deve seguir?

- [ ] Sim — anexar link ou arquivo.
- [ ] Não — usamos tema neutro dark (streaming-friendly, seguindo skill `dataviz-html-dax`).

---

## P10 · Título do relatório

**Pergunta:** o nome público do dashboard será:

- [ ] "Dash Streamings" (nome atual do projeto).
- [ ] "Streaming Catalog Intelligence".
- [ ] Outro — sugestão do cliente.

---

## Prioridade das respostas

- **Bloqueiam V1:** P4, P5, P6 (afetam modelagem e UX imediatamente).
- **Resolvida:** P1 (fonte confirmada; KPIs de assinantes fora do escopo).
- **Ajustes finais:** P2, P3, P8, P9, P10.
- **Operacional:** P7.
