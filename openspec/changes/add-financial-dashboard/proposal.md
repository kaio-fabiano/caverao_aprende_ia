# Proposal: Dashboard Financeira — Leitura de Google Sheets

**Change ID:** `add-financial-dashboard`
**Created:** 2026-09-17
**Status:** Draft
**Planilha:** `1cesHZjVHRatNerEo5jIZTHk2MPkDwG2qmyRDVJdt7l8`

---

## Problem Statement

O controle de contas a pagar vive hoje numa planilha do Google. Ler a situação financeira exige abrir a planilha, rolar até o bloco certo e interpretar linhas de total manualmente. Não existe visão de evolução ao longo do tempo, nem alerta de vencimento, nem análise por categoria.

### Estado real da planilha (verificado via API em 2026-09-17)

- Uma aba única contendo **12 blocos byte-a-byte idênticos** — template mensal duplicado, sem dados reais diferenciados por mês.
- **Nenhum rótulo de mês** em lugar algum: é impossível determinar a que mês um bloco pertence.
- **Nenhuma coluna de categoria** — apenas o nome do gasto.
- Colunas atuais: `Gasto` (string), `Valor` (number, formato BRL), `Vencimento` (number, dia 1–31), `Estado` (boolean = pago).
- Linhas de resumo (`Total Gasto`, `Salário`, `Renda Extra`, `Pago`, `Falta Pagar`, `Valor Restante`) misturadas na mesma coluna dos nomes de gasto.
- Valores atuais do template: Total Gasto **R$ 3.271,59** · Salário **R$ 8.500,00** · Renda Extra **R$ 1.000,00** · Valor Restante **R$ 6.228,41**.

### Consequência

Duas das quatro telas desejadas — *evolução entre meses* e *comparação por gasto* — são **tecnicamente impossíveis** sobre a estrutura atual, porque não há como distinguir um mês do outro. Isso torna a reestruturação da planilha uma pré-condição, não uma melhoria opcional.

---

## Proposed Solution

Duas frentes, em ordem obrigatória:

1. **Reestruturar a planilha** para um contrato de dados estável: uma aba por mês nomeada `YYYY-MM`, mais uma coluna `Categoria`.
2. **Construir uma SPA React** que lê essa planilha pela API do Google (somente leitura, via API key), recalcula todos os totais e apresenta quatro visões.

### Descoberta que simplifica a implementação

Lendo pela API do Google com `valueRenderOption=UNFORMATTED_VALUE`, `Valor` chega como **número nativo** e `Estado` como **boolean**. Todo o parsing de moeda pt-BR (`"R$ 1.000,00"`) que seria necessário via CSV é eliminado.

---

## Scope

### In Scope

- Migração da planilha para o contrato de dados (Fase 0, manual)
- Leitura via Google Sheets API com API key, somente leitura
- Recálculo de todos os totais na aplicação
- Quatro telas: mês atual, próximos vencimentos, evolução entre meses, comparação por gasto
- Categorização de gastos via coluna na planilha
- Testes unitários cobrindo todas as regras de negócio

### Out of Scope

- Escrita na planilha (marcar como pago pela dashboard) — exigiria OAuth ou service account
- Exportação de PDF/Excel
- Envio automático ou agendado de relatórios
- Autenticação e multiusuário
- Deploy público (previsto como fase futura, com restrição de key por referrer)

---

## Impact Analysis

| Componente | Muda? | Detalhes |
|---|---|---|
| Planilha | **Sim** | Reestruturação bloqueante: 1 aba por mês + coluna `Categoria` |
| Backend | Não | Arquitetura sem servidor |
| API externa | Sim | Google Sheets API v4, somente leitura |
| Estado | Sim | Hook local `useSheetData`, sem store global |
| UI | Sim | Aplicação nova, quatro telas |

---

## Architecture Considerations

Projeto greenfield — não há padrões prévios a respeitar além do `CLAUDE.md` do graphify.

O padrão central introduzido é a **separação entre domínio puro e camadas de I/O e apresentação** (ver `openspec/project.md`). A justificativa é específica deste domínio: erro silencioso em cálculo financeiro é o pior modo de falha possível aqui, e regras puras são testáveis sem mock de rede.

Decisão de recalcular os totais em vez de lê-los da planilha: garante consistência quando o usuário aplica filtros na tela e permite detectar erro de fórmula na origem.

---

## Success Criteria

- [ ] Planilha migrada para o contrato `YYYY-MM` com coluna `Categoria`
- [ ] Totais recalculados batem com os da planilha para um mês de referência
- [ ] As quatro telas funcionam com dados reais
- [ ] Todas as regras RN-01 a RN-12 cobertas por teste unitário
- [ ] Ausência da aba do mês corrente não quebra a aplicação (RN-09)
- [ ] `npm run lint`, `npm run test` e `npm run build` limpos

---

## Risks & Mitigations

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Código implementado antes da migração da planilha | Média | Alto | Fase 0 declarada bloqueante; nenhuma task de código depende de formato não migrado |
| Aba renomeada fora do padrão `YYYY-MM` | Média | Médio | Regex ignora a aba e a aplicação exibe aviso das abas descartadas |
| API key exposta em deploy futuro | Baixa | Médio | Restrição por API desde já; deploy exigirá restrição por referrer |
| Cota da API do Google excedida | Baixa | Baixo | Cache no hook e refresh manual, nunca polling |
| Divergência entre total da planilha e total calculado | Média | Médio | Passo de verificação compara os dois valores explicitamente |
