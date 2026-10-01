# ₿ Dashboard Cripto — Cotações em tempo real no Power BI

Painel em **Power BI** que consulta **APIs públicas de criptomoedas** e mostra, para 8 moedas, a cotação das últimas 24 horas e a evolução do preço de fechamento nos últimos 90 dias, em reais.

![Power BI](https://img.shields.io/badge/Power%20BI-PBIX-F2C811?logo=powerbi&logoColor=black)
![Power Query](https://img.shields.io/badge/Power%20Query-APIs%20REST-2E7D32)
![DAX](https://img.shields.io/badge/DAX-medidas-00B4D8)

![Página de análise: cards de cotação, tabela de indicadores e gráfico de 90 dias](docs/img/analise.jpg)

<table>
  <tr>
    <td width="50%"><img src="docs/img/homepage.jpg" alt="Capa com texto explicativo sobre criptomoedas e botão Iniciar"><br><sub><b>Homepage</b> · introdução e botão para a análise</sub></td>
    <td width="50%"><img src="docs/img/analise.jpg" alt="Análise com cards, tabela e gráfico de área"><br><sub><b>Análise</b> · cotações e histórico</sub></td>
  </tr>
</table>

---

## 📊 Fontes dos dados

O diferencial deste projeto é que **os dados não vêm de uma planilha estática**: a cada atualização, o Power Query chama duas APIs públicas.

| Fonte | O que traz | Uso no painel |
|---|---|---|
| **[Mercado Bitcoin](https://www.mercadobitcoin.net/)** — endpoint `api/{moeda}/ticker` | Máxima, mínima, abertura, último preço, volume, compra e venda das últimas 24h | Cards e tabela de indicadores |
| **[CoinGecko](https://www.coingecko.com/)** — endpoint `coins/{moeda}/ohlc?vs_currency=brl&days=90` | Preços OHLC (abertura, máxima, mínima, fechamento) dos últimos 90 dias, em BRL | Gráfico de fechamento |
| `CriptoDataset.xlsx` | Lista das 8 moedas: nome, sigla e URL do logotipo | Tabela-base que dispara as consultas |

**Moedas acompanhadas:** Bitcoin, Ethereum, Solana, Cardano, ChainLink, Near, Sui e Dogecoin.

> As capturas de tela mostram a cotação da última atualização salva no arquivo (10/12/2024). Ao clicar em **Atualizar**, os valores passam a ser os do momento.

---

## 🖥️ Páginas

| Página | Conteúdo |
|---|---|
| **Homepage** | Texto introdutório sobre criptomoedas, data e hora da última atualização e botão **INICIAR** |
| **Análise** | Cards com o último preço de 5 moedas, tabela com os indicadores de 24h e os logotipos, seletor de moeda em botões e gráfico de área com o fechamento dos últimos 90 dias |

A página de análise fica oculta na barra de abas e é acessada pelo botão da capa, como num aplicativo.

---

## 🏗️ Como foi construído

### Power Query (M)

- **Funções personalizadas que chamam as APIs:**
  - `FD-1(Moeda)` consulta o *ticker* de 24h no Mercado Bitcoin;
  - `f90D(Moeda)` consulta o histórico OHLC de 90 dias na CoinGecko.
- **Uma linha por moeda, uma chamada por linha:** a tabela de moedas do Excel invoca as funções para cada sigla e expande o resultado.
- **`Web.Contents` com `RelativePath`:** a URL base fica fixa e só o trecho da moeda muda, o que permite a atualização agendada no Power BI Service.
- **Conversão de data Unix:** os carimbos de tempo das APIs (em segundos ou milissegundos) viram data e hora, já ajustados para o fuso de Brasília (UTC−3).
- **Tabela despivotada:** os indicadores (máxima, mínima, último…) viram linhas de `Atributo` × `Valor`, o que simplifica as medidas.
- **Data da atualização:** a tabela `Att` guarda `DateTime.LocalNow()` no momento da carga e alimenta o "Data/Hora – Atualização" da capa.

### Modelo e DAX

| Tabela | Conteúdo |
|---|---|
| `DatasetCripto_Diario` | Indicadores de 24h por moeda (Mercado Bitcoin) |
| `DatasetCripto_History` | Fechamento, abertura, máxima e mínima por data (CoinGecko) |
| `Att` | Data e hora da última atualização |

Medidas principais:

```dax
Ultimo valor 24h =
CALCULATE ( SUM ( DatasetCripto_Diario[Valor] ), DatasetCripto_Diario[Atributo] = "ticker.last" )

Rank =
CALCULATE (
    RANKX ( ALL ( DatasetCripto_Diario ), [Ultimo valor 24h], , DESC, DENSE ),
    DatasetCripto_Diario[Atributo] = "ticker.last"
)
```

### Visual

- Tema próprio (`temadashcripto.json`) e imagem de fundo criada para o painel.
- Visuais personalizados do AppSource: **Chiclet Slicer** (seleção de moeda por botões com logotipo) e **Simple Image** (logotipos a partir de URL).

---

## ▶️ Como abrir

1. Instale o **Power BI Desktop**.
2. Abra `Dash_Cripto.pbix`. Os dados da última atualização já vêm salvos no arquivo.
3. Para atualizar as cotações:
   - Em **Transformar dados → Configurações da fonte de dados**, aponte o arquivo `CriptoDataset.xlsx` para a pasta deste repositório.
   - Clique em **Atualizar**. É preciso internet, porque as cotações vêm das APIs.

> ⚠️ APIs públicas podem mudar formato ou limitar o número de chamadas. Se a atualização falhar, confira se os endpoints continuam respondendo.

---

## 📁 Arquivos

```
Dash_Cripto/
├── Dash_Cripto.pbix          ← relatório
├── CriptoDataset.xlsx        ← lista das moedas (nome, sigla, logotipo)
├── BackgroundCripto.jpg      ← imagem de fundo
├── BackgroundCripto.pptx     ← arquivo editável do fundo
└── docs/img/                 ← capturas usadas neste README
```

---

**Dados:** APIs públicas do Mercado Bitcoin e da CoinGecko. Projeto de estudo, sem recomendação de investimento.
