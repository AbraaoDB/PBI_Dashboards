# Levantamento de requisitos — Dash_CampeonatoBrasileiro

Data: 27/09/2026 · Base: modelo semântico (`fDesempenho`, `fGols`, `fCartoes`, `dPartida`, `dClube`, `dCalendario`) e perfil dos quatro CSVs de origem.

> **Não há transcrição nem briefing.** Todas as dores, a audiência e os critérios de sucesso abaixo são **hipóteses derivadas do que os dados permitem responder**, marcadas como `[HIPÓTESE]`. Só o que foi medido diretamente nos dados leva `[CONFIRMADO]`. As decisões que precisam do dono do projeto estão em `pendencias-cliente.md`.

---

## 0. Audiência e decisão

| Item | Definição |
|---|---|
| **Audiência principal** `[HIPÓTESE]` | Analistas e entusiastas de futebol (jornalista esportivo, analista de desempenho, torcedor engajado). Perfil de **exploração e comparação**, não de monitoramento operacional. |
| **Audiência secundária** `[HIPÓTESE]` | Recrutador ou avaliador de portfólio de BI: precisa entender a história em 30 segundos na primeira página. |
| **Trabalho a ser feito** | Comparar clubes e temporadas; entender a história de uma temporada; achar padrões e outliers (vantagem de mando, gols tardios, disciplina). |
| **Tom** | Narrativo-analítico: capa com a história da temporada, páginas de exploração com filtros. |
| **Critério de sucesso** `[HIPÓTESE]` | O relatório funcionou se… **um usuário escolhe uma temporada e um clube e, sem ajuda, responde em menos de 1 minuto: em que posição o clube terminou, como foi a campanha em casa e fora, quem fez os gols e como ele se compara à média da liga.** |

---

## 1. Dores (derivadas do modelo)

| # | Dor | Evidência (dados) | Área | Impacto | Status |
|---|---|---|---|---|---|
| D1 | Não existe uma visão consolidada de classificação por temporada; é preciso somar pontos jogo a jogo | Placar por partida em `fDesempenho`, sem tabela de classificação na fonte | Análise de campeonato | Perguntas básicas (campeão, rebaixados, G4) exigem cálculo manual | `[HIPÓTESE]` |
| D2 | Não se sabe o quanto jogar em casa pesa no resultado, nem se isso mudou ao longo dos anos | Vitória do mandante oscila de 44% (2017, 2022) a 55% (2008) `[CONFIRMADO]` | Desempenho | Leitura errada de campanhas ("fez 70% dos pontos em casa") | `[HIPÓTESE]` |
| D3 | Não se sabe quando e como os gols acontecem | 23,5% dos gols saem entre 75' e 90', 10,5% nos acréscimos e 9,5% de pênalti `[CONFIRMADO]` | Desempenho tático | Não dá para identificar times que "decidem no fim" ou que sofrem gols tardios | `[HIPÓTESE]` |
| D4 | Não há ranking de artilheiros por temporada ou clube | `fGols[Atleta]` com 1.636 nomes, de 2014 em diante `[CONFIRMADO]` | Jogadores | Pergunta frequente sem resposta direta | `[HIPÓTESE]` |
| D5 | Não se compara disciplina (cartões, faltas) entre clubes | `fCartoes` 2014–2024, 5,2% dos cartões vermelhos `[CONFIRMADO]` | Disciplina | Sem base para comparar estilo de jogo e árbitros | `[HIPÓTESE]` |
| D6 | Estatísticas de jogo (posse, chutes, passes) não são ligadas ao resultado | `fDesempenho` com estatísticas em 2015–2023 e 2025 `[CONFIRMADO]` | Análise de desempenho | Não dá para testar "quem tem mais posse vence mais?" | `[HIPÓTESE]` |
| D7 | Não se avalia o trabalho dos técnicos | `Tecnico` preenchido a partir de 2014 `[CONFIRMADO]` | Gestão esportiva | Trocas de técnico sem leitura de aproveitamento | `[HIPÓTESE]` |

---

## 2. Perguntas de negócio

Cada pergunta leva a uma decisão ou conclusão esperada (critério de qualidade da skill).

