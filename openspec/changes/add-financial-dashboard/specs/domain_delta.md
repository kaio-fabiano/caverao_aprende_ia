# Delta: Domínio Financeiro

**Change ID:** `add-financial-dashboard`
**Affects:** `src/domain/` (parse, calc, series, format)

Todas as regras abaixo são novas — projeto greenfield. Cada uma exige teste unitário nomeado com seu ID.

---

## ADDED

### Requirement: RN-01 — Status de pagamento

Um gasto é **pago** quando `Estado = TRUE`; caso contrário é **pendente**.

#### Scenario: Gasto marcado na planilha
- GIVEN uma linha com `Estado = TRUE`
- WHEN o gasto é parseado
- THEN `pago` é `true`

#### Scenario: Célula de estado vazia
- GIVEN uma linha cuja coluna `Estado` está vazia
- WHEN o gasto é parseado
- THEN `pago` é `false` (ausência nunca significa pago)

---

### Requirement: RN-02 — Composição da data de vencimento

A data real de vencimento combina o dia da coluna `Vencimento` com o ano e mês extraídos do nome da aba.

#### Scenario: Dia válido no mês
- GIVEN a aba `2026-03` e um gasto com `Vencimento = 10`
- WHEN a data é composta
- THEN o resultado é `2026-03-10`

#### Scenario: Dia inexistente no mês
- GIVEN a aba `2026-02` e um gasto com `Vencimento = 30`
- WHEN a data é composta
- THEN o resultado é `2026-02-28`, o último dia do mês
- AND nenhum erro é lançado

#### Scenario: Ano bissexto
- GIVEN a aba `2028-02` e um gasto com `Vencimento = 30`
- WHEN a data é composta
- THEN o resultado é `2028-02-29`

---

### Requirement: RN-03 — Atraso

Um gasto está **atrasado** quando pertence ao mês corrente E está pendente E sua data de vencimento é anterior a hoje.

#### Scenario: Pendente e vencido no mês corrente
- GIVEN hoje é `2026-09-17` e a aba `2026-09` tem um gasto pendente com vencimento dia 5
- WHEN o status é calculado
- THEN o status é `atrasado`

#### Scenario: Pendente e ainda não vencido
- GIVEN hoje é `2026-09-17` e um gasto pendente com vencimento dia 22 no mês corrente
- WHEN o status é calculado
- THEN o status é `pendente`, nunca `atrasado`

#### Scenario: Pendência de mês anterior
- GIVEN hoje é `2026-09-17` e a aba `2026-07` tem um gasto pendente
- WHEN o status é calculado
- THEN ele é classificado como `pendência histórica`
- AND não é somado ao total de atrasados do mês corrente

#### Scenario: Pago após o vencimento
- GIVEN um gasto com `Estado = TRUE` e vencimento já passado
- WHEN o status é calculado
- THEN o status é `pago`, nunca `atrasado`

---

### Requirement: RN-04 — Total Gasto

**Total Gasto** é a soma de `Valor` de todas as linhas de gasto do mês, incluindo as de valor zero.

#### Scenario: Mês de referência
- GIVEN os 11 gastos reais da planilha
- WHEN o total é calculado
- THEN o resultado é `3271.59`

#### Scenario: Mês sem nenhum gasto
- GIVEN uma aba cujo bloco de gastos está vazio
- WHEN o total é calculado
- THEN o resultado é `0` e a aplicação não quebra

---

### Requirement: RN-05 — Pago e Falta Pagar

**Pago** é a soma dos gastos com `Estado = TRUE`. **Falta Pagar** é Total Gasto menos Pago.

#### Scenario: Nada pago
- GIVEN todos os gastos com `Estado = FALSE` somando `3271.59`
- WHEN os totais são calculados
- THEN `pago` é `0` e `faltaPagar` é `3271.59`

#### Scenario: Pagamento parcial
- GIVEN o Aluguel (`1350`) marcado como pago e o restante pendente
- WHEN os totais são calculados
- THEN `pago` é `1350` e `faltaPagar` é `1921.59`

---

### Requirement: RN-06 — Renda, Valor Restante e Caixa Atual

**Renda Total** é a soma do bloco de renda. **Valor Restante** é Renda Total menos Total Gasto, um valor **projetado**. **Caixa Atual** é Renda Total menos Pago, o dinheiro efetivamente disponível.

#### Scenario: Mês de referência sem pagamentos
- GIVEN Salário `8500`, Renda Extra `1000` e Total Gasto `3271.59` com nada pago
- WHEN as métricas são calculadas
- THEN `rendaTotal` é `9500`
- AND `valorRestante` é `6228.41`
- AND `caixaAtual` é `9500`

