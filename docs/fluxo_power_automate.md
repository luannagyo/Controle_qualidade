# ⚡ Automação com Power Automate & Adaptive Cards

Este documento descreve como configurar o fluxo agendado no **Power Automate** para monitorar prazos e enviar alertas interativos no **Microsoft Teams** e **Copilot**.

---

## 1. Visão Geral do Fluxo

* **Tipo de Fluxo:** Agendado (*Scheduled Cloud Flow*).
* **Frequência:** Diariamente às 07:00 (horário local).
* **Conectores Utilizados:**
  1. `Recurrence` (Gatilho de tempo)
  2. `SharePoint - Obter itens` (Get items)
  3. `Microsoft Teams - Postar cartão adaptável em um chat ou canal`

---

## 2. Parâmetros de Consulta (OData Filter)

No passo **"Obter itens"** da lista do SharePoint:

* **Site Address:** URL do site do Laboratório.
* **List Name:** Nome da lista de Controle de Qualidade.
* **Filter Query (Filtro OData):**
  ```text
  (Status ne 'Concluído') and (Data_Prevista le '@{addDays(utcNow(), 3, 'yyyy-MM-dd')}')
  ```
  > *Explicação:* Retorna apenas os procedimentos que ainda não foram finalizados e cujo prazo vence hoje ou nos próximos 3 dias.

---

## 3. Modelo de Adaptive Card (JSON)

Copie e cole o JSON abaixo na ação **"Postar cartão adaptável em um canal"** do Power Automate:

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "size": "Medium",
      "weight": "Bolder",
      "text": "⚠️ Alerta de Controle de Qualidade Laboratorial",
      "color": "Warning"
    },
    {
      "type": "FactSet",
      "facts": [
        {
          "title": "Procedimento:",
          "value": "@{items('Apply_to_each')?['Nome_Processo']}"
        },
        {
          "title": "Setor:",
          "value": "@{items('Apply_to_each')?['Setor']?['Value']}"
        },
        {
          "title": "Periodicidade:",
          "value": "@{items('Apply_to_each')?['Periodicidade']?['Value']}"
        },
        {
          "title": "Data Limite (SLA):",
          "value": "@{items('Apply_to_each')?['Data_Prevista']}"
        },
        {
          "title": "Status Atual:",
          "value": "@{items('Apply_to_each')?['Status']?['Value']}"
        }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.OpenUrl",
      "title": "Abrir no Microsoft Lists",
      "url": "@{items('Apply_to_each')?['{Link}']}"
    }
  ]
}
```

---

## 4. Integração com o Copilot Agent

O mesmo canal do Teams que recebe os alertas pode ter o **Copilot Studio** integrado como um agente assistente:
* **Fonte de Conhecimento:** Apontar para o site do SharePoint do Laboratório onde estão os POPs (Procedimentos Operacionais Padrão) em PDF e a lista de Controles.
* **Gatilho de Linguagem Natural:** O biomédico pode marcar o bot com `@Copilot quais os controles atrasados na bancada de Bioquímica?` e obter resposta imediata em tempo real.
