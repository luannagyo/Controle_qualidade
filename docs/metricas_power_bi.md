# 📊 Guia de Métricas & DAX | Power BI

Este documento contém a modelagem analítica e as fórmulas em **DAX (Data Analysis Expressions)** para construção do dashboard de Controle de Qualidade Laboratorial.

---

## 1. Métricas Principais (Medidas DAX)

Crie uma tabela de medidas chamada `_Medidas` no Power BI e adicione:

### A. Total de Procedimentos Cadastrados
```dax
Total_Procedimentos = COUNTROWS('Controles')
```

### B. Procedimentos Concluídos
```dax
Total_Concluidos = 
CALCULATE(
    COUNTROWS('Controles'),
    'Controles'[Status] = "Concluído"
)
```

### C. Procedimentos em Andamento
```dax
Total_Em_Progresso = 
CALCULATE(
    COUNTROWS('Controles'),
    'Controles'[Status] = "Em Progresso"
)
```

### D. Procedimentos em Atraso (Críticos)
```dax
Total_Atrasados = 
CALCULATE(
    COUNTROWS('Controles'),
    'Controles'[Status] = "Atrasado" || 
    (
        'Controles'[Status] <> "Concluído" && 
        'Controles'[Data_Prevista] < TODAY()
    )
)
```

### E. Taxa de Conformidade no Prazo (SLA %)
Mede a eficiência do laboratório em manter os controles dentro da vigência regulamentar:
```dax
Taxa_Conformidade_Prazo = 
VAR ConcluidosNoPrazo = 
    CALCULATE(
        COUNTROWS('Controles'),
        'Controles'[Status] = "Concluído",
        'Controles'[Data_Conclusao] <= 'Controles'[Data_Prevista]
    )
VAR TotalGeral = [Total_Procedimentos]
RETURN
    DIVIDE(ConcluidosNoPrazo, TotalGeral, 0)
```

---

## 2. Recomendação de Layout do Dashboard

* **Cabeçalho:** Logotipo fictício do laboratório, título do relatório e segmentadores de dados (`Setor`, `Periodicidade`, `Status`).
* **Linha de Cards (KPIs no topo):**
  1. `% Conformidade no Prazo` (com meta de 95% para auditoria).
  2. `Controles Atrasados` (cor vermelha de alerta).
  3. `Em Progresso` (cor amarela/laranja).
  4. `Concluídos` (cor verde).
* **Gráficos Centrais:**
  * **Gráfico de Barras Clusterizadas:** Controles por Setor divididos pelo Status.
  * **Gráfico de Donut / Rosca:** Distribuição por Periodicidade (Diário vs Semanal vs Mensal).
* **Tabela Detalhada com Semáforo Condicional:**
  * Colunas: `ID_Processo`, `Nome_Processo`, `Setor`, `Periodicidade`, `Data_Prevista`, `Status`, `Responsavel_Cargo`.
  * Formatação condicional de fundo na coluna `Status`:
    * Verde claro: Concluído
    * Amarelo: Em Progresso
    * Vermelho claro: Atrasado / Vencido
