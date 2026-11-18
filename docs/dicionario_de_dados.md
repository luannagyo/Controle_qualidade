# 📖 Dicionário de Dados | Microsoft Lists

Este documento especifica a modelagem dos dados para a lista de **Controle de Qualidade** no Microsoft Lists / SharePoint.

---

## 1. Estrutura dos Campos

| Nome da Coluna | Tipo no SharePoint | Obrigatório | Opções / Formato | Justificativa Laboratorial |
| :--- | :--- | :---: | :--- | :--- |
| **ID_Processo** | Linha única de texto | Sim | Ex: `CQ-001`, `CQ-002` | Identificação única para rastreabilidade de auditoria. |
| **Nome_Processo** | Linha única de texto | Sim | Texto livre (até 255 caracteres) | Nome padronizado do POP ou manutenção. |
| **Setor** | Opção (Menu suspenso) | Sim | `Bioquímica`, `Hematologia`, `Imunologia`, `Microbiologia`, `Uroanálise`, `Geral`, `Qualidade` | Segmentação por bancada analítica. |
| **Periodicidade** | Opção (Menu suspenso) | Sim | `Diário`, `Semanal`, `Quinzenal`, `Mensal`, `Semestral`, `Anual` | Define as regras de cálculo de vencimento (SLA). |
| **Responsavel_Cargo** | Linha única de texto / Pessoa | Sim | Texto ou Seleção de Usuário do M365 | Define a alçada técnica responsável pela execução. |
| **Data_Prevista** | Data | Sim | `AAAA-MM-DD` | Prazo limite regulamentar ou operacional para a execução. |
| **Status** | Opção (Menu suspenso) | Sim | `Pendente`, `Em Progresso`, `Concluído`, `Atrasado`, `Cancelado` | Estado de ciclo de vida do procedimento. |
| **Data_Conclusao** | Data e Hora | Não | `AAAA-MM-DD HH:mm` | Evidência cronológica do momento em que a rotina foi finalizada. |
| **Evidencia_Anexo** | Anexos do SharePoint | Não | Arquivos (PDF, PNG, CSV) | Comprovante obrigatório para acreditação (RDC 786/PALC/ISO 15189). |
| **Acao_Corretiva_Obs** | Várias linhas de texto | Não | Texto simples | Registro de desvios (ex: Regras de Westgard violadas) e conduta tomada. |

---

## 2. Regras de Negócio e Validações

1. **Transição de Status:**
   * Um item nasce com status `Pendente`.
   * Quando o biomédico inicia a rotina no analisador, o status transiciona para `Em Progresso`.
   * Ao finalizar e registrar a validação, o status passa para `Concluído` e o campo `Data_Conclusao` é preenchido.
2. **Cálculo de Atraso Automático:**
   * Se a data atual for maior que `Data_Prevista` e o `Status` não for `Concluído`, o procedimento é classificado como **Atrasado**.
3. **Audit Trail (Trilha de Auditoria):**
   * O Microsoft Lists mantém nativamente a autoria de quem criou e quem alterou (`Criado por`, `Modificado por`), garantindo não repúdio das informações.
