# Matriz de rastreabilidade — Dash_CampeonatoBrasileiro

Data: 27/09/2026 · As colunas **Medida DAX**, **Visual** e **Página** são preenchidas na construção.

Status: `[VIÁVEL]` o modelo já responde · `[REQUER MODELO]` falta medida, coluna ou tabela (os dados existem) · `[REQUER FONTE]` o dado não existe.

| Dor | Pergunta de negócio | KPI | Fórmula de negócio | Fonte | Medida DAX | Visual | Página | Prioridade | Status |
|---|---|---|---|---|---|---|---|---|---|
| D1 | Qual a classificação final de cada temporada? | Pontos | Σ Pontos (3/1/0) | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida |
| D1 | Qual a classificação final de cada temporada? | Posição | Ranking: Pontos → Vitórias → Saldo → Gols Pró | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida |
| D1 | Qual a classificação final de cada temporada? | Vitórias / Empates / Derrotas | Contagem por Resultado | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida |
| D1 | Qual a classificação final de cada temporada? | Saldo de Gols | Σ GolsPro − Σ GolsContra | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida |
| D1 | Quem foi campeão e quem caiu? | Zona (Campeão / Rebaixado) | Posição = 1; Posição > nº de clubes − 4 | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida; 2003–2005 [PENDENTE CLIENTE] |
| D1 | Como a posição evoluiu rodada a rodada? | Pontos Acumulados / Posição na Rodada | Σ Pontos com Rodada ≤ rodada atual, na temporada | fDesempenho, dPartida | - | - | - | Alta | [REQUER MODELO] medida |
| D2 | Qual o aproveitamento em casa × fora? | Aproveitamento % | Pontos ÷ (Jogos × 3), por Mando | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida |
| D2 | A vantagem de mando está caindo ao longo dos anos? | % Vitórias do Mandante | Vitórias Casa ÷ partidas | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida |
| D2 | O campeonato ficou mais equilibrado? | % Empates | Partidas empatadas ÷ partidas | fDesempenho | - | - | - | Média | [REQUER MODELO] medida |
| D3 | O campeonato ficou mais ou menos ofensivo? | Gols por Jogo | Σ gols ÷ partidas distintas | fDesempenho | - | - | - | Alta | [REQUER MODELO] medida |
| D3 | Em que minuto cada clube marca e sofre gols? | Gols por Faixa de Minuto | Contagem de gols por faixa de 15' + acréscimos | fGols | - | - | - | Alta | [REQUER MODELO] coluna FaixaMinuto + medida |
| D3 | Quais times sofrem gols no fim? | Gols Sofridos por Faixa | Gols do adversário na partida, por faixa | fGols, fDesempenho | - | - | - | Média | [REQUER MODELO] medida com TREATAS no adversário |
| D4 | Quem são os artilheiros? | Artilharia | Contagem de gols por atleta, sem gol contra | fGols | - | - | - | Alta | [REQUER MODELO] medida |
| D4 | Qual a dependência de pênaltis? | % Gols de Pênalti | Gols de pênalti ÷ gols | fGols | - | - | - | Média | [REQUER MODELO] medida |
| D4 | Quem é o artilheiro de verdade quando há homônimos? | Artilharia por atleta único | Contagem por ID de atleta | — | - | - | - | Baixa | [REQUER FONTE] sem ID de atleta |
| D5 | Quais clubes são mais indisciplinados? | Cartões por Jogo | Cartões ÷ jogos do clube | fCartoes, fDesempenho | - | - | - | Média | [REQUER MODELO] medida; 2025 sem dados |
| D5 | Quais posições recebem mais cartões? | Cartões por Posição | Contagem por Posicao | fCartoes | - | - | - | Baixa | [REQUER MODELO] medida |
| D6 | Posse de bola explica pontos? | Posse Média | Média de PossePct (ignora vazio) | fDesempenho | - | - | - | Média | [REQUER MODELO] medida |
| D6 | Quem finaliza melhor? | Conversão de Chutes | Gols Pró ÷ Chutes, só com TemEstatistica | fDesempenho | - | - | - | Média | [REQUER MODELO] medida |
| D6 | Quem passa melhor? | Precisão de Passe Média | Média de PrecisaoPassesPct | fDesempenho | - | - | - | Baixa | [REQUER MODELO] medida |
| D7 | Qual o aproveitamento de cada técnico? | Aproveitamento por Técnico | Aproveitamento % por Tecnico | fDesempenho | - | - | - | Média | [REQUER MODELO] medida; só 2014+ |
| D7 | Qual formação rende mais? | Aproveitamento por Formação | Aproveitamento % por Formacao | fDesempenho | - | - | - | Baixa | [REQUER MODELO] medida; só 2015+ |
| D1 | Qual o retrospecto contra um adversário? | Retrospecto no Confronto | V/E/D e gols filtrando Adversario | fDesempenho | - | - | - | Média | [REQUER MODELO] medida + slicer em Adversario |
| — | Quanto cada clube arrecada? | Arrecadação | Σ Arrecadacao (linha Casa) | fDesempenho | - | - | - | Baixa | [REQUER FONTE] só 2025 |
| — | Qual o público médio por clube? | Público Pagante | Σ público ÷ jogos em casa | — | - | - | - | Baixa | [REQUER FONTE] não existe coluna |
| D1 | A classificação calculada bate com a oficial após punições? | Pontos Ajustados | Pontos − punições STJD | — | - | - | - | Baixa | [REQUER FONTE] punições não estão nos dados |
| D1 | Quem se classificou para a Libertadores? | Zona Continental | Posição ≤ vagas da temporada | — | - | - | - | Baixa | [REQUER FONTE] tabela de vagas por ano |

**Resumo:** 27 linhas. Nenhuma `[VIÁVEL]`, porque o modelo ainda não tem medidas. São 22 `[REQUER MODELO]` (os dados existem) e 5 `[REQUER FONTE]`.
