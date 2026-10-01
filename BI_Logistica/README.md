# 🚚 BI Logística Brasil — OTIF, lead time e custo logístico

> 🚧 **Projeto em construção.** Este repositório contém a base de dados e o roteiro de desenvolvimento; os arquivos do relatório ainda não foram publicados.

Painel em **Power BI** para gestão de entregas no Brasil: nível de serviço (**OTIF**), prazos, atrasos, custo logístico, qualidade (devoluções, avarias e extravios) e emissões de CO₂, por transportadora, região, modal e categoria de produto.

![Power BI](https://img.shields.io/badge/Power%20BI-PBIP-F2C811?logo=powerbi&logoColor=black)
![Status](https://img.shields.io/badge/status-em%20constru%C3%A7%C3%A3o-orange)
![Dados](https://img.shields.io/badge/dados-1.000%20pedidos%20simulados-0A1F2E)

---

## 📊 Dados

Arquivo `base_logistica_simulada_brasil.xlsx`, com **dados 100% simulados** para estudo e construção de dashboards. Os nomes de transportadoras são fictícios.

| Aba | Conteúdo |
|---|---|
| `Base_Logistica` | **1.000 pedidos**, 58 colunas, de 01/07/2025 a 10/07/2026 |
| `Dicionario_Dados` | Descrição, tipo e uso sugerido de cada coluna |
| `Guia_Dashboard` | KPIs e visuais sugeridos |
| `Premissas` | Natureza da base, período e regras de coerência aplicadas |

**O que cada pedido traz:**

- **Datas:** pedido, aprovação, expedição, prazo prometido e entrega.
- **Operação:** origem e destino (cidade, UF e região), transportadora, modal, distância, peso e volume.
- **Custos:** valor do pedido, frete, armazenagem, separação, última milha e custo total.
- **Serviço:** lead time, dias de atraso, entrega no prazo, pedido completo, OTIF, status e motivo de atraso.
- **Qualidade:** avaria, extravio, devolução e nota do cliente.
- **Ambiente:** emissão de CO₂.

**Perfil da base:** 7 transportadoras · 5 regiões de destino (49% Sudeste) · 3 modais (95% rodoviário) · 8+ categorias de produto · 5 canais de venda · 92% dos pedidos com status *Entregue*.

---

## 🎯 Escopo planejado

O desenvolvimento segue um roteiro em 7 etapas, registrado em [`Prompts.txt`](Prompts.txt):

| Etapa | Entrega |
|---|---|
| 1. Power Query e modelo | Diagnóstico da granularidade, staging rastreável, modelo estrela e `dCalendario` |
| 2. Medidas DAX | Pedidos, custos, **OTIF**, on-time, in-full, lead time, atraso médio, devoluções, avarias, extravios, nota do cliente, CO₂ e comparação com o mês anterior |
| 3. Camada Gold | Medida `JSON Gold Dashboard Logistica`, única fonte de dados das páginas HTML |
| 4. Página principal | Dashboard em HTML/CSS/JS via DAX, com filtros internos, modo claro e escuro, cards, gráfico mensal e rankings |
| 5. Análises dinâmicas | Leitura executiva, mediana do OTIF, risco por modal e tabela de transportadoras ordenável |
| 6. Acabamento e capa | Header animado, ticker de produtos e capa com anel de progresso do OTIF |
| 7. Sparklines | Mini-gráficos de tendência nos cards de KPI |

**Regras de negócio já definidas no roteiro:**

- Pedidos contados com contagem distinta quando a fato estiver no nível de item.
- OTIF exige *on time* e *in full* no **mesmo pedido**.
- Pedidos pendentes permanecem na base e **não** são classificados como atrasados sem regra confirmada.
- Nenhuma métrica é criada sem campo e regra que a sustentem.

---

## 📁 Arquivos

```
BI_Logistica/
├── BI_Logistica_Claude.pbip            ← atalho do projeto (PBIP)
├── base_logistica_simulada_brasil.xlsx ← base simulada de 1.000 pedidos
└── Prompts.txt                         ← roteiro de desenvolvimento em 7 etapas
```

> ⚠️ O arquivo `.pbip` aponta para a pasta `BI_Logistica_Claude.Report`, que ainda não foi enviada ao repositório. Por isso o projeto ainda não abre no Power BI Desktop.

---

**Dados:** base 100% simulada. Não representa informações reais de mercado nem o desempenho de empresas.
