# Análise de Big Data sobre Antidepressivos no Brasil

Projeto de análise de dados sobre o consumo de antidepressivos no Brasil, desenvolvido no **Databricks** com pipeline em **PySpark/Delta Lake**, modelagem estatística e previsão de séries temporais. Os resultados tratados alimentam dashboards no **Power BI**.

## Objetivo

Investigar padrões de vendas de antidepressivos por região e estado, relacionando o consumo a indicadores de saúde pública (prevalência de depressão e densidade de psiquiatras), e gerar projeções futuras para apoiar análises de saúde pública e tomada de decisão.

## Escopo da análise

- Limpeza e padronização de dados de vendas e indicadores sociossanitários
- Criação de KPIs comparáveis entre regiões (vendas por 100 mil habitantes)
- Regressão linear para explicar fatores associados ao consumo
- Previsão de tendências com **Prophet** (horizonte de 4 anos)
- Exportação de tabelas Delta prontas para consumo em BI

## Arquitetura do pipeline

```
antidepressivos_brasil_unificado  (dados brutos)
            │
            ▼  ETL PySpark
antidepressivos_limpos_bi         (dados limpos + KPIs)
            │
            ├──► Regressão Linear (scikit-learn)
            │
            └──► Previsão Prophet
                        │
                        ▼
         antidepressivos_previsao_prophet
                        │
                        ▼
                   Power BI
```

## Tabelas Delta

| Tabela | Descrição |
|--------|-----------|
| `workspace.default.antidepressivos_brasil_unificado` | Fonte unificada com dados brutos (entrada) |
| `workspace.default.antidepressivos_limpos_bi` | Dados tratados e enriquecidos para análise e BI |
| `workspace.default.antidepressivos_previsao_prophet` | Projeções com intervalo de confiança de 95% |

## Campos principais (tabela limpa)

| Coluna | Descrição |
|--------|-----------|
| `id_linha` | Identificador da linha |
| `ano_referencia` | Ano dos dados (filtro: ≥ 2019) |
| `regiao_nome` | Região (texto padronizado em maiúsculas) |
| `estado_nome` | Estado |
| `uf_sigla` | Sigla da UF (`BR` quando nulo — nível nacional) |
| `vendas_unidades` | Unidades vendidas estimadas |
| `percentual_prevalencia_depressao` | Prevalência regional de depressão (%) |
| `densidade_psiquiatras_por_100mil` | Psiquiatras por 100 mil habitantes |
| `kpi_vendas_por_100mil_hab` | KPI principal: vendas por 100 mil habitantes |

## Modelos utilizados

### 1. Regressão linear

Estima o KPI de vendas com base em:

- Ano de referência
- Prevalência de depressão
- Densidade de psiquiatras
- Região (codificada com `LabelEncoder`)

Métricas de avaliação: **MAE**, **RMSE** e **R²**. Inclui gráficos de valores reais vs. preditos e distribuição de resíduos.

### 2. Prophet (série temporal)

Agrega o KPI nacional por ano e projeta **4 anos** à frente, usando como regressores a prevalência de depressão e a densidade de psiquiatras. Gera intervalo de confiança de 95% e, quando possível, validação cruzada temporal.

## Tecnologias

- **Databricks** — ambiente de execução
- **PySpark** — ETL e persistência em Delta Lake
- **pandas / NumPy** — preparação para modelagem
- **scikit-learn** — regressão linear e métricas
- **Prophet** — previsão de séries temporais
- **matplotlib** — visualizações
- **Power BI** — dashboards finais

## Como executar

1. Importe o notebook no workspace Databricks:

   `(Clone) Explore workspace.default.antidepressivos_brasil_unificado 2026-05-25 19_40_09.ipynb`

2. Confirme que a tabela de origem existe:

   ```sql
   SELECT * FROM workspace.default.antidepressivos_brasil_unificado LIMIT 10;
   ```

3. Execute as células **em ordem**:
   - **Célula 1** — ETL e gravação em `antidepressivos_limpos_bi`
   - **Célula 2** — regressão linear e gráficos de diagnóstico
   - **Célula 3** — instalação do Prophet, previsão e exportação para `antidepressivos_previsao_prophet`

4. Conecte o Power BI às tabelas Delta geradas para montar os dashboards.

> **Requisito:** cluster Databricks com suporte a PySpark e permissão de escrita nas tabelas do schema `workspace.default`.

## Estrutura do repositório

```
.
├── README.md
└── (Clone) Explore workspace.default.antidepressivos_brasil_unificado 2026-05-25 19_40_09.ipynb
```

## Observações

- O filtro `ano_referencia >= 2019` mantém apenas dados históricos recentes.
- Na previsão Prophet, anos futuros sem valor observado usam a **média histórica** de prevalência e densidade de psiquiatras como fallback.
- A validação cruzada do Prophet pode ser ignorada automaticamente se a série temporal for curta demais para os parâmetros configurados.

## Licença

Consulte o repositório para informações de licenciamento.