| # | Pergunta | Decisão / conclusão que ela permite | Dor |
|---|---|---|---|
| P1 | Qual a classificação final de cada temporada? Quem foi campeão e quem caiu? | Situar qualquer campanha no contexto da temporada | D1 |
| P2 | Como a posição de um clube evoluiu rodada a rodada? | Identificar arrancadas e quedas (quando a campanha virou) | D1 |
| P3 | Qual o aproveitamento em casa × fora de cada clube, e da liga ao longo dos anos? | Dizer se um clube depende do mando e se a vantagem de mando está caindo | D2 |
| P4 | Quantos gols por jogo a liga tem por temporada, e qual o percentual de empates? | Avaliar se o campeonato ficou mais ou menos ofensivo | D2, D3 |
| P5 | Em que faixa de minuto cada clube marca e sofre gols? | Identificar times fortes ou frágeis no fim do jogo | D3 |
| P6 | Quem são os artilheiros por temporada e por clube? Qual o percentual de gols de pênalti? | Ranquear jogadores; medir dependência de pênaltis | D4 |
| P7 | Quais clubes e posições recebem mais cartões? | Comparar disciplina; relacionar cartões com resultado | D5 |
| P8 | Posse, chutes e precisão de passe explicam pontos? | Validar ou derrubar narrativas de estilo de jogo | D6 |
| P9 | Qual o aproveitamento de cada técnico por clube e temporada? | Avaliar o impacto de trocas de técnico | D7 |
| P10 | Qual o retrospecto de um clube contra um adversário específico? | Contexto de clássicos e confrontos diretos | D1 |

---

## 3. KPIs

Grão de referência: **clube × partida** em `fDesempenho`. Métricas **de liga** (gols por jogo, % vitórias do mandante) contam partidas uma vez só, pela regra R4.

| KPI | Objetivo | Fórmula de negócio | Grão | Dimensões | Fonte | Critério de aceite | Escopo |
|---|---|---|---|---|---|---|---|
| Jogos | Base de todas as taxas | Nº de linhas clube×partida | clube×partida | Clube, Temporada, Mando | fDesempenho | 38 jogos por clube em 2006–2025 | 1ª entrega |
| Pontos | Classificação | Σ Pontos (3/1/0) | clube×partida | Clube, Temporada, Rodada | fDesempenho | 2023: Palmeiras 70, Grêmio 68 | 1ª entrega |
| Vitórias / Empates / Derrotas | Campanha | Contagem por Resultado | clube×partida | Clube, Temporada, Mando | fDesempenho | V+E+D = Jogos | 1ª entrega |
| Aproveitamento % | Comparar campanhas de tamanhos diferentes | Pontos ÷ (Jogos × 3) | clube×partida | Clube, Temporada, Mando, Técnico | fDesempenho | 0–100%; campeão 2020 = 71/114 = 62,3% | 1ª entrega |
| Gols Pró / Contra / Saldo | Ataque e defesa | Σ GolsPro; Σ GolsContra; diferença | clube×partida | Clube, Temporada, Mando | fDesempenho | Soma de Gols Pró da liga = soma de Gols Contra | 1ª entrega |
| Posição | Classificação | Ranking por Pontos → Vitórias → Saldo → Gols Pró (R2) | clube×temporada | Clube, Temporada, Rodada | fDesempenho | Bate com a tabela oficial nas temporadas de controle | 1ª entrega |
| Gols por Jogo (liga) | Tendência ofensiva | Σ gols ÷ nº de partidas distintas | partida | Temporada | fDesempenho / dPartida | 2025 = 2,52; 2018 = 2,18 | 1ª entrega |
| % Vitórias do Mandante | Vantagem de mando | Vitórias com Mando=Casa ÷ partidas | partida | Temporada | fDesempenho | 2008 = 55%; 2022 = 44% | 1ª entrega |
| % Empates (liga) | Equilíbrio | Partidas empatadas ÷ partidas | partida | Temporada | fDesempenho | 2010 = 31% | 1ª entrega |
| Gols por Faixa de Minuto | Momento do gol | Contagem de gols por faixa de 15' (+ acréscimos) | gol | Clube, Temporada, Faixa | fGols | Faixas somam 100%; 75'–90' ≈ 23,5% | 1ª entrega |
| Artilharia | Ranking de jogadores | Contagem de gols por atleta, **sem gol contra** (R5) | gol | Atleta, Clube, Temporada | fGols | Top-N por temporada, a partir de 2014 | 1ª entrega |
| % Gols de Pênalti | Dependência de bola parada | Gols de pênalti ÷ gols | gol | Clube, Atleta, Temporada | fGols | Liga ≈ 9,5% | 1ª entrega |
| Cartões por Jogo | Disciplina | (Amarelos + Vermelhos) ÷ jogos do clube | cartão / clube×partida | Clube, Temporada, Posição | fCartoes, fDesempenho | Só 2014–2024 (R7) | 2ª entrega |
| Posse Média / Precisão de Passe | Estilo | Média de PossePct / PrecisaoPassesPct, ignorando vazio | clube×partida | Clube, Temporada, Resultado | fDesempenho | Posse média da liga ≈ 50% | 2ª entrega |
| Conversão de Chutes | Eficiência | Gols Pró ÷ Chutes (só com TemEstatistica) | clube×partida | Clube, Temporada | fDesempenho | Denominador só com jogos que têm estatística | 2ª entrega |
| Aproveitamento por Técnico | Gestão | Aproveitamento % filtrado por Tecnico | clube×partida | Técnico, Clube, Temporada | fDesempenho | Só 2014+ | 2ª entrega |
| Retrospecto no Confronto | Clássicos | V/E/D e gols filtrando Adversario | clube×partida | Clube, Adversário | fDesempenho | V do clube A = D do clube B | 2ª entrega |
| Arrecadação | Receita de bilheteria | Σ Arrecadacao (só linha Casa) | partida | Clube, Arena | fDesempenho | Só 2025 (R7) | Diferido |

