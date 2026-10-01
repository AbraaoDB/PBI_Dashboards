# Relatório — páginas, decisões e pendências (27/09/2026)

## Estrutura
| Página | Id | Tipo | Conteúdo |
|---|---|---|---|
| Capa | `pgCapa` | Nativa | Título, KPIs gerais 2003–2025 (1 cardVisual), cards de navegação, cobertura dos dados |
| Temporada | `926615a032c9883d1b7b` | HTML Content (lite) + nativos | Campeão, gols/jogo, artilheiro, rebaixados, classificação, corrida pelo título, gols por minuto, mando |
| Clube | `pgClube` | Nativa | KPIs do clube, pontuação acumulada, casa × fora, gols por faixa de minuto, artilheiros |

- **Chassi:** barra lateral de navegação (224 px) em Temporada e Clube; capa sem barra (é a porta de entrada).
- **Tema:** `BrasileiraoDark.json` (RegisteredResources), escuro com acento ciano; mesmos hex das medidas `91. Cores`.
- **Filtros:** `dPartida[Temporada]` sincronizado entre páginas (`SyncTemporada`, padrão 2025, seleção única); `dClube[Clube]` na página Clube (padrão Flamengo, com busca).

## Decisões
- **HTML Content edição lite** (`htmlContent443BE3AD55E043BF878BED274D3A6865`, AppSource `coacervolimited1596856650797.htmlcontent_certified`): certificado, exporta PDF/PPT. Por isso a página usa só CSS + SVG, sem JavaScript.
- **Por que HTML na Temporada:** composição com anel de aproveitamento, sparklines com ponto da temporada, barra empilhada de mando e corrida com marcação de turno numa única leitura — combinação que os visuais nativos não entregam juntos. Capa e Clube provam o mesmo design com nativos.
- **Medida `Temporada Página HTML`** embrulha só medidas aprovadas (Posição, Pontos, Zona, Artilheiro, Gols por Jogo (Liga), % Vitórias do Mandante, Pontos Acumulados…); nenhuma regra de negócio nova.
- **Tokens de cor:** 14 medidas em `91. Cores` (tema escuro); nenhum hex literal na medida HTML.

## Pendências
1. **1ª abertura baixa o HTML Content do AppSource** — exige internet e visuais do AppSource liberados no tenant. Sem pinning de versão (atualiza sozinho).
2. **Validar no Desktop:** sanitização lite (gradientes SVG `url(#…)`), exportação PDF, e ausência de barra de rolagem no bloco HTML.
3. **Mobile:** layout de celular não configurado.
4. Rebaixamento 2003–2005 e punições do STJD seguem pendentes (ver `docs/levantamento-requisitos/pendencias-cliente.md`).