#### Scenario: As duas métricas divergem após pagamento
- GIVEN o mesmo mês com `1350` já pago
- WHEN as métricas são calculadas
- THEN `valorRestante` permanece `6228.41` (projeção não muda)
- AND `caixaAtual` é `8150`

#### Scenario: Mês sem bloco de renda
- GIVEN uma aba sem bloco de renda
- WHEN as métricas são calculadas
- THEN `rendaTotal` é `0` e a aplicação sinaliza renda ausente, sem quebrar

---

### Requirement: RN-07 — Rejeição de linhas de resumo

Linhas cujo nome pertença ao conjunto reservado nunca são tratadas como gasto, mesmo que apareçam dentro do bloco de gastos.

Conjunto reservado: `Total Gasto`, `Pago`, `Falta Pagar`, `Valor Restante`, `Salário`, `Renda Extra`.

#### Scenario: Layout legado com totais misturados
- GIVEN um bloco de gastos contendo uma linha `Total Gasto` com valor `3271.59`
- WHEN o bloco é parseado
- THEN essa linha não aparece na lista de gastos
- AND o Total Gasto calculado não a inclui (evitando dobrar o total)

#### Scenario: Comparação ignora espaços e caixa
- GIVEN uma linha chamada `  total gasto  `
- WHEN o bloco é parseado
- THEN ela também é rejeitada

---

### Requirement: RN-08 — Gastos de valor zero

Gastos com `Valor = 0` permanecem visíveis mas são excluídos das proporções e dos alertas.

#### Scenario: Fatura sem valor no mês
- GIVEN o gasto `Fatura Cartão Nubank` com `Valor = 0`
- WHEN o mês é processado
- THEN o gasto aparece na tabela com status `sem fatura`
- AND não entra no gráfico de categorias
- AND não gera alerta de vencimento, mesmo pendente e vencido

---

### Requirement: RN-09 — Ausência da aba do mês corrente

Se a aba do mês corrente não existir, a aplicação exibe o mês mais recente disponível com aviso explícito.

#### Scenario: Mês corrente ainda não criado
- GIVEN hoje é `2026-09-17` e existem apenas as abas `2026-01` a `2026-07`
- WHEN a aplicação carrega
- THEN `2026-07` é exibido
- AND um aviso informa que o mês corrente não existe na planilha

#### Scenario: Nenhuma aba válida
- GIVEN uma planilha sem nenhuma aba no formato `YYYY-MM`
- WHEN a aplicação carrega
- THEN é exibido um estado vazio explicando o formato esperado de nome de aba
- AND a aplicação não quebra

---

### Requirement: RN-10 — Dados inválidos

Linha com coluna A vazia encerra o bloco. Linha com `Gasto` preenchido mas `Valor` não numérico é descartada e reportada, nunca tratada como zero.

#### Scenario: Valor corrompido
- GIVEN uma linha `Energia` com `Valor = "quatrocentos"`
- WHEN o bloco é parseado
- THEN a linha não entra nos cálculos
- AND um `DataIssue` é registrado identificando aba, linha e motivo

#### Scenario: Linha em branco encerra o bloco
- GIVEN um bloco de gastos seguido de linha vazia e depois outras linhas
- WHEN o bloco é parseado
- THEN apenas as linhas anteriores à linha vazia são consideradas gastos

---

### Requirement: RN-11 — Formatação monetária

Todo valor monetário é formatado em `pt-BR`/`BRL` via `Intl.NumberFormat`. Cálculos operam sobre `number`; formatação ocorre apenas na renderização.

#### Scenario: Exibição de valor
- GIVEN o número `3271.59`
- WHEN formatado para exibição
- THEN o resultado é `R$ 3.271,59`

#### Scenario: Cálculo nunca usa string formatada
- GIVEN qualquer função de `domain/calc.ts`
- WHEN ela é executada
- THEN ela recebe e devolve `number`, nunca string formatada

---

### Requirement: RN-12 — Séries multi-mês

Comparações entre meses incluem apenas meses cuja aba existe. Meses ausentes são lacunas, nunca zero.

#### Scenario: Lacuna no meio do ano
- GIVEN abas `2026-01`, `2026-02` e `2026-04`
- WHEN a série de evolução é montada
- THEN ela contém três pontos, e `2026-03` não aparece como `0`

#### Scenario: Gasto ausente em um mês
- GIVEN o gasto `Energia` presente em `2026-01` e ausente em `2026-02`
- WHEN a série de comparação por gasto é montada
- THEN `2026-02` é uma lacuna, não um ponto com valor `0`

#### Scenario: Dados insuficientes para gráfico
- GIVEN menos de quatro meses disponíveis
- WHEN a tela de evolução é montada
- THEN os valores são apresentados como cards de estatística em vez de gráfico de linha

---

## MODIFIED

(Nenhum — projeto greenfield.)

## REMOVED

(Nenhum.)
