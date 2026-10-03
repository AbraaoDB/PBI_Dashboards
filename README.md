# 📊 Portfólio Power BI — Abraão Barbosa

Coleção de dashboards em **Power BI** que mostra a evolução do meu trabalho com dados: dos primeiros relatórios em `.pbix` a projetos completos em **PBIP** (versionáveis em Git), com Power Query, modelagem dimensional, DAX avançado e páginas em **HTML/CSS/SVG renderizadas por DAX**.

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-avan%C3%A7ado-00B4D8)
![Power Query](https://img.shields.io/badge/Power%20Query-M-2E7D32)
![PBIP](https://img.shields.io/badge/PBIP-PBIR%20%7C%20TMDL-7B61FF)
![Projetos](https://img.shields.io/badge/projetos-7-0A1F2E)

---

## 🗂️ Projetos

<table>
  <tr>
    <td width="50%" valign="top">
      <a href="Dash_Ecommerce_Sales/"><img src="Dash_Ecommerce_Sales/docs/img/visao-executiva.png" alt="Página Visão Executiva do dashboard E-Commerce Sales"></a>
      <h3><a href="Dash_Ecommerce_Sales/">🛒 E-Commerce Sales &amp; Customer Analytics</a> &nbsp;<sub>✨ mais recente</sub></h3>
      138 mil pedidos e 397 mil itens: receita realizada × perdida, margem com e sem imposto, nível de serviço de entrega, devolução e retorno por canal de marketing — em 5 páginas <i>dark mode</i> com navegação lateral fixa.<br><br>
      O diferencial não é o visual: a auditoria do ETL encontrou que o cabeçalho do pedido na fonte <b>agrega no máximo 5 itens e subconta 7,9% da receita</b>. O modelo lê o grão do item e documenta por que diverge do CSV oficial.<br>
      <sub><b>PBIP</b> · modelo estrela com chaves surrogadas · 91 medidas · <b>SVG via DAX, sem custom visual</b> · auditoria de ETL, requisitos e modelagem documentados</sub>
    </td>
    <td width="50%" valign="top">
      <a href="Dash_Streamings/"><img src="Dash_Streamings/docs/img/01-capa.png" alt="Capa do dashboard StreamIQ"></a>
      <h3><a href="Dash_Streamings/">🎬 StreamIQ — Catálogo de Streaming</a></h3>
      15 mil títulos de 10 plataformas: volume, qualidade, eficiência de orçamento e prestígio, em 8 páginas.<br>
      <sub><b>PBIP</b> · modelo estrela · 96 medidas · HTML/CSS via DAX · mapa em blocos · filtro cruzado a partir do HTML</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <a href="Dash_CampeonatoBrasileiro/"><img src="Dash_CampeonatoBrasileiro/docs/img/temporada.png" alt="Página Temporada do Brasileirão"></a>
      <h3><a href="Dash_CampeonatoBrasileiro/">⚽ Campeonato Brasileiro 2003–2025</a></h3>
      23 temporadas da Série A: classificação rodada a rodada, campanhas, artilharia, disciplina e estilo de jogo.<br>
      <sub><b>PBIP</b> · 3 fatos · 65 medidas · funções em Power Query · página em HTML via DAX</sub>
    </td>
    <td width="50%" valign="top">
      <a href="Dash_Cripto/"><img src="Dash_Cripto/docs/img/analise.jpg" alt="Dashboard de criptomoedas"></a>
      <h3><a href="Dash_Cripto/">₿ Dashboard Cripto</a></h3>
      Cotações de 8 criptomoedas consultadas em <b>APIs públicas</b> (Mercado Bitcoin e CoinGecko): 24h e histórico de 90 dias.<br>
      <sub>Power Query com funções que chamam APIs REST · Chiclet Slicer · imagens por URL</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <a href="Dash_Projetos/"><img src="Dash_Projetos/docs/img/overview.jpg" alt="Dashboard de gestão de projetos"></a>
      <h3><a href="Dash_Projetos/">📁 Gestão de Projetos</a></h3>
      Orçado × realizado: consumo do orçamento, Curva S, comparação com o ano anterior e ranking por equipe ou gerente.<br>
      <sub>Modelo constelação · inteligência de tempo (YTD, YOY) · parâmetro de campo</sub>
    </td>
    <td width="50%" valign="top">
      <a href="Dash_DRE/"><img src="Dash_DRE/docs/img/dre.jpg" alt="Dashboard financeiro DRE"></a>
      <h3><a href="Dash_DRE/">📒 Dashboard Financeiro — DRE</a></h3>
      DRE gerencial a partir de 273 mil lançamentos contábeis, com subtotais, margens e análises vertical e horizontal.<br>
      <sub>Máscara de DRE dinâmica em DAX (<code>SWITCH</code> + <code>ISINSCOPE</code>) · ingestão de pasta</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <h3><a href="BI_Logistica/">🚚 BI Logística Brasil</a> &nbsp;<sub>🚧 em construção</sub></h3>
      Nível de serviço (OTIF), prazos, custo logístico, qualidade e emissões de CO₂ para 1.000 pedidos simulados.<br><br>
      Planejado em 7 etapas: modelo estrela, medidas DAX em camadas, camada <i>JSON Gold</i> e páginas HTML interativas com filtros internos.<br>
      <sub>Base simulada · roteiro de desenvolvimento documentado</sub>
    </td>
    <td width="50%" valign="top"></td>
  </tr>
</table>

---

## 🧰 O que cada projeto demonstra

| Competência | E-commerce | StreamIQ | Brasileirão | Cripto | Projetos | DRE | Logística |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Power Query: limpeza e tipagem | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 🚧 |
| Power Query: funções e parâmetros | ✅ | ✅ | ✅ | ✅ | | | 🚧 |
| Power Query: camada de staging (Fonte → Staging → Modelo) | ✅ | | | | | | 🚧 |
| Consumo de APIs REST | | | | ✅ | | | |
| Ingestão de pasta de arquivos | | | | | | ✅ | |
| Modelo estrela / constelação | ✅ | ✅ | ✅ | | ✅ | ✅ | 🚧 |
| Chaves surrogadas inteiras | ✅ | | | | | | |
| Tabela calendário | ✅ | ✅ | ✅ | | ✅ | ✅ | 🚧 |
| DAX: inteligência de tempo | ✅ | ✅ | ✅ | | ✅ | ✅ | 🚧 |
| DAX: lógica avançada (rankings, `SWITCH`, `ISINSCOPE`, `CROSSFILTER`) | ✅ | ✅ | ✅ | ✅ | | ✅ | 🚧 |
| HTML/CSS/SVG renderizado por DAX | ✅ | ✅ | ✅ | | | | 🚧 |
| Parâmetro de campo | | | | | ✅ | | |
| Tema e identidade visual próprios | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | |
| PBIP / PBIR / TMDL (versionável) | ✅ | ✅ | ✅ | | | | 🚧 |
| Documentação (requisitos, modelagem, validação) | ✅ | ✅ | ✅ | | | | ✅ |
| **Auditoria de dados: reconciliação de grão e defeito de fonte** | ✅ | | | | | | |
| Acessibilidade (contraste AA, `altText`, cor nunca sozinha) | ✅ | | | | | | |

---

## 📈 Evolução

1. **Primeiros projetos (`.pbix`)** — *DRE, Gestão de Projetos e Cripto*: fundamentos de Power Query, modelagem, DAX e design de relatórios, incluindo consumo de APIs.
2. **Projetos em PBIP** — *Campeonato Brasileiro e StreamIQ*: relatório e modelo em arquivos de texto (PBIR + TMDL), versionados em Git, com levantamento de requisitos, auditoria do ETL, modelo estrela documentado, medidas validadas por consultas DAX e páginas em HTML/CSS via DAX.
3. **Engenharia analítica** — *E-Commerce Sales*: o trabalho deixa de ser "construir o dashboard" e passa a ser **confiar no número**. Auditoria do ETL com achados medidos (parsing de CSV corrompendo 8,7% das linhas, decimais tipados como inteiro, sete moedas somadas sem conversão), reconciliação entre duas fontes de receita que divergiam R$ 15,2 milhões, modelagem dimensional com cada decisão justificada, e 91 medidas com o valor esperado conferido contra os arquivos de origem.
4. **Em construção** — *BI Logística*: dashboard orientado a um roteiro de desenvolvimento por etapas, com camada de dados *Gold* em JSON alimentando páginas HTML interativas.

---

## ▶️ Como abrir os projetos

1. Instale o **Power BI Desktop** (versão recente, para suportar PBIP/PBIR).
2. Clone o repositório:
   ```bash
   git clone https://github.com/AbraaoDB/PBI_Dashboards.git
   ```
3. Abra o arquivo do projeto:
   - **`.pbix`** (Cripto, DRE, Projetos): abre direto, com os dados salvos no arquivo.
   - **`.pbip`** (E-Commerce Sales, StreamIQ, Campeonato Brasileiro): ajuste o parâmetro com o caminho da pasta de dados e clique em **Atualizar**. O passo a passo está no README de cada projeto.
4. Projetos com páginas HTML usam o visual certificado **HTML Content**, que o Power BI baixa do AppSource na primeira abertura (requer internet). O **E-Commerce Sales é exceção**: usa SVG via `dataCategory: ImageUrl`, que renderiza nativamente — abre sem custom visual e sem internet.

---

## 📁 Estrutura

```
PBI_Dashboards/
├── Dash_Ecommerce_Sales/       ← E-Commerce Sales (PBIP)
├── Dash_Streamings/            ← StreamIQ (PBIP)
├── Dash_CampeonatoBrasileiro/  ← Brasileirão (PBIP)
├── Dash_Cripto/                ← Cripto (PBIX + APIs)
├── Dash_Projetos/              ← Gestão de Projetos (PBIX)
├── Dash_DRE/                   ← DRE (PBIX)
└── BI_Logistica/               ← Logística (em construção)
```

Cada pasta tem seu próprio `README.md` com fonte dos dados, páginas, modelo, medidas e instruções.

---

## 🙏 Créditos dos dados

| Projeto | Fonte |
|---|---|
| E-Commerce Sales | [E-Commerce Sales and Customer Analytics](https://www.kaggle.com/datasets/datascikhan/e-commerce-sales-and-customer-analytics) — Kaggle, Shair Khan (CC0, dados sintéticos) |
| StreamIQ | [Streaming Content Catalog: Netflix, Prime, Disney+](https://www.kaggle.com/datasets/meruvakodandasuraj/streaming-content-catalog-netflix-prime-disney) — Kaggle, Meruva Kodanda Suraj (dados sintéticos) |
| Campeonato Brasileiro | [Brasileirao_Dataset](https://github.com/adaoduque/Brasileirao_Dataset) — GitHub, Adão Duque |
| Cripto | APIs públicas do Mercado Bitcoin e da CoinGecko |
| Gestão de Projetos | Base fictícia de projetos |
| DRE | Base de lançamentos contábeis de um desafio de estudo |
| Logística | Base 100% simulada |

Projetos de estudo e portfólio, sem fins comerciais. Marcas e nomes citados pertencem a seus donos.

---

**Contato:** [github.com/AbraaoDB](https://github.com/AbraaoDB)
