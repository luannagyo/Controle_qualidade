# Automação com Power Automate & Adaptive Cards

Este documento descreve a arquitetura de automação no **Power Automate** para gestão integrada da qualidade laboratorial, composta por dois fluxos complementares:

1. **Fluxo Orientado a Eventos:** Monitoramento automático de pastas do SharePoint (baixa automática com registro de evidência).
2. **Fluxo Agendado:** Monitoramento diário de SLA e disparo de alertas proativos no **Microsoft Teams** e **Copilot**.

---

## 1. Fluxo 1: Monitoramento Automático de Pastas (Baixa Automatizada)

* **Tipo de Fluxo:** Automatizado por evento (*Automated Cloud Flow*).
* **Objetivo:** Eliminar trabalho manual de digitação da equipe técnica. Ao salvar o laudo ou dados brutos na pasta de rede/SharePoint da bancada, o sistema dá baixa imediata no controle correspondente.
* **Conectores Utilizados:**
  1. `SharePoint - Quando um arquivo é criado em uma pasta` (ou `em uma biblioteca de documentos`)
  2. `SharePoint - Obter itens` (Get items)
  3. `SharePoint - Atualizar item` (Update item)
  4. `Microsoft Teams - Postar mensagem em um chat ou canal` (Confirmação)

### Etapas do Fluxo:
1. **Gatilho (*Trigger*):**
   * **Gatilho:** `Quando um arquivo é criado em uma pasta` (*When a file is created in a folder*).
   * **Caminho da Pasta:** Ex.: `/Controles_Qualidade/Setor_Bioquimica` (ou biblioteca geral de controles).
2. **Identificação do Procedimento (*Lookup*):**
   * O fluxo obtém o nome do arquivo (ex.: `CQ-001_Laudo_20261120.pdf`) ou metadados da pasta para extrair o código do controle (`ID_Processo`).
   * Executa um filtro OData no Microsoft Lists:
     ```text
     (Status ne 'Concluído') and (ID_Processo eq '@{variables('ID_Processo_Extraido')}')
     ```
3. **Atualização da Lista:**
   * Atualiza a linha correspondente no Microsoft Lists com:
     * `Status` = `Concluído`
     * `Data_Conclusao` = `triggerOutputs()?['body/{Created}']` (carimbo de data/hora oficial de criação do laudo)
     * `Evidencia_Anexo` = `triggerOutputs()?['body/{Link}']` (link direto para abrir o laudo/gráfico na pasta)
4. **Notificação de Sucesso (Opcional):**
   * Mensagem no Teams:  
     `"✅ Controle [Nome_Processo] concluído com sucesso. Laudo técnico arquivado e vinculado ao histórico de auditoria."`

---

## 2. Fluxo 2: Monitoramento Agendado de SLAs e Alertas Preventivos

* **Tipo de Fluxo:** Agendado (*Scheduled Cloud Flow*).
* **Frequência:** Diariamente às 07:00 (abertura da rotina laboratorial).
* **Objetivo:** Notificar os biomédicos caso algum controle obrigatório do dia ainda não tenha tido seu arquivo salvo na pasta ou quando manutenções periódicas estiverem próximas do vencimento.

### Parâmetros de Consulta (OData Filter)
No passo **"Obter itens"** da lista do SharePoint:
* **Filter Query (Filtro OData):**
  ```text
  (Status ne 'Concluído') and (Data_Prevista le '@{addDays(utcNow(), 3, 'yyyy-MM-dd')}')
  ```
  > *Explicação:* Retorna procedimentos pendentes com prazo para os próximos 3 dias ou já em atraso.

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

O canal do Teams que centraliza as rotinas é integrado ao **Copilot Studio**:
* **Fontes de Conhecimento:**
  * Biblioteca de Documentos do SharePoint (onde residem os laudos brutos e POPs).
  * Lista do Microsoft Lists (onde residem os status, prazos e indicadores).
* **Consultas em Linguagem Natural:**  
  O biomédico ou Responsável Técnico pode acionar:
  * *"@Copilot o controle da bancada de Hematologia já foi concluído hoje?"*
  * *"@Copilot mostre os laudos anexados para manutenções semestrais deste mês."*
