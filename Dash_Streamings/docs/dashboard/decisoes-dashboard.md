# Dashboard StreamIQ — decisões de design e operação

**Data:** 2026-09-28 · **Relatório:** `Dash_Streamings.Report` (8 páginas, 1920 × 1080)

## Páginas

| # | Página (`pageId`) | Macroestrutura | Pergunta que responde |
|---|---|---|---|
| 0 | Capa (`pgCapa`) | Landing | O que é o painel e por onde começar |
| 1 | Visão Executiva (`pgExecutiva`) | Prioridade executiva | Como está o catálogo hoje? |
| 2 | Plataformas (`pgPlataformas`) | Comparação | Quem domina — e com que qualidade? |
| 3 | Qualidade & Eficiência (`pgEficiencia`) | Exceções + ranking | Onde o orçamento vira audiência? |
| 4 | Mapa Global (`pgGlobal`) | Distribuição geográfica | De onde vem o conteúdo? |
| 5 | Prêmios & Prestígio (`pgPremios`) | Comparação + correlação | Quem converte catálogo em prêmio? |
| 6 | Tendências (`pgTendencias`) | Tendência | Para onde o catálogo está indo? |
| 7 | Catálogo (`pgCatalogo`) | Investigação detalhada | Qual é o título X? (tabela nativa, exportável) |

## Tecnologia escolhida (e por quê)

- **HTML Content (edição lite, certificada)** para painéis ricos — GUID `htmlContent443BE3AD55E043BF878BED274D3A6865`,
  origem AppSource (`coacervolimited1596856650797.htmlcontent_certified`), registrado em `report.json → publicCustomVisuals`.
  Justificativa: o layout pedido (cartograma, matriz de especialização, áreas empilhadas por plataforma, hall da fama com
  troféus, capa com parede de plataformas) não é alcançável com visuais nativos. A lite foi escolhida por **exportar para
  PDF/PPT** e passar em tenants que só aceitam visuais certificados.
- **Visuais nativos** onde a interatividade importa: 4 slicers sincronizados (Plataforma, Tipo, Gênero, Ano de entrada),
  busca por título, tabela de investigação e botões de navegação transparentes sobre o menu desenhado em HTML.
- **Filtro cruzado a partir do HTML** (papel *Granularity*): cartões de plataforma, ranking de eficiência por gênero e ranking
  de países filtram as demais visualizações da página ao clique.
- **Sem JavaScript**: animações só em CSS (respeitam `prefers-reduced-motion`), tooltips por CSS `:hover`, realce de camadas
  por `:hover` em SVG.
- **Tom visual**: escuro com violeta como cor de marca; cores de plataforma são identidade (medida `Cor Plataforma`),
  todas as cores vêm das medidas em `91. Cores` (nenhum hex solto nas medidas de página).

## Correções de dados feitas durante a construção

1. **Cultura de leitura do CSV**: o modelo pt-BR lia `6.3` como `63` (IMDb médio aparecia 68,1). A etapa de tipos agora usa
   `"en-US"`; `budget_million_usd` passou a decimal.
2. **Calendário**: `dCalendario` começava em 2008 e deixava 2.646 títulos (18%) fora de qualquer análise temporal; agora
   cobre 1980–2026.
3. **Índice de Novidade**: o denominador ignorava todos os filtros (valor não mudava ao filtrar plataforma); agora ignora
   só o filtro de período.

## Pendências de ambiente (para o usuário)

- **1ª abertura baixa o visual HTML Content do AppSource** — exige internet e AppSource liberado no tenant. Sem isso, os
  painéis mostram "Can't display this visual".
- **Sem fixação de versão**: o AppSource atualiza o visual automaticamente; revalidar após atualizações.
- **Mobile**: layout desenhado para 1920 × 1080; o comportamento do HTML Content no app móvel não é documentado — testar.
- **Leitor de tela**: blocos HTML não expõem estrutura acessível; a página **Catálogo** (tabela nativa) é a alternativa
  com os mesmos números.
- **Dados sintéticos**: a base do Kaggle é sintética; distribuições muito uniformes (ex.: ~60% dos títulos fora dos EUA
  em todas as plataformas) foram tratadas como tal e não viraram insight.

## Como regenerar

Os fontes das medidas (`m*.dax`), o layout (`pages.py`) e o gerador (`gen_pbir.py`) ficam fora do projeto; o gerador é
idempotente (substitui medidas pelo nome e recria as páginas).