---

## 4. Regras de negócio

| # | Regra | Marcação |
|---|---|---|
| R1 | Vitória = 3 pontos, empate = 1, derrota = 0 em todas as temporadas (pontos corridos desde 2003) | `[CONFIRMADO]` pelos totais de 2020 e 2023 |
| R2 | Critério de desempate na classificação: Pontos → Vitórias → Saldo de gols → Gols pró | `[HIPÓTESE]` (critério CBF usual; os critérios seguintes, como confronto direto e cartões, não entram) |
| R3 | A temporada 2020 vai de ago/2020 a fev/2021 (`fxTemporada`) | `[CONFIRMADO]` (380 jogos em cada temporada) |
| R4 | Métricas de liga contam cada partida uma vez (`DISTINCTCOUNT(PartidaId)` ou `Mando = "Casa"`), porque `fDesempenho` tem 2 linhas por jogo | `[CONFIRMADO]` pelo grão |
| R5 | Gol contra conta para o clube beneficiado, e não entra na artilharia do atleta que o marcou | `[CONFIRMADO]` para o clube (gols batem com o placar) / `[HIPÓTESE]` para a artilharia |
| R6 | Zona de rebaixamento = 4 últimos (2006+); em 2003–2005, com 24 e 22 clubes, o número muda | `[PENDENTE CLIENTE]` para 2003–2005 |
| R7 | Cobertura por tema: gols detalhados 2014+; cartões 2014–2024; estatísticas 2015–2023 e 2025; técnico 2014+; formação 2015+; arrecadação só 2025 | `[CONFIRMADO]` |
| R8 | Punições com perda de pontos no tribunal (STJD) **não** estão nos dados: a classificação calculada reflete só o campo | `[PENDENTE CLIENTE]` |
| R9 | 2016 tem 379 jogos (Chapecoense × Atlético-MG não disputado) | `[CONFIRMADO]` na contagem |
| R10 | Vagas de Libertadores e Sul-Americana variam por ano e dependem da Copa do Brasil | `[PENDENTE CLIENTE]` (fora do escopo sem uma tabela de regras) |

---

## 5. Inventário do modelo

```
Fatos:
  fDesempenho  (grão: clube × partida, 18.330)   chaves: PartidaId, Data, Clube
  fGols        (grão: gol, 10.820; 2014+)         chaves: PartidaId, Data, Clube
  fCartoes     (grão: cartão, 20.953; 2014–2024)  chaves: PartidaId, Data, Clube
Dimensões:
  dPartida (9.165) · dClube (46) · dCalendario (8.401 dias, com Temporada)
Medidas existentes: nenhuma
Trabalho de modelo faltando:
  - Tabela Medidas + todas as medidas da seção 3
  - Faixa de minuto em fGols e fCartoes (coluna FaixaMinuto + ordem)
  - Tabela de regras por temporada (nº de rebaixados, vagas) se R6/R10 entrarem
Riscos:
  - Dupla contagem em métricas de liga (R4)
  - Lacunas de cobertura (R7): um visual de estatísticas em 2024 mostra vazio, não zero
  - Atleta identificado só pelo nome: homônimos somam juntos
  - Nome de arena com variantes históricas (Castelão / Arena Castelão)
```

---

## 6. Entrega e restrições

| Item | Valor |
|---|---|
| **Alvo** | PBIP local (`Dash_CampeonatoBrasileiro`) `[CONFIRMADO]`; publicação `[PENDENTE CLIENTE]` |
| **Permissão de editar o modelo** | Sim `[CONFIRMADO]` (reestruturado nesta sessão) |
| **Acessibilidade** | WCAG AA: contraste, sem depender só de cor para vitória/derrota, texto alternativo nos visuais |
| **Ressalvas de dados** | Cobertura R7; sem punições (R8); arena com variantes; sem ID de atleta |
| **Ferramental** | Power BI Desktop, MCP `powerbi-modeling-mcp` |
| **Atualização** | Carga manual dos CSVs; dados até o fim de 2025 `[CONFIRMADO]` |

---

## 7. Escopo: primeira entrega × diferido

| Página | Conteúdo | Perguntas | Entrega |
|---|---|---|---|
| **1. Temporada** (capa) | Campeão, artilheiro, gols por jogo, % mandante; tabela de classificação com zonas; evolução da posição por rodada | P1, P2, P4 | 1ª |
| **2. Clube** | Cartões de pontos, aproveitamento e saldo; casa × fora; campanha rodada a rodada; artilheiros do clube | P1, P2, P3, P6 | 1ª |
| **3. Gols** | Gols por faixa de minuto (pró × contra), tipo de gol, ranking de artilharia | P5, P6 | 1ª |
| **4. Histórico da liga** | Tendências 2003–2025: gols por jogo, % mandante, % empates | P3, P4 | 1ª |
| **5. Estilo de jogo** | Posse × pontos, conversão, precisão de passe | P8 | 2ª |
| **6. Disciplina** | Cartões por clube, posição e minuto | P7 | 2ª |
| **7. Técnicos e confrontos** | Aproveitamento por técnico; retrospecto clube × adversário | P9, P10 | 2ª |
| Arrecadação/público | Só 2025, amostra insuficiente para tendência | — | Diferido |
| Vagas continentais | Exige tabela de regras por temporada (R10) | — | Diferido |

**Por que essa divisão:** a 1ª entrega usa só dados completos de 2003 a 2025 (ou de 2014+ para gols) e responde ao critério de sucesso. A 2ª depende de temas com lacunas (R7) que exigem avisos de cobertura nos visuais.

---

## 8. Interações esperadas

- **Slicers globais (sincronizados):** Temporada (seleção única, padrão = última) e Clube (busca, 46 valores)
- **Slicers de página:** Mando (Casa/Fora) nas páginas 2 e 5; Tipo de gol na página 3; faixa de Rodada na página 1
- **Drill-through:** Temporada → Clube (a partir da linha da classificação); Clube → Gols do clube
- **Tooltip de página:** ao passar sobre um clube na classificação, mostrar a mini-campanha (V/E/D, gols, últimos 5 jogos)
- **Navegação:** barra de páginas com botões; bookmark para alternar entre "Tabela" e "Gráfico de evolução" na página 1
- **Aviso de cobertura:** texto dinâmico quando a temporada filtrada não tem dados do tema (ex.: "Estatísticas indisponíveis para 2024")
